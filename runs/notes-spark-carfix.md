# Notes Spark Carfix – 2026-11-14
## Segment 1: Zeilen 1-207 (Promptkopf + Szene + embed.W 0-120)
- Datum: 2026-11-14, Segment 1-207 abgedeckt.
- Kernaussagen: Kinetix-Auto-auf-Rampe; 679-dim Observation; Action 6 ints, nur Entry 0 (Motor an beiden Rädern, bindings) wirksam; Policy-Logits 16 Spalten (0:2 für Entry0). Episode 256 Steps, Erfolg = grün berührt blau, Misserfolg = rot berührt.
- Ergebnis: 0/256 Erfolge, 0 rot, 256 timeouts; mean return 0.152, mean progress 0.339; Most-likely timeout. Sample-Trajektorie: Auto startet (2.8,0.73), fährt kurz rückwärts/links bis x~2.0 (Step16-32), bleibt dann stehen bei (2.00,0.92) Winkel -0.27 für Rest (Steps 32-255) – steckt an Rampe/Hindernis fest. Action-Probs oszillieren (0 vs 1), kein Drive -1.
- Szene-Geometrie: shape5 Chassis dynamisch (2.8,0.725); shape12 grünes Rad (2.537,0.735), shape13 neutrales Rad (3.107,0.75); shape6 schräge statische Rampe/Block bei (2.227,0.558) Winkel 0.95; shape7 rotes flaches Rechteck (1.622,0.402); blaues Ziel (0.232,0.762) links oben. Auto muss nach links über Rampe/hinunter zum Blau.
- Zitierte Gewichte (embed.W scale 1e-2): Row54 auffällig groß: [...,70,13,-11,...,225,-18,...,-37,...] – Joint-Slot? Row59: [...,-100,56,...,-97], Row64: [-46,-98,19,...], Row55: [-43,-60,11,...]. Deuten auf große Joint/Positions-Empfindlichkeit, aber finales Verhalten = Stillstand.
- Edit-Sprache: `actor_head.b[0:3]=...`, `ZERO tensor`, letzte Edits-Block zählt.

## Segment 1b: Zeilen 208-267 (embed.W 121-180)
- Datum: 2026-11-14, Zeilen 208-267 abgedeckt.
- Kernaussagen: Fortsetzung embed.W, durchgehend kleine Ints (±~18), keine weiteren Ausreißer wie Row54/59/64. Typische Zeilen z.B. 121: [-5 9 -2 ...], 127 mit 18 bei Spalte ~95. Skala 1e-2 → Gewichte ±0.2.
- Zitierte Gewichte: Row127: [...,18,-5,9,3,7,...]; Row138: [...,20,3,...]; alles klein vs Row54:225, Row59:-100, Row64:-98 etc. Bestätigt: nur wenige Rows (54,55,59,64) haben große Beträge, vermutlich Joint/Positions-Kanäle.

## Segment 2-Anteil: Zeilen 268-327 (embed.W 181-240)
- Datum: 2026-11-14, Zeilen 268-327 abgedeckt.
- Kernaussagen: Weiter embed.W klein (±0.2). Keine neuen Großgewichte; bestätigt sparse große Rows früh.
- Zitierte Gewichte: Row234: [...,-22,...,14,...,-22...], Row236: [...,48,-1,-10,...,57,...,237,-44,...], Row237: [...,82,...,-80,...,-81,...,90,-16,...,80] – zweite Gruppe großer Joint-Rows (236/237) mit Beträgen bis 2.37 nach Skala. Vermutlich Joint-Slots 0/1 (Beobachtung 306-413) + Chassis-Pose. Verhalten trotzdem Stillstand → Actor-Kopf entscheidet falsch / treibt nicht.

