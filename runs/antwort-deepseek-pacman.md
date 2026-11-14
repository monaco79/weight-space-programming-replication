Observations:

Skalenbild in W1. Die grosse Mehrheit der 500 Zeilen von W1 ist Rauschen der Initialisierung: Beträge typisch 0.02-0.13, z. B. Zeile für Input 0 (Feld (0,0), Kanal 0) `-0.051 0.030 -0.024 ... 0.133 0.004 0.010 -0.082`. Eine Minderheit von Zeilen ist um den Faktor 20-50 grösser (Beträge 2-6). b1 ist winzig (`0.003 0.002 0.076 -0.041 ... 0.004 0.034 -0.006 ...`, max +0.076, min -0.126), d. h. kein Hidden-Unit ist von sich aus an/aus; die gesamte Information kommt aus den Eingängen. Die grossen Zeilen haben untereinander keine räumliche Glattheit (benachbarte Zellen haben völlig unkorrelierte 32-dim Codes), also sind es gelernte One-Hot-Positions-Embeddings, keine lokalen Filter.

Kanal 0 (Agent). Genau 48 der 64 Innenzellen haben riesige Zeilen, z. B. Input 55 = (1,1,ch0):
`1.706 -3.242 0.524 -1.906 1.067 ... 3.377 1.205 2.603 ... -3.734 4.081 0.128 1.642` (Units 12/28/29: +3.38 / -3.73 / +4.08).
Input 120 = (2,4,ch0): `-3.315 -3.218 ... 5.140 4.144 ...`.
16 Innenzellen bleiben dagegen auf Init-Niveau, über ALLE fünf Kanäle hinweg. Beispiel Input 110 = (2,2,ch0): `0.119 0.015 -0.048 ... -0.131 ... -0.152`; die zugehörigen Zeilen (2,2,ch1)=111 `0.019 -0.121 0.139 ...`, (2,2,ch2)=112, (2,2,ch3)=113, (2,2,ch4)=114 sind ebenso klein. Diese 16 Zellen sind (2,2),(2,3),(2,6),(2,7),(3,2),(3,7),(4,4),(4,5),(5,4),(5,5),(6,2),(6,7),(7,2),(7,3),(7,6),(7,7) – ein exakt spiegel- und punktsymmetrisches Muster. Sie sind also Zellen, die in keiner Episode je als Agent, Objekt, Ziel oder Hindernis auftreten: feste Wände. Die 48 aktiven Zellen bilden damit ein festes, handgebautes Labyrinth (Randkorridor plus Kreuz in der Mitte, fünf Wandinseln: vier L-förmige + 2x2-Zentrum):

```
. . . . . . . .     (Reihe 1, alles Korridor)
. X X . . X X .     (Reihe 2)
. X . . . . X .     (Reihe 3)
. . . X X . . .     (Reihe 4)
. . . X X . . .     (Reihe 5)
. X . . . . X .     (Reihe 6)
. X X . . X X .     (Reihe 7)
. . . . . . . .     (Reihe 8)
```

Kanal 1 (bewegte Objekte). Praktisch jede Korridorzelle hat eine grosse ch1-Zeile, z. B. Input 56 = (1,1,ch1): `-1.161 0.066 2.453 1.978 ... 1.550 ... -1.373 1.007`; Input 121 = (2,5,ch1): `-0.560 -1.328 3.745 ...`; Input 126 = (2,6,ch1): `-0.461 0.174 2.762 -0.962 1.309 -5.209 ...`. Ein autonom umherziehendes Etwas besucht also den ganzen Korridor.

Kanal 4 (Vorframe). Ebenfalls gross auf fast allen Korridorzellen, z. B. Input 59 = (1,1,ch4): `0.487 -0.215 -0.550 0.253 -0.004 -0.241 -0.739 -0.646 0.176 -1.316 ... 0.593`; Input 74 = (1,4,ch4): `-0.940 0.177 -3.515 -0.771 -2.097 3.194 ... 4.961 ... -4.237`. Die Vorposition von Agent/Objekt ist also stark benutzt – das Netz braucht Bewegungsrichtung/Prädiktion des anderen Objekts.

Kanal 2 (Reward-Ziele). Auf allen Korridorzellen vorhanden, aber eine Größenordnung kleiner als ch0/ch1 (typisch 0.2-0.7): Input 77 = (1,5,ch2) `0.032 -0.031 ...`; Input 87 `... 0.602 ...`; Input 92 `... 0.748`; einige Ausreisser: Input 62 = (1,2,ch2) `... 0.102 -2.361 ...` und Input 122 = (2,4,ch2) `... -0.041 -4.876 ...`. Das passt zu Zielen, die auf vielen Zellen liegen, aber im Episodenverlauf verschwinden (gefressen werden) – also Punkte/Pellets, nicht ein einzelnes statisches Ziel.

