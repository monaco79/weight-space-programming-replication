# DeepSeek — Carfix Experiment, Chunk-Protokoll

Datei: runs/exp4-car-prompt.txt (1.566 Zeilen, 171.664 Gewichte)
Auftrag: Netz-Diagnose (0/256 Erfolge) + Reparatur per ```edits```. Keine Simulation/Ausführung.

---

## Segment 1 — Zeilen 1–267 (Task, Szene, Regeln, Architektur, Ergebnis, embed.W Zeilen 0–180)

### A. Aufgabe / Format
- Zeile 1–3: PPO, 1e9 Schritte, nie Erfolg. "First read the weights and explain why the trained network fails".
  Danach nur Gewichte ändern; Erfolg > wenige Gewichte ändern. 1024 neue Episoden, Aktionen gesampelt.
- Zeile 70–80: Edit-Sprache: `actor_head.b[0:3] = [..]`, `embed.W[26,5] = 1.25`,
  `actor0.W[:,3] *= 0.5`, `ZERO actor4.W`, ranges halb-offen, `:` = ganze Achse, spätere Zeile überschreibt.
  Nur der LETZTE edits-Block zählt.

### B. Szene (Zeilen 9–23)
- shape 0: static rect, centre (2.5,-4.8), halb 2.5x5.2 → x 0..5, y -10..0.4  (Boden/Block links)
- shape 1: static poly (-5,5),(0,5),(0,0),(-5,0) — Decke links
- shape 2: static poly (5,5),(10,5),(10,0),(5,0) — Decke rechts
- shape 3: static rect, centre (2.5,10.2) → x 0..5, y 5..15.4 — Decke
- shape 4: **dynamic rect, BLUE**, centre (0.232,0.762), halb 0.207x0.382 → x 0.025..0.44, y 0.38..1.145
  (hochkant stehende "Zielflagge" ganz links, steht auf dem Boden y=0.4)
- shape 5: **dynamic rect, car body**, centre (2.8,0.725), halb 0.332x0.11 → x 2.467..3.132, y 0.615..0.835
- shape 6: **static rect, RAMPE**, centre (2.227,0.558), halb 0.067x0.352, **angle 0.95 rad**,
  Ecken (1.98,0.817),(2.553,0.407),(2.475,0.298),(1.902,0.708)  → schräge Platte zwischen Car und Ziel
- shape 7: **static rect, RED (fail)**, centre (1.622,0.402), halb 0.442x0.05 → x 1.18..2.065, y 0.352..0.452
  (flache rote Platte direkt links der Rampe, auf dem Boden)
- shape 12: **dynamic circle, GREEN**, centre (2.537,0.735), r 0.22  ← linkes Rad, = Erfolgs-Kontaktobjekt
- shape 13: dynamic circle, centre (3.107,0.75), r 0.229  ← rechtes Rad
- Zeile 19: shapes 1,2,3 = statische Wände/Decke, NICHT in der Observation.

### C. Joints / Action (Zeilen 22–30)
- joint 0: revolute, a=5 (car body, lokal (0.307,0.025)) ↔ b=13 (rechtes Rad). motor on, speed 3, power 1,
  keine Limits, **binding = action entry 0**.
- joint 1: revolute, a=5 (lokal (-0.262,0.01)) ↔ b=12 (grünes Rad). motor on, speed 3, power 1,
  **binding = action entry 0**.
- Action: 6 ints. entry0: 0=off, 1=+1, 2=-1. Ziel-Drehzahl = m*3*6 = ±18 rad/s, Torque ~ base power 900.
  entries 1..5 wirkungslos (keine weiteren Motoren/Thruster).
- Erfolg: grün berührt blau (+1). Rot berührt = -1 (rot gewinnt bei Gleichzeitigkeit). 256 Steps, 2 substeps à 0.01667 s.
- Reward-Shaping: +0.2*(d_prev - d) Distanz grün↔blau.

### D. Observation (Zeilen 32–42)
- 679 float32, clip [-10,10]. poly slots 0,4,5,6,7,8,9,10,11 (je 26) → 0..233;
  circle slots 12..15 (je 18) → 234..305; joint slots 0..5 (je 54) → 306..629; thruster 0..1 (24) → 630..677; 678 = gravity_y/10.