## Segment 2b: Zeilen 328-387 (embed.W 241-300)
- Datum: 2026-11-14, Zeilen 328-387 abgedeckt.
- Kernaussagen: Weitere embed-Rows klein; Ausreißer Row246 (62,-68,...65,...), Row247 (31,-78,65,...-74...), Row254 (74,...173,...), Row255 (-85,53,...64,...-72), Row259 (-45,...,52,...46,-40,-33,52), Row264 (-56,88,...81,61,...75), Row265 (-94,-43,...,53,...). Vermutlich weitere Joint/Positions-Kanäle (Beob. 306+). Gesamtbild: embed hat wenige sehr große Spalten – Netz sieht Joints/Positionen, aber nutzt sie nicht zum Fahren.
- Zitierte Gewichte: Row246: [62,...,-68,...,65,...]; Row254: [...,74,...,...,173,...,-38,...]; Row264: [-56,...,88,...,81,...,75,...]; Row265: [...,-94,-43,...,53,...]. Skala 1e-2.

## Segment 3: Zeilen 388-447 (embed.W 301-360)
- Datum: 2026-11-14, Zeilen 388-447 abgedeckt.
- Kernaussagen: embed.W 301-360 weiter klein (±0.2). Keine neuen Großgewichte. Typisch ±15.
- Zitierte Gewichte: Row301: [4 -3 -8 -13 6 8 ...]; Row310: [5 -1 -7 1 4 4 ...]; alles <0.2. Kontrast zu Rows 54/59/64/236/237/246/254/259/264.

## Segment 4: Zeilen 448-507 (embed.W 361-420)
- Datum: 2026-11-14, Zeilen 448-507 abgedeckt.
- Kernaussagen: embed 361-420 unauffällig klein. Keine neuen Ausreißer. Stützt Diagnose: Perzeption intakt, Fehler liegt tiefer (Actor).
- Zitierte Gewichte: Row361: [1 6 0 16 7 3 ...]; Row380: [...,20,...]; alles ≤0.2.

## Segment 5-Anfang: Zeilen 508-567 (embed.W 421-480)
- Datum: 2026-11-14, Zeilen 508-567 abgedeckt.
- Kernaussagen: embed-Rest weiter klein. Keine neuen Großgewichte; Perzeption scheint über viele Rows verteilt aber moderat.
- Zitierte Gewichte: Row421: [...,18,...]; Row422: [...,15,...,-18,...]; alles ≤0.2 nach Skala.

## Segment 5b: Zeilen 568-627 (embed.W 481-540)
- Datum: 2026-11-14, Zeilen 568-627 abgedeckt.
- Kernaussagen: embed 481-540 klein. Keine Großgewichte. Damit embed.W komplett gelesen (Rows 0-~540+? weiter).
- Zitierte Gewichte: Row481: [...,16,...]; Row487: [...,20,...,-17,...]; alles ≤0.2.

## Segment 5c: Zeilen 628-687 (embed.W 541-600)
- Datum: 2026-11-14, Zeilen 628-687 abgedeckt.
- Kernaussagen: embed 541-600 weiter klein. Bald embed Ende (679 rows).
- Zitierte Gewichte: Row541: [...,16]; Row549: [...,18,9,...]; Row571: [...,22,8,...]; alles ≤0.22.

## Segment 6-Anfang: Zeilen 688-747 (embed.W 601-660)
- Datum: 2026-11-14, Zeilen 688-747 abgedeckt.
- Kernaussagen: embed 601-660 klein. Keine Großgewichte. Nähert sich embed-Ende (679).
- Zitierte Gewichte: Row601: [...,23,...]; Row620: [...,20,...]; alles ≤0.23.

## Segment 6b: Zeilen 748-807 (embed.W 661-677 + embed.b + actor0.W 0-36)
- Datum: 2026-11-14, Zeilen 748-807 abgedeckt.
- Kernaussagen: embed.W Ende (Rows 661-678, Radius/Gravity-Kanäle), embed.b (Skala 1e-4, viele 0, z.B. -288, -255, -242, -232). Dann actor0.W Beginn (Skala 1e-3, große Werte ±300-500): Row0 [...,316,...,-301,...,-312,-340,...,422,180,-394,...], Row1 [...,443,...,-391,...,-483,...], Row5 [...,483,...,-463,...,461...], Row9 [...,486,...,-341,...,-336...], Row21 [...,-573,...,-416...,-395...], Row27 [...,461,...,504,...], Row34 [...,524,...,-409,...,-379,-337...]. Actor0 mischt embed stark – kein einfacher toter Pfad.
- Zitierte Gewichte: actor0 Row0: [-59 -36 81 ... 422 180 -394 ...]; Row21: [-387 ... -573 ... -416 ...]; Row34: [524 ... -409 ... -379 -52 -337]; embed.b: [-117 39 -211 ... -288 ... -255 ... -232 ...].

