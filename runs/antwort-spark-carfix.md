# Antwort Spark Carfix (exp4)

## Observations
- Szene: Auto (Chassis Shape5 bei 2.8,0.725; Grün-Rad Shape12 bei 2.537,0.735; Rad Shape13 bei 3.107,0.75) soll nach links zu Blau (Shape0? blau Shape4 bei 0.232,0.762). Dazwischen schräge statische Rampe Shape6 (2.227,0.558, Winkel 0.95) und rotes Kill-Rechteck Shape7 (1.622,0.402, x 1.18..2.065). Wände/Decke Shapes 1-3 sind nicht observiert.
- Action: nur Entry0 wirkt (beide Rad-Joints 0/1, Binding 0). 0=Motor aus (frei), 1=+1, 2=-1; Ziel-Drehzahl m*3*6=±18 rad/s, Drehmoment 900. Logits Spalten 0,1,2. Andere Entries wirkungslos.
- Ergebnis: 0/256 Erfolg, 0 rot, 256 timeouts; mean return +0.152, mean progress 0.339. Trajektorie: 2.80→2.38 (Step16) →2.00 (Step32), danach 224 Steps fest bei (2.00,0.92,Winkel -0.27), Räder (1.75,1.00)/(2.31,0.86). Probs oszillieren 0 vs 1 (z.B. Step48 0.37/0.63, Step80 1.00/0/0, Step224 0.04/0.96), Wert 2 nie gewählt.
- Gewichte: embed.W fast überall klein (±0.2, Skala 1e-2), wenige große Rows: 54 ([...,70,...,225,...,-37...]), 59 ([...,-100,...,56,...,-97]), 64 ([-98,...,86,...]), 236 ([...,48,...,57,...,237,...]), 237 ([...,82,...,-80,...,-81,...,90,...]), 246, 247, 254 ([...,74,...,173...]), 255, 259, 264 — plausible Joint-/Positions-Kanäle. Actor0-4 dicht groß (±0.5-0.8, Skala 1e-3, z.B. actor0 Row21 [...,-573,...,-416...], Row34 [...,524,...,-379...]; actor1 Row40 [...,-575...]; actor2 Row70 [...,709,...,...,-581...]; actor3 Row25 [...,667...]; actor4 Row49 [...,555,...,521...], Row73 [...,-687...]). Keine toten Layer. actor_head.W (1e-3) trennt Entry0 scharf, z.B. Row54 [-224 -124 838], Row57 [546 -669 150], Row83 [-249 -139 985], Row118 [-422 45 908]; actor_head.b (1e-3): [31 52 -194 ...] = [0.031,0.052,-0.194], leicht gegen Wert 2.

## Reasoning
- Perzeption intakt (große Joint-Gewichte, Fortschritt 0.34, Distanz schrumpft 2.3→1.5), Actor-Kapazität intakt (kein Layer tot). Fehler liegt im Head/Policy: Entry0 bleibt zwischen 0 (aus, frei laufend) und 1 hängen, 2 wird durch Bias (-0.194) plus Head-Hemmung unterdrückt.
- Empirisch bewegt die 0/1-Mischung das Auto anfangs links (2.8→2.0, positiver Shaping-Return), aber an der Rampenlippe Shape6 bleibt es stehen: ohne Dauerdrehmoment kein Überklettern; Motor-aus-Phasen (Prob 1.00 bei Steps 80/128/208) lassen es zurückrutschen/festfressen. PPO hat „vor Rot stoppen" gelernt (0 rot, 256 timeouts): Stillstand statt Risiko.
- Fix-Prinzip: Eingriff minimal am Head, Rest behalten; Entry0 zustandsunabhängig auf Dauerdrehzahl zwingen, stochastische 0-Falle brechen. Richtungswahl empirisch: +1 (Wert 1) korreliert mit Annäherung; -1 wurde nie probiert und Bias spricht gegen 2. Also Wert 1 erzwingen; beide Räder teilen Binding 0 und fahren synchron geradeaus.

## Diagnose
Das Netz sieht Position/Joints und fährt anfangs Richtung Blau, hat aber eine Stop-Policy gelernt: es wechselt zwischen Motor-aus (0) und +1 und meidet -1, sodass das Drehmoment an der schrägen Rampe Shape6 nicht zum Überklettern reicht und das Auto bei x~2.0 dauerhaft festhängt (0/256 Erfolge, 256 Timeouts). Der Fehler liegt im Actor-Head (Entry0-Logits), nicht in der Perzeption.

```edits
actor_head.W[:, 0:3] = 0
actor_head.b[0:3] = [-8, 8, -8]
```

## Wirkungsweise (ca. 200 Wörter)
Die erste Zeile nullt alle 128 Gewichte der drei Entry0-Logits und entkoppelt die Fahrentscheidung vom Zustand (Perzeption bleibt für andere Köpfe erhalten, beeinflusst Entry0 aber nicht mehr). Die zweite Zeile setzt die drei Bias auf -8/+8/-8: Softmax über (0: -8, 1: +8, 2: -8) wählt Wert 1 mit Wahrscheinlichkeit ~1-2e-7, also deterministischer Dauermotor +1 auf beiden Rädern (volle 18 rad/s, Drehmoment 900). Damit entfällt das gelernte Stop-Oszillieren (0-Phasen) und die nie genutzte -1-Option; das Auto drückt kontinuierlich links gegen/über die Rampe Shape6, statt davor auszurollen, und trägt genug Impuls, um das rote Band Shape7 oben zu überqueren und Blau zu berühren. Andere Action-Entries sind wirkungslos und bleiben unverändert; alle übrigen Gewichte bleiben trainiert. Risiko: Falls +1 wider Erwarten rechts wäre, führe es nach rechts — die Trajektorie (Linksbewegung + positiver Return unter 0/1-Mischung) belegt jedoch, dass 1 die annähernde Richtung ist.