- Poly-Felder (26): 0 pos x, 1 pos y, 2 vel x, 3 vel y, 4 inv mass, 5 inv inertia, 6 density,
  7 tanh(angvel/10), 8 role none, 9 role green, 10 role blue, 11 role red, 12 sin(angle), 13 cos(angle),
  14 friction, 15 restitution, 16 =0, 17..24 vertices, 25 = (verts<=3).
- **Wichtig: slot 5 = car body; Felder 54=velx, 55=vely, 59=tanh(angvel), 64=sin, 65=cos von shape 5.**

### E. Netz (Zeile 47)
- embed: relu(x@embed.W + b) → 128; a0..a4 = tanh, je 128; logits = a4@actor_head.W + b (16).
- Logit-Spalten: entry0 → 0,1,2 ; entry1 → 3,4,5 ; entry2 → 6,7,8 ; entry3 → 9,10,11 ; entry4 → 12,13 ; entry5 → 14,15.
- value head nicht gezeigt/irrelevant.

### F. Trainings-Ergebnis (Zeilen 49–68) — DER SCHLÜSSEL
- 0/256 Erfolg, 0 red, **256 timeout**, invalid 0, mean return 0.152, mittlere Länge 256,
  **mean progress 0.339** (1 = Ziel), most-likely-action = timeout.
- Episode-Trace (jede 16 Steps): car body startet (2.80,0.73,0.00). Bis Step 32 fährt es nach LINKS
  auf (2.00,0.92) und kippt (angle -0.64 → -0.27). Danach **exakt eingefroren: (2.00,0.92,-0.27) bzw.
  (2.01,0.92,-0.28) für Steps 32..255**. Grünes Rad bei (1.75,1.00), rechtes Rad (2.30,0.86).
- Action entry 0 wechselt nur zwischen 0 (p≈1.00) und 1 (p≈0.63–0.96); **Wert 2 (rückwärts) hat
  durchgehend p = 0.00**. Also: Netz fährt vorwärts, bleibt an der Rampe hängen und kann nicht zurücksetzen.
- Geometrie-Check: grünes Rad steht bei x=1.75 und damit **genau auf der roten Platte (x 1.18..2.065,
  y 0.352..0.452)**? Nein — Rad-y=1.00, rot-y nur bis 0.452, also noch kein Rot-Kontakt.
  Aber car body bei y=0.92, angle -0.27: das Fahrzeug ist auf die Rampe (shape 6, Spitze 0.817) gekippt
  und liegt/klemmt dort. "progress 0.339" heißt: es ist nur ~1/3 des Weges zum Ziel gekommen.

### G. embed.W, Zeilen 87–267 = Zeilen 0..180, scale 1e-2
Beobachtete Muster (Auszug, Zitate):
- Zeile 54 (slot5, vel x): `9 -2 -7 -4 7 18 -1 0 -5 -3 -4 8 -1 8 -15 -6 23 21 9 -6 -4 70 13 -11 -19 -11 -1 13`
- Zeile 55 (slot5,**vel y**): `-43 -60 11 2 -5 42 -7 3 5 -30 4 -28 -14 -1 11 -8 1 -85 -56 8 -4 -43 0 13 1 -38 ...`
  (Spitzen -85, -56, -60, -43 — sehr groß für 1e-2-Skala)
- Zeile 59 (slot5, **tanh angvel**): `-4 22 -18 15 8 -74 5 6 4 49 0 1 13 0 -40 3 5 -1 ...`
  (Spitzen -74, +49, -40)
- Zeile 64 (slot5, **sin angle**): `-46 -98 19 10 5 29 ... -70 ... -90 ...` (Spitzen -98, -90, -70)
- Zeile 52 (slot5, pos x): `7 19 -2 -5 ...` ; Zeile 53 (pos y), 56..58, 60..63, 65 ff. sind klein (|w|<20).
- Zeilen 141, 142, 146, 151 tragen ebenfalls Riesenwerte (`225`, `-100`, `-97`, `70`), liegen aber in
  **slots 8..11 = leere Slots (Input exakt 0)** → tote Gewichte, ohne Wirkung. (Zeile 141: 130..155 = slot 8.)
- Übrige Zeilen (0..~53, 66..140, 152..180) wirken wie normalverteilte Standard-PPO-Gewichte (|w| ≲ 20,
  viele ±1..±15), d.h. embed ist insgesamt "gesund" und nicht kollabiert.

