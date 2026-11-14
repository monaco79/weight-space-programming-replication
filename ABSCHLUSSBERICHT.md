# Abschlussbericht 14.11.2026: Können auch andere LLMs Gewichte lesen/schreiben?

Frage: Replikation von „Sparks of Weight-Space Programming" (Takehi et al., MIT),
Fokus: liest/schreibt auch ein anderes LLM als Astra Gewichte? Modelle: Muse Spark 1.3,
DeepSeek V4.1 Flash, Gemini 4 Argon. Setup: MacBook Air M1 8GB, Platte 98% voll.

## Exp. 1 – Spiel aus Gewichten erkennen: 12/12 ✅ (komplett)

4 PPO-Netze (17.253 Gewichte, 115KB Prompt, bei Astra ~73k Tokens), open-answer, keine Tools.

| Modell | Breakout | Pac-Man | Pong | Freeway |
|---|---|---|---|---|
| Astra (Ref.) | ✅ 99% | ✅ 96% | ✅ 96% | ✅ 97% |
| Muse Spark 1.3 | ✅ 88% | ✅ 85% | ✅ 92% | ✅ 88% |
| DeepSeek V4.1 Flash | ✅ 93% | ✅ 78% | ✅ 90% | ✅ 86% |
| Gemini 4 Argon | ✅ 98% | ✅ 99% | ✅ 98% | ✅ 97% |

Alle Antworten in `runs/antwort-<modell>-<spiel>.md`. Alle Begründungen zitieren
Einzelgewichte und rekonstruieren Paddle/Ball/Bricks, Labyrinth/Geist/Pellets,
Paddle-links/Torlinie, 6 Verkehrsspuren + Ziellinie oben. Tiefpunkt: DeepSeek
Pac-Man 78% (richtig, aber unsicher). Lesen ist kein Astra-Exklusiv.

## Exp. 4 – Car-Fix: lesen ja, schreiben mit Lücke (komplett, qualitativ)

Prompt 578KB / 171.664 Gewichte (bei Astra 355k Tokens). Echte Wertung (Kinetix-
Simulator, 1024 Episoden) auf dem Air nicht möglich; Wertung vs. Astra-Referenz
(Diagnose + Edit-Plausibilität; Astras Fix: 51 Statements, konditionaler Pfad mit
Rückwärts-Anlauf, 1024/1024).

| Modell | Lesen | Diagnose | Fix |
|---|---|---|---|
| Spark | ✅ (gechunked, Retry) | Stop-Policy, Fehler im Head | 2 Stmts, Bias +1 |
| DeepSeek | ✅ (gechunked, Retry) | detaillierteste (Spalte 2 gesättigt, p=0, Pos. (2.00,0.92,−0.27)) | 2 Stmts, Bias +1 |
| Gemini | ✅ (ein Turn) | Oszillation an Rampe | 3 Stmts, Bias links |

Erstversuch Spark + DeepSeek: Kontext-Overflow → kein Fenster-Problem (Retry mit
6-Segment-Protokoll + Notizen las alles), sondern Verfahrensabbruch. Aber: Retry
heißt externes Gedächtnis, das Astra nicht hatte. Alle drei Fixes = grobes
„Vollgas"-Muster ohne Anlauf-Logik und Konditionen – als Programme eine andere
Klasse als Astras 51er-Pfad. Befund: Lesen replizierbar, Schreiben mit Lücke.
Fairness-Notiz: Prompt war bereits getunt (Trial-Runs unveröffentlicht) – eher
Vorteil für unsere Modelle; Exp. 6/8 (iterativ mit Feedback) wurden nicht getestet.

## Infrastruktur

- `logs/`: Sparse-Clone (exp1/exp4/exp8/harness). `runs/`: Prompts + 21 Antwort-/Notizdateien.
- JAX-Probe: `jax[cpu]` importiert, CpuDevice ok. Aber: alte Version (meldet 0.12.0.1,
  pip 23.2.1) – vor Kinetix pip upgraden + neues jax. Nächster Schritt (2–4 Tage):
  Kinetix installieren, Level laden, Base 0/256 + Astra-Edit ~1024/1024 reproduzieren,
  dann unsere 3 Fixes scoren.
- Lokal-LLM (`llama-cli` + Qwen3-0.6B-Q4): hängt beim Modell-Laden (Server-Modus,
  52min CPU-Spin). Offen: `-ngl 0` / älterer Build.
- Hinweis: Qwen3-0.6B-Q4-GGUF ≠ Qwen3.5-0.8B-bf16 aus Exp. 8 – p50c-Diff nicht übertragbar.

## Offen (nicht heute)

Exp. 7 (Sykophantie), Exp. 8-Harness (GPU), Teil B Qwen-Loop-Diagnose, llama-Fix.