## Segment 7: Zeilen 808-867 (actor0.W 37-96)
- Datum: 2026-11-14, Zeilen 808-867 abgedeckt.
- Kernaussagen: actor0 weiter dicht mit großen Werten (±0.5). Keine Null-Zeilen; tanh-Netz aktiv. Beispiel Row53 […,-541,…], Row37 […,442,…,-339,…], Row59 […,408,…,-380,…,492,…], Row62 […,-538,…,312,-285,…], Row74 […,557,…,535,…,535…]. Netz hat Kapazität, aber gelernte Abbildung führt zu Stillstand.
- Zitierte Gewichte: Row37: [442 ... -339 ... 370 ...]; Row53: [-541 ... 259 ... -361 ...]; Row59: [408 ... -380 ... 492 ...]; Row74: [557 ... 535 ... 423 ...]. Skala 1e-3.

## Segment 8: Zeilen 868-927 (actor0.W 97-127 + actor0.b + actor1.W 0-23)
- Datum: 2026-11-14, Zeilen 868-927 abgedeckt.
- Kernaussagen: actor0 Rest groß (Row105 [...,495,...], Row116 [...,520,...,-500,...], Row127 [...,454,...,-462,...]). actor0.b Skala 1e-4 mit Ausreißern (945, -589, 586, 692, -544...). actor1.W Beginn groß (±0.7): Row3 [...,590,...,546,...,539...], Row6 [...,754,...], Row11 [...,-562,...,523,...].
- Zitierte Gewichte: actor0 Row116: [...,520,...,-500,...]; actor0.b: [...,945,...,-589,...,586,...,692,...]; actor1 Row3: [590 ... 546 ... 539 ...]; actor1 Row6: [...,754,...,-591,...]. Skalen 1e-3 / 1e-4.

## Segment 9: Zeilen 928-987 (actor1.W 24-83)
- Datum: 2026-11-14, Zeilen 928-987 abgedeckt.
- Kernaussagen: actor1 weiter dicht groß. Beispiele Row24 [...,460,...,460...? 318], Row30 [...,-590,-431,...], Row31 [...,-573,...,610,...], Row40 [...,-575,...,-450...], Row45 [...,-692,...,540,...,529...], Row70 [...,659,...,...], Row81 [...,-547,...,-508,...]. Netz hat durchgehend hohe Gewichte, keine toten Layer.
- Zitierte Gewichte: Row30: [...,-590,-431,...]; Row31: [-573 ... 610 ...]; Row40: [-575 ... -450 ...]; Row70: [...,659,...]; Skala 1e-3.

## Segment 10: Zeilen 988-1047 (actor1.W 84-127 + actor1.b + actor2.W 0-10)
- Datum: 2026-11-14, Zeilen 988-1047 abgedeckt.
- Kernaussagen: actor1 Rest groß: Row84 [...,540,...,512...], Row93 [...,-610,-610...], Row103 [...,566...,-631...,-425,-378...], Row122 [...,736,..., -742...]. actor1.b klein (Skala 1e-3, ±0.17). actor2.W Beginn groß (±0.6): Row3 [...,496,...,496...], Row10 [...,602,...,-482...].
- Zitierte Gewichte: Row93: [...,-610,-610,-396...]; Row103: [...,566...,-631...,-425...]; Row122: [...,736,...,-742...]; actor1.b: [54 -8 22 ... 170 27 ...]. Skalen 1e-3.