## Segment 2 — Zeilen 261–520 = embed.W Zeilen 174–433

Wichtig fürs Lesen: Dateizeile = Label + 87. (Label = Zeilenindex im Tensor.)

- Zeilen 268–320 = Labels 181–233: Ende Polygon-Slots (slot 10 = 182..207, slot 11 = 208..233, beide LEER).
  Durchgehend "normale" Werte (|w| ≲ 20), viele ±1..±15. Keine Anomalie.
- Zeile 321 = Label 234 = **circle slot 12 (grünes Rad) pos x**: `11 11 -1 -8 ...` normal.
- Zeile 322 = Label 235 = slot12 pos y: normal.
- Zeile 323 = Label 236 = **slot12 vel x**: `16 -3 0 -7 -2 23 -1 -13 0 -12 1 21 -2 1 -13 -3 24 34 3 1 -6 48 ... -33 ... -42 ... 30 57 ... 237 ... -44 ... -40 ... -31`
  → Ausreisser 237, 57, 48, 34.
- Zeile 324 = Label 237 = **slot12 vel y**: `-15 -59 13 -12 -12 82 0 8 8 -80 6 -10 ... -52 ... 20 ... 71 ... -81 ... -83 ... 90 ... 80`
- Zeile 328 = Label 241 = **slot12 tanh(angvel/10)**: `-26 -25 -15 15 5 12 0 11 4 -19 ... 28 ... -38 ... 37 ... 34 ... -31 ... 36`
- Zeile 333 = Label 246 = **slot12 sin(angle)**: `-27 18 16 10 1 62 5 -1 6 10 5 -7 16 -6 -68 2 15 26 0 27 7 -17 -4 5 1 -48 -1 -50 ... -64 ... -67 ... 55 46 ... 42 42 ... -65 ... 58 ... 65`
- Zeile 334 = Label 247 = **slot12 cos(angle)**: `-13 -22 18 3 0 31 0 -11 6 -26 6 -28 ... -74 ... -78 ... 46 52 ... -39 ... 65 46 29 ... -40 ... -74 ... -60`
- Zeile 341 = Label 254 = **slot13 (rechtes Rad) vel x**: `3 -18 -33 12 11 -2 ... -23 ... 25 16 ... 74 ... 52 44 ... 173 ... -37 9 ... 53 -37`
- Zeile 342 = Label 255 = **slot13 vel y**: `-43 -9 -4 33 3 -36 ... -42 -31 ... -37 ... 59 ... -38 ... -36 ... -85 53 ... 64 16 -72`
- Zeile 346 = Label 259 = **slot13 tanh(angvel)**: `-24 -32 3 19 19 2 2 -8 -16 -45 ... 27 ... -36 ... 53 ... 42 ... -41 ... -55 -46 ... -48 ... -52`
- Zeile 351 = Label 264 = **slot13 sin(angle)**: `-17 -26 -18 6 -17 -45 ... -56 ... -51 33 ... -27 ... 88 ... 65 81 ... 61 ... -63 ... -59`
- Zeile 352 = Label 265 = **slot13 cos(angle)**: `9 18 21 -29 -3 0 0 -3 4 1 -10 5 41 3 -29 ... -18 -22 33 ... -27 ... 28 ... -59 ... -67 ... 65 ... -64 ... -94 -43 ... 56 41 ... 53 ... -44 ... 39`
- Zeilen 361–392 = Labels 274–305: slot 14 und 15 (LEER, Input = 0) — Werte normal; ohne Wirkung.
- Zeilen 393–406 = Labels 306–319 (**joint slot 0**: active, fixed, pos_b, pos_a, sin/cos ref, motor speed, power, on, limits,
  binding one-hot). Alle |w| ≲ 20. Kein Joint-spezifischer Ausreisser.
- Zeilen 407–459 = Labels 320–372 (restliche joint0-Felder + joint1 Anfang). Ebenfalls normal (|w| ≲ 20).
- Zeilen 460–520 = Labels 373–433 (joint1 + erste Thruster-Slots 630 ff. kommen erst später). Normal.

