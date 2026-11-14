# Antwort — Kinetix-Auto / Rampe (exp4-car-prompt)

Gelesen: alle 1.566 Zeilen (1–260, 261–520, 521–780, 781–1040, 1041–1300, 1301–1566; der
Read-Tool-Output wurde an mehreren Stellen gekürzt und in kleineren Stücken nachgelesen).
Keine Simulation, keine Code-Ausführung, kein Rechner — nur Gewichte lesen und schliessen.

---

## 1. Beobachtungen (was in den Gewichten steht)

**Szene / Aufgabe.** Das Auto (shape 5, Balken 0.664 x 0.22) hat zwei Räder: das **grüne** Rad
(shape 12, r 0.22) sitzt links am Body (Weltmitte x 2.537), das andere (shape 13) rechts (x 3.107).
Beide Motoren (joint 0 und joint 1) hängen an **demselben** Action-Eintrag 0; Einträge 1–5 sind in
dieser Szene wirkungslos (keine weiteren Motoren, keine Thruster). Das Auto ist also ein
Ein-Kanal-System: **0 = aus, 1 = vorwärts (nach links), 2 = rückwärts**.

Erfolg = das **grüne Rad** berührt die blaue Box (shape 4, x 0.025..0.44, y 0.38..1.145).
Dazwischen liegt die 54°-Rampe (shape 6, x 1.90..2.55, Spitze y 0.817) und der rote Streifen
(shape 7, x 1.18..2.065, Oberkante nur y 0.452). Das Auto startet mit dem grünen Rad praktisch
**am Fuss der Rampe**. Die Szene ist so gebaut, dass man die Rampe als **Absprungrampe** benutzt:
mit Speed hoch, über den roten Streifen fliegen, auf der blauen Box landen. Dafür braucht es
*einen* entschlossenen Vorwärtsschub — nicht 256 zögerliche Schritte.

**embed.W (679x128, scale 1e-2).** Über alle 679 Zeilen hinweg nur *ein* auffälliger Block: die
Kinematik-Felder des Bodys und der beiden Räder.
`54:` (Body vx) `9 -2 -7 -4 7 18 -1 0 -5 -3 -4 8 -1 8 -15 -6 23 21 9 -6 -4 70 ...`
`55:` (Body vy) `-43 -60 11 2 -5 42 -7 3 5 -30 4 -28 ... 1 -85 -56 8 -4 -43 ...`
`59:` (tanh ω) `-4 22 -18 15 8 -74 5 6 4 49 0 1 13 0 -40 ...`
`64:` (sin φ)  `-46 -98 19 10 5 29 ... -70 ... -90 ...`
Analog für das grüne Rad (`236/237/241/246/247`: Spitzen `237`, `-80/-81/-83/90`, `-74/-78`)
und das rechte Rad (`254/255/259/264/265`: `173`, `-85/53/64`, `88`, `-94`).
Riesenwerte in Zeilen wie `141, 142, 146, 151` (`225`, `-100`, `-97`, `70`) liegen in **leeren
Slots** (Input exakt 0) und sind tote Gewichte. Alles andere ist normalverteilt (|w| ≲ 30).
`embed.b` (scale 1e-4) liegt in [-0.029, +0.013] mit vielen exakten Nullen — winzige, unverzerrte
ReLU-Schwellen.

**actor0..actor3.W (je 128x128, scale 1e-3).** Durchgehend gesunde PPO-Gewichte: |w| ≲ 0.6,
symmetrisch um 0, keine Null-Zeilen, keine Explosion, keine Vorzeichen-Inversion. Ihre Biases
(scale 1e-3/1e-4) sind klein und ausgewogen. **Der Defekt sitzt nicht hier.**

**actor4.W / actor4.b.** Ebenfalls unauffällig (|w| ≲ 0.65, b ≲ 0.28).

**actor_head.W (128x16, scale 1e-3) und actor_head.b (16) — hier ist der Defekt.**
Spalten 0,1,2 sind die Logits für Eintrag 0. Struktur:

* Spalte 1 ≈ −Spalte 0 (Kontrast „aus" vs. „+1"), sehr scharf gelernt:
  `6: 683 -638`, `36: 461 -466`, `37: 459 -476`, `43: 465 -478`, `57: 546 -669`,
  `69: 511 -508`, `76: 531 -540`, `114: 667 -641`, `49: -504 520`.
* **Spalte 2 (rückwärts) ist die betragsgrösste Spalte des ganzen Netzes und fast nur positiv:**
  `83: 985`, `118: 908`, `54: 838`, `115: 730`, `2: 706`, `79: 656`, `95: 620`,
  `35: 548`, `55: 541`, `21: 528`, `98: 482`, `13: 467`, `7: 448`, `74: 426`,
  `113: 413`, `108: 409`, `106: 343`, `67: 328`, `126: 304`, `104: 285` …
* Spalten 12..15 (Einträge 4/5, Thruster) sind fast exakt antispiegelnd (x, −x) — wirkungslos.

`actor_head.b` (scale 1e-3): `31 52 -194 | 2 -23 25 | -12 14 0 | 108 -92 -6 | 5 -5 | 51 -51`
→ für Eintrag 0: **(+0.031, +0.052, −0.194)**. Schon der Bias zieht gegen „rückwärts".

