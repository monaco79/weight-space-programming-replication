# Teil A – Durchführung

## Material (fertig in `runs/`)
- `exp1-breakout-a9f535cc-prompt.txt` → Lösung: Breakout (Astra: 99%)
- `exp1-pacman-bdf99279-prompt.txt` → Lösung: Pac-Man (Astra: 96%)
- Je ~115KB / ~73k Tokens. Unverändert per Copy-Paste an jedes Editor-LLM geben.

## Protokoll pro Lauf
1. Neuer Chat / frische Session, keine Hinweise aufs Spiel geben.
2. Gesamte Datei als eine Nachricht schicken.
3. Antwort ablegen: `runs/antwort-<modell>-<spiel>.md`.
4. Werten: `Game:`-Zeile → Spiel erkannt? Confidence notieren.

## Ergebnis-Tabelle

| Modell | Breakout | Pac-Man | Pong | Freeway |
|---|---|---|---|---|
| Astra (Ref.) | ✅ 99% | ✅ 96% | ✅ 96% | ✅ 97% (Frogger=Freeway) |
| Muse Spark 1.3 | ✅ 88% | ✅ 85% | ✅ 92% | ✅ 88% |
| DeepSeek V4.1 Flash | ✅ 93% | ✅ 78% | ✅ 90% | ✅ 86% |
| Gemini 4 Argon | ✅ 98% | ✅ 99% | ✅ 98% | ✅ 97% |

Erfolgskriterium: mind. 1/2 richtig MIT Gewichtsbegründung
(z.B. Breakout: Channel-0-Gewichte in unterer Reihe = Paddle;
Astra-Referenzantworten in `logs/exp1_which_game/runs/*/answer.md`).

## Teil B – Qwen lokal (Status: blockiert, Alternativplan)`llama-cli` (Build 10985) startet im Server-Modus und hängt beim
Modell-Laden (52min CPU-Spin, 90s-Test kam nicht über Metadaten-Dump hinaus).
Mögliche Ursachen: Metal-Compile-Loop, Memory-Pressure (8GB RAM, Platte 98% voll).
Nächste Schritte wenn Zeit: `-fit off`, `-ngl 0` (CPU-only), oder älteres
llama.cpp-Release bauen. Blockiert Teil A NICHT.
Fallback für Teil B: Loop-Diagnose per API-Modell statt lokal
(kleine MATH-Prompts an Qwen via HF Inference mit temp 0.6, Loop-Rate zählen).

## Exp. 4 – Car-Fix (578KB Prompt, bei Astra 355k in / 10k out)

| Modell | Gelesen | Diagnose | Edit | Bewertung |
|---|---|---|---|---|
| Astra (Ref.) | voll | Throttle/Coast-Controller, Reverse nie gesampelt | 51 Statements, konditionaler Pfad (Rückwärts-Anlauf, Vollgas ab x=3.8, aus nach Grün>0.95), 1024/1024 ✅ | Referenz |
| Muse Spark 1.3 | ✅ voll – aber nur gechunked (6 Segmente + Notizen, Retry) | Stop-Policy: oszilliert aus/+1, nie -1; Fehler im Actor-Head | 2 Statements: Head-Spalten nullen, Bias auf +1 | Diagnose plausibel, Fix wie Gemini ohne Anlauf/Kondition – unverifizierbar |
| DeepSeek V4.1 Flash | ✅ voll – aber nur gechunked (Retry, Notizen) | Head-Spalte 2 (rückwärts) gesättigt/fehlverdrahtet, p=0 überall; Auto pendelt 0/1, Timeout | 2 Statements: Head nullen, Bias +1 (p≈0.99995) | Diagnose detailliertste von allen, Fix gleiches Muster wie Spark/Gemini |
| Gemini 4 Argon | voll (~200-220k tok) | Risikoaverses Optimum, Oszillation vorwärts/aus an Rampenspitze | 3 Statements: act0+act2-Spalten nullen, Bias auf links | Diagnose teils richtig, Fix unplausibel (kein Rückwärts-Anlauf, keine Kondition) – ohne Simulator nicht verifizierbar |