**Muster-Bestätigung:** Die einzigen wirklich "heissen" embed-Zeilen sind die Kinematik-Felder
(vel x, vel y, tanh(w), sin, cos) von **car body (54/55/59/64)**, **grünem Rad (236/237/241/246/247)**
und **rechtem Rad (254/255/259/264/265)**. Genau diese Features sind für "stehe ich an der Rampe /
rolle ich / kippe ich" zuständig. Joints, Positionen, Rollen-One-Hots sind unauffällig.
Es gibt also KEINEN Hinweis auf einen kaputten Encoder — das Netz "sieht" den Stuck-Zustand sehr genau
und hat eine starke, gelernte Repräsentation dafür. Der Fehler muss im Policy-Kopf liegen (der diese
Repräsentation falsch auf die 16 Logits abbildet), nicht in embed.

## Segment 3 — Zeilen 521–780 = embed.W Zeilen 434–678 + embed.b (Z. 767–768)

- Zeilen 528–765 = Labels 441–678 (Thruster-Slots 630..677, gravity 678). Durchgehend normale
  PPO-Werte: |w| fast immer < 30, gelegentlich ±50. KEINE Ausreisser, kein Muster, keine Null-Spalten.
- Zeile 765 = Label 678 = gravity y/10: `0 -12 6 -4 -2 2 -13 -5 -11 5 7 2 ...` → normal.
  (D.h. das Netz nutzt die Gravitations-Konstante nur schwach.)
- **embed.b (Zeile 768), scale 1e-4**: `-117 39 -211 -176 -108 91 0 0 0 -63 -5 -239 -128 0 -76 0 -116 -242 ...`
  Werte alle in [-288, +130] × 1e-4 = [-0.029, +0.013]; **viele exakt 0**. Also: winzige Biases,
  ReLU-Schwellen praktisch unbiased. Das embed ist gesund (kein toter/explodierender Layer).

**Wichtig:** embed.W hat über 679 Zeilen hinweg nur EINEN auffälligen Block: die Kinematik-Zeilen
54,55,59,64 (car body) und 236,237,241,246,247 / 254,255,259,264,265 (die beiden Räder),
plus tote Zeilen in leeren Slots (130–233, 270–305). Kein kaputter Sensor-Encoder.

## Segment 4 — Zeilen 781–1040 (actor0.W 9–127, actor0.b, actor1.W 0–4)

- actor0.W (scale 1e-3): Zeilen 771–898 = 128x128. Werte typisch ±50..±400, Ausreisser bis ±600
  (z.B. Zeile 18: `204 -291 -127 ... -459`, Zeile 22: `... -573 ... -416`). **Statistisch normal,
  keine Null-Zeilen, keine Explosion, kein Vorzeichen-Muster.** Klassische PPO-Gewichte.
- actor0.b (Z. 901): `-5 -246 -96 ... 945 ... -563 416 ... 692 ...` (scale 1e-4) → ±0.09, unauffällig.
- actor1.W (Z. 903 ff., scale 1e-3): Zeile 0..127 ebenfalls normal (|w| ≲ 480), z.B. Zeile 6
  `-58 -123 -340 379 ... 754 ... -591`; Zeile 17 `... 510 ... 455 ...`.

## Segment 5 — Zeilen 1041–1300 (actor2.W, actor2.b, actor3.W, actor3.b)

- actor2.W (Z. 1037–1164, scale 1e-3): 128x128, normal; Ausreisser z.B. Zeile 70 `... 709 ... 626 ... -581`,
  Zeile 83 `... 985?` nein — max hier ~754. Keine Struktur, die auf Kollaps hindeutet.
- actor2.b (Z. 1167): `124 -115 ... -146 ... 194` (scale 1e-3) → ±0.2, normal.
- actor3.W (Z. 1170–1297, scale 1e-3): normal, Ausreisser Zeile 19 `... 552 ...`, Zeile 85 `... 605 ...`.
- actor3.b (Z. 1300): `65 -151 8 ... 201 ...` → normal.

**Kernaussage Segmente 4+5:** Alle Hidden-Layer (actor0..actor3) sind unauffällige, gesunde
PPO-Gewichte (scale 1e-3, |w| ≲ 0.6, symmetrisch um 0, keine toten Einheiten). Der Defekt sitzt
NICHT in den Hidden-Layern und nicht in embed. Er sitzt ausschliesslich im Auslese-Kopf
(actor4 / actor_head) für Action-Entry 0.

## Segment 6 — Zeilen 1301–1566 (actor4.W, actor4.b, actor_head.W, actor_head.b)