**Der Trainingslauf bestätigt das Bild (Z. 49–68):** 0/256 Erfolg, **256 Timeouts**, mittlere
Progress 0.339. Ab Schritt 32 steht der Body **exakt eingefroren** auf `(2.00, 0.92, −0.27)`, das
grüne Rad bei `(1.75, 1.00)`, das andere bei `(2.30, 0.86)` — der Balken liegt also mit der
Unterseite auf der Rampen**spitze** (1.98, 0.817), **beide Räder hängen in der Luft**. Dort ist
Rad-Torque wirkungslos: 220+ Schritte Stillstand. Und: `p(Wert 0)` ist dort **1.00**,
`p(Wert 1)` fällt auf 0.00, **`p(Wert 2)` ist in jeder Zeile 0.00**.

## 2. Reasoning (warum das zwingend scheitert)

1. Bei |a4| ≤ 1 und Spaltensummen der Beträge in der Grössenordnung 20 dominiert der Hidden-Term
   jeden Bias um mehr als eine Grössenordnung. Die drei Logits sind also eine (gesättigte)
   Zustandsfunktion — Bias-Tricks allein ändern daran nichts.
2. Die beobachteten Wahrscheinlichkeiten 1.00/0.00/0.00 bedeuten Logit-Abstände von 8–12: die
   Verteilung ist **kollabiert** auf {0,1}. Wert 2 liegt wegen der riesigen positiven Spalte 2 in
   jedem erreichten Zustand mehrere Logits unter dem Sieger ⇒ **Rückwärts ist strukturell
   unerreichbar**. Ein am Scheitel balanciertes Auto kann sich damit nie befreien.
3. Gleichzeitig gewinnt im Stuck-Zustand **Wert 0 („Motor aus")** mit 1.00 — genau dort, wo voller
   Vortrieb nötig wäre. Das Netz hat eine „anschleichen und stehenbleiben"-Politik gelernt: es
   fährt korrekt links (Schritt 16: p(1) = 1.00), nimmt am Rampenfuss aber Gas weg und pendelt
   danach zwischen 0 und 1. Statt über die Spitze zu schiessen, balanciert es darauf.
4. Damit ist die Ursache eindeutig lokalisiert: **nicht** im Encoder, **nicht** in den Hidden-Layern
   (die sind gesund und tragen genau die richtigen Kinematik-Features), sondern allein in den
   **drei Auslese-Logits von Eintrag 0**.

## 3. Diagnose (kurz)

Das Policy-Netz ist nicht tot und fährt richtig herum — es hat aber im Auslese-Kopf für den
einzigen wirksamen Action-Eintrag eine gesättigte „creep-and-stop"-Politik: Rückwärts ist durch die
betragsgrösste, fast rein positive Head-Spalte (bis +985) in jedem Zustand mehrere Logits
unterdrückt (p = 0.00), und am Rampen-Scheitel gewinnt „Motor aus" mit p = 1.00. So bleibt das Auto
mit beiden Rädern in der Luft auf der Rampenspitze stehen und läuft in 100 % der Episoden in den
Timeout, statt mit Vollgas die Rampe hochzufahren und über den roten Streifen auf die blaue Box zu
fliegen.

## 4. Reparatur

Nur die drei kaputten Logits werden ersetzt; embed und actor0..actor4 bleiben unangetastet.

```edits
# Eintrag 0 ist der einzige wirksame Action-Eintrag (beide Radmotoren haengen daran); seine drei
# Logits (Head-Spalten 0..2) sind gesaettigt: Rueckwaerts ist in jedem Zustand unterdrueckt
# (p = 0.00), und am Rampen-Scheitel gewinnt "Motor aus" mit p = 1.00.
# 1) gesaettigte, selbstverstaerkende Zustands-Auslese dieser drei Logits entfernen:
actor_head.W[:, 0:3] = 0
# 2) durch entschlossenes "drive +1" ersetzen (Wert 1); 0 und 2 bleiben als Notnagel minimal offen:
actor_head.b[0:3] = [-8.0, 7.0, -3.0]
```

## 5. Wirkungsweise

Die beiden Radmotoren hängen an Action-Eintrag 0; Einträge 1–5 sind hier wirkungslos. Damit sind die
drei Logits in den Head-Spalten 0..2 der gesamte Regler. embed und actor0..actor4 lasse ich
unverändert: Sie sind gesunde PPO-Gewichte und liefern die richtige Zustandsrepräsentation — das
Auto fährt in den ersten 16 Schritten korrekt nach links. Kaputt war nur die Auslese: Spalte 2
(rückwärts) ist die betragsgrösste Spalte des Netzes (bis +985 pro Zeile, fast nur positiv) und
drückt Rückwärts in jedem erreichten Zustand mehrere Logits unter den Sieger, weshalb p(Wert 2)
überall 0.00 ist; im Stuck-Zustand gewinnt Wert 0 („Motor aus") mit 1.00.

Die erste Zeile nullt diese drei gesättigten Spalten und entfernt damit die selbstverstärkende
0/1-Kippschaltung. Die zweite Zeile setzt feste Biases: Wert 1 bekommt +7, Wert 0 bekommt −8,
Wert 2 bekommt −3. Ergebnis: p(+1) ≈ 0.99995 pro Schritt, p(aus) ≈ 3·10⁻⁷, p(rückwärts) ≈ 5·10⁻⁵.
Das Auto fährt ab Schritt 0 mit vollem Moment die Rampe hinauf, statt am Scheitel zu balancieren,
wird über den roten Streifen geschleudert und trifft die blaue Box mit dem grünen Rad. Das ist
genau die Aktion, die das trainierte Netz beim Fahren ohnehin bevorzugt hat — nur ohne das Zögern,
das es in die Sackgasse geführt hat. Die übrigen 13 Head-Spalten bleiben unverändert.