Kanal 3 (Hindernisse). Praktisch tot, alle Zeilen ~0.1: Input 73 = (1,4,ch3) `-0.007 -0.145 -0.019 ... 0.173`; Input 168 = (3,3,ch3) max 0.409. Es gibt also keine dynamische Gefahr in diesem Kanal; Wände sind konstant (und werden implizit über die 48 Agent-Zellen kodiert).

Ausgabe. b3 = `-0.873 0.703 0.712 0.595 -0.092`: No-op ist stark unterdrückt, die drei Richtungen up/down/left sind leicht positiv, right neutral – die Policy bewegt sich praktisch immer. W3 (5 Spalten = no-op, up, down, left, right) hat klare Gegenspieler-Struktur zwischen Gegenrichtungen:
u16 `-1.061 -1.254 0.200 2.490 -0.338` (left stark, up/down gehemmt),
u25 `0.209 1.893 -0.742 -2.491 -0.386` (up stark, left gehemmt),
u30 `-0.501 1.326 -1.247 -1.168 2.223` (right stark, down gehemmt),
u31 `1.439 2.019 -0.268 -0.516 -2.023` (up gegen right),
u14 `0.262 1.662 -0.822 0.481 -1.710`,
u13 `0.147 -2.190 0.991 -1.094 1.858`.
Das ist eine reine 4-Richtungs-Lenk-Policy (Ziel anziehen / Gefahr abstossen), kein statisches Aktionsmuster. W2 ist voll durchmischt (z. B. `0.125 -0.732 0.764 ... 1.188 ...`), also keine lineare Durchreichung.

Reasoning:

1. Feste Wände: 16 Zellen sind in allen Kanälen stumm und symmetrisch angeordnet. Das schliesst Offenfeld-Spiele (Snake auf freiem 10x10, Pong, Breakout) und pro Episode randomisierte Labyrinthe aus; es ist ein handgebautes, statisches Labyrinth.
2. Kanal 3 ist überall Init-Niveau. Damit fehlt jede dynamische Gefahr in diesem Kanal. Snake wäre damit ausgeschlossen (der Körper als Hindernis müsste überall starke ch3-Gewichte erzeugen) und Sokoban ebenfalls (Kisten als Hindernis/Ziele wären in ch1/ch2 konzentriert, nicht flächig; ausserdem würde Warten/No-op genutzt statt unterdrückt).
3. Kanal 1 ist auf allen Korridorzellen riesig: es gibt ein selbstbewegtes Objekt, das den gesamten Korridor durchstreift. Reine Ziel-/Futter-Suche ohne Gegner (FrozenLake, reines Maze-Solving) ist damit ausgeschlossen.
4. Kanal 4 ist fast so stark wie ch0/ch1: die Vorpositionen werden aktiv genutzt, d. h. Richtung/Prädiktion des bewegten Objekts ist entscheidend. Das ist Verfolgung/Flucht, nicht nur Navigation.
5. Kanal 2 ist flächig, aber schwach und mit einzelnen Ausreissern: viele Ziele, die im Episodenverlauf verschwinden (gefressen werden).
6. b3 unterdrückt No-op und bevorzugt Bewegung; W3 kodiert Richtungsgegensätze. Das passt zu einem Spiel, in dem man ständig weiterlaufen muss und in dem ein bewegter Gegner dieselben Gänge nutzt.
7. Zusammen: festes symmetrisches Labyrinth + ein (oder wenige) umherziehende(r) Gegner in ch1 + flächige, verbrauchbare Rewards in ch2 + toter Hinderniskanal + erzwungene Dauerbewegung = Maze-Chase. Der kanonische Vertreter mit genau dieser Merkmalskombination (Labyrinth, Punkte fressen, Geist, Kontakt = Episodenende) ist Pac-Man. Ein Labyrinth-Spiel mit Jäger wäre die generische Alternative; Pac-Man ist die spezifischste passende Antwort.

Game: Pac-Man (stark vereinfachter Pac-Man-/Maze-Chase-Klon auf einem festen 8x8-Labyrinth: Ringkorridor plus Kreuz, ein umherziehender Geist, Pellets als Reward)

Description: Der Agent navigiert Zelle für Zelle durch ein festes, symmetrisches Labyrinth, frisst Pellets für Reward und muss dem Geist ausweichen, der durch dieselben Gänge zieht; Kontakt mit dem Geist beendet die Episode (Wände sind unpassierbar).

Confidence: 78%