- actor4.W (Z. 1303–1430, 128x128, scale 1e-3): normal (|w| ≲ 0.65).
- actor4.b (Z. 1433): `42 -85 -84 9 ... 200 ... 276 ... -245` → normal.
- **actor_head.W (128 x 16), scale 1e-3, Z. 1436–1563** — hier liegt der Defekt.
  Spalten 0,1,2 = Logits für Entry 0 (Werte 0/1/2). Muster:
  * Spalte 1 ≈ −Spalte 0 in vielen Zeilen (Kontrast "Motor aus" vs "+1"): z.B.
    Zeile 6 `683 -638`, Zeile 36 `461 -466`, Zeile 37 `459 -476`, Zeile 43 `465 -478`,
    Zeile 57 `546 -669`, Zeile 69 `511 -508`, Zeile 76 `531 -540`, Zeile 114 `667 -641`,
    Zeile 49 `-504 520`.
  * **Spalte 2 (Rückwärts) ist die betragsgrösste Spalte des ganzen Tensors und fast nur positiv:**
    Zeile 83 `985`, Zeile 118 `908`, Zeile 54 `838`, Zeile 115 `730`, Zeile 2 `706`,
    Zeile 79 `656`, Zeile 95 `620`, Zeile 35 `548`, Zeile 55 `541`, Zeile 21 `528`,
    Zeile 98 `482`, Zeile 13 `467`, Zeile 7 `448`, Zeile 74 `426`, Zeile 113 `413`,
    Zeile 108 `409`, Zeile 106 `343`, Zeile 67/104/126 `328/285/304` usw.
  * Spalten 12..15 (Entry 4/5, Thruster) sind fast exakt antispiegelnd (x, −x), z.B. Zeile 58
    `-208 208`, Zeile 1494 `93 -92`. Wirkungslos (keine Thruster in der Szene).
- **actor_head.b (16), scale 1e-3, Z. 1566:** `31 52 -194 2 -23 25 -12 14 0 108 -92 -6 5 -5 51 -51`
  → Entry 0: Bias = (0.031, 0.052, **−0.194**). Also ist bereits der Bias gegen Rückwärts gerichtet.

**Entscheidende Rechnung (rein qualitativ, keine Ausführung):** Bei |a4| ≤ 1 und Summen der
Spaltenbeträge in der Grössenordnung 20 dominiert der Hidden-Term jeden Bias um mehr als eine
Grössenordnung. Beobachtet: p(0)/p(1) springt zwischen 1.00/0.00 und 0.00/1.00, p(2) ist in JEDEM
erreichten Zustand 0.00 ⇒ Logit-Abstand ≈ 8–12 (Sättigung). Spalte 2 drückt Rückwärts in jedem
Zustand um mehrere Logits unter den Sieger ⇒ **Rückwärts ist strukturell unerreichbar**.
Im Stuck-Zustand gewinnt Wert 0 ("Motor aus") mit p = 1.00 — genau dort, wo voller Vortrieb nötig wäre.

**Geometrie-Konsequenz:** Auto startet mit dem grünen Rad (x 2.537) direkt am Fuss der Rampe
(shape 6, x 1.90..2.55, Spitze y 0.817). Erfolg = mit Vollgas die 54°-Rampe hochfahren und über den
roten Streifen (x 1.18..2.065, nur bis y 0.452) hinweg auf die blaue Box (x 0.025..0.44,
y 0.38..1.145) fliegen. Dafür braucht es genau EINEN entschlossenen Vorwärts-Schub, nicht 256
zögerliche 0/1-Wechsel. Das trainierte Netz bremst stattdessen am Rampen-Scheitel ab und
balanciert dort (beide Räder in der Luft, Torque wirkungslos).

**Zwischenfazit Segment 1:** Das Policy-Netz ist nicht zufällig/tot — es fährt gezielt links, bis es an
der Rampe (shape 6) hängen bleibt, und hat gelernt, in diesem Zustand almost-deterministisch "0" (Motor aus)
oder "1" (vorwärts) zu wählen. Die Reverse-Aktion (Wert 2) wird nie gewählt. Ohne Rückwärts-Manöver ist
die Rampe nicht überwindbar → 100 % Timeout. Kandidat für den Defekt: actor_head / actor4 (Policy-Kopf)
und/oder die starken sin/vel-Gewichte in slot 5, die den "stuck at ramp"-Zustand einrasten lassen.