## Segment 11: Zeilen 1048-1107 (actor2.W 11-70)
- Datum: 2026-11-14, Zeilen 1048-1107 abgedeckt.
- Kernaussagen: actor2 weiter dicht groß: Row11 [...,-593,...,548...], Row14 [...,-487,...], Row27 [...,-541,...,517,...,...,-491...], Row55 [...,546,...,-540,...], Row68 [...,-825,...,-531,...,-439...], Row69 [...,-574,...], Row70 [...,709,...,626...,-581...].
- Zitierte Gewichte: Row11: [-593 ... 548 ...]; Row24: [...,-541 ... -541 ...]; Row55: [546 ... -540 ...]; Row70: [709 ... 626 ... -581 ...]. Skala 1e-3.

## Segment 12: Zeilen 1108-1167 (actor2.W 71-127 + actor2.b)
- Datum: 2026-11-14, Zeilen 1108-1167 abgedeckt.
- Kernaussagen: actor2 Rest groß, z.B. Row71 [...,478,459...], Row74 [...,-617...], Row77 [...,-525,...,...,-476...], Row80 [...,-552,...,-672...], Row84 [...,-553,...], Row90 [...,658,...], Row93 [...,847 (!) ...], Row98 [...,488,...], Row103 [...,566...,-631...]. actor2.b klein (±0.19, Skala 1e-3).
- Zitierte Gewichte: Row84: [-553 ... -460 ...]; Row90: [...,658,...]; Row93: [...,847,...]; Row80: [...,-672,...]. Skala 1e-3.

## Segment 13: Zeilen 1168-1227 (actor3.W 0-57)
- Datum: 2026-11-14, Zeilen 1168-1227 abgedeckt.
- Kernaussagen: actor3 dicht groß: Row0 [...,370,...,-369...], Row4 [...,-517,...,...,-361...], Row9 [...,-452,...,428...], Row16 [...,539,...,-513...], Row19 [...,552...], Row25 [...,667,...,-442...], Row37 [...,-518,...,...,-494...], Row40 [...,485,...,-418...]. Keine toten Layer.
- Zitierte Gewichte: Row4: [-517 ... -361 ...]; Row16: [539 ... -513 ...]; Row25: [...,667,...,-442...]; Row40: [...,485,...,-418...]. Skala 1e-3.

## Segment 14: Zeilen 1228-1287 (actor3.W 58-117)
- Datum: 2026-11-14, Zeilen 1228-1287 abgedeckt.
- Kernaussagen: actor3 Rest groß: Row64 [...,541,...,-534,...], Row75 [...,486,...,527,...,-584...], Row77 [...,517,...,-526...], Row79 [...,-659,...,416,...], Row85 [...,605,...,-438,...,588...], Row86 [...,545,...,545,...], Row91 [...,-554,...,564...], Row114 [...,516,...,-427...]. Durchgehend aktiv.
- Zitierte Gewichte: Row64: [541 ... -534 ...]; Row75: [486 ... 527 ... -584 ...]; Row79: [-659 ... 416 ...]; Row85: [605 ... -438 ... 588 ...]. Skala 1e-3.

## Segment 15: Zeilen 1288-1347 (actor3.W 118-127 + actor3.b + actor4.W 0-44)
- Datum: 2026-11-14, Zeilen 1288-1347 abgedeckt.
- Kernaussagen: actor3 Ende groß: Row121 [...,484,...,...,-373...,...479...], Row124 [...,620,...], Row126 [...,-639...], Row127 [...,-517,...,509,...]. actor3.b klein (±0.2, Skala 1e-3). actor4 Beginn groß: Row0 [...,-502,...,-528,...,-422...], Row19 [...,637...,-581...], Row32 [...,601...], Row37 [...,619...].
- Zitierte Gewichte: Row121: [484 ... -373 ... 479 ...]; Row127: [...,-517,...,509,...]; actor4 Row0: [...,-502,...,-528,...,-422...]; Row19: [...,637...,-581...]. Skalen 1e-3.

## Segment 16: Zeilen 1348-1407 (actor4.W 45-104)
- Datum: 2026-11-14, Zeilen 1348-1407 abgedeckt.
- Kernaussagen: actor4 dicht groß: Row45 [...,-501,...,447,...], Row49 [...,555,...,...,521,...], Row58 [...,614,...,-497...], Row73 [...,-687,...,449...], Row80 [...,610...,-457...], Row99 [...,-549,...], Row103 [...,576,...,-533...], Row104 [...,-553,...,448,...].
- Zitierte Gewichte: Row49: [...,555,...,...,521,361...]; Row58: [...,614,...,-497...]; Row73: [...,-687,...,449...]; Row80: [...,610...,-457...]; Skala 1e-3.

## Segment 17: Zeilen 1408-1467 (actor4.W 105-127 + actor4.b + actor_head.W 0-31)
- Datum: 2026-11-14, Zeilen 1408-1467 abgedeckt.
- Kernaussagen: actor4 Ende groß (Row108 [...,717...], Row121 [...,-541,...], Row124 [...,-609,...,578...], Row125 [...,-563,...,-554...]). actor4.b klein (±0.27). actor_head.W Beginn Skala 1e-3: Row0 [-244 344 -181 ...], Row1 [-376 431 -162 ...], Row2 [-267 -19 706 ...], Row6 [683 -638 -5 ...], Row13 [-274 60 467 ...], Row14 [230 38 -542 ...], Row15 [222 -446 468 ...]. Kopf unterscheidet scharf zwischen Entry0=0/1/2 – entscheidend für Fahrtrichtung. Trajektorie zeigt Stillstand trotz Logit-Unterschieden → Verdacht: Logits für Entry0 balanciert/gegenläufig oder Drive-Richtung falsch (vorwärts vs rückwärts), sodass Auto an Rampe (Shape6 bei 2.227) hängen bleibt.
- Zitierte Gewichte: actor4 Row108: [...,717,...]; Row124: [...,-609,...,578...]; actor_head Row2: [-267 -19 706 ...]; Row6: [683 -638 -5 ...]; Row14: [230 38 -542 ...]. Skalen 1e-3.

## Segment 18 (final): Zeilen 1468-1566 (actor_head.W 32-127 + actor_head.b)
- Datum: 2026-11-14, Zeilen 1468-1566 abgedeckt. Damit 1-1566 vollständig.
- Kernaussagen: actor_head Rest entscheidend für Entry0 (Spalten 0,1,2): Row35 [-266 17 548 ...] stark pro Wert 2 (Drive -1); Row36 [461 -466 -37 ...] stark pro Wert 0 (Motor aus) vs 1; Row37 [459 -476 227 ...]; Row43 [465 -478 60 ...]; Row48 [400 -329 -87 ...]; Row49 [-504 520 -243 ...]; Row54 [-224 -124 838 ...] extrem pro 2; Row57 [546 -669 150 ...] extrem pro 0; Row69 [511 -508 13 ...]; Row76 [531 -540 89 ...]; Row79 [-235 -18 656 ...]; Row83 [-249 -139 985 ...] extrem pro 2; Row115 [-145 -157 730 ...]; Row118 [-422 45 908 ...] extrem pro 2. Kopf kann alle drei Werte stark ansteuern – kein toter Logit. actor_head.b Skala 1e-3: [31 52 -194 2 -23 25 ...] → Bias für Entry0: [0.031, 0.052, -0.194] leicht gegen Wert 2.
- Diagnose-Verdacht bestätigt: Netz fährt anfangs (Steps 0-16 bewegen sich), bleibt dann stehen (x~2.0) vor Rampe/Block Shape6 und rotem Shape7. Entry0 oszilliert zwischen 0 (frei) und 1 (+1), nie 2 (-1). Vermutlich falsche Drehrichtung: +1 dreht Räder so, dass Auto gegen Rampe drückt statt darüber; -1 (nie gewählt, Bias -0.194 + Head-Hemmung) wäre nötig zum Überklettern/Rücksetzen. Fix: Entry0-Policy auf dauerhaft vollen Antrieb in EINE Richtung zwingen (beide Motoren gleiche Binding 0), stochastische Stillstands-Falle brechen.
- Zitierte Gewichte: Row54: [-224 -124 838 ...]; Row57: [546 -669 150 ...]; Row83: [-249 -139 985 ...]; Row118: [-422 45 908 ...]; actor_head.b: [31 52 -194 ...] (×1e-3).
