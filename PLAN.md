# 1-Tages-Plan: Weight-Space-Programming mit M1 8GB

Stand: 14.11.2026. Constraints: MacBook Air M1 8GB, Platte 98% voll (12GB frei),
lokal vorhanden: `qwen3-0.6b-q4_0.gguf` (409MB, Q4) + llama.cpp Build (`llama-cli`).
Kein vLLM/CUDA, kein torch. Editor-LLMs: 2-3 im Vergleich via API.

Repo-Basis (sparse clone): `logs/` mit exp1, exp4, exp8/code, harness.

## Teil A – Read-Test, kein lokales ML nötig (vormittags, ~3h)

Portabelster Teil des Posts. Exp.1: 17k-Gewichte-Netz als Text (~113KB),
Prompt `logs/exp1_which_game/runs/<game>/prompt.txt` (~115KB, ~73k Tokens).
Gleiche Datei unverändert an 2-3 Editor-LLMs schicken, Antwortformat aus Prompt
erzwingen (`Game:`-Zeile). Scoring wie `eval/answer_key.json` per Keyword.

Läufe (je LLM 2 Netze statt 4, um Kosten zu decken):
1. `a9f535cc` (= Breakout), 2. `bdf99279` (= Pac-Man).
Referenz Astra: 4/4, 96-99% confident, 71-107s, ~73k in / ~1.8k out Tokens.

Ergebnis-Tabelle: pro LLM x Netz: geratenes Spiel, Confidence, Kosten/Tokens.
Erfolg = mind. 1/2 richtig mit nachvollziehbarer Gewichtsbegründung
(z.B. Breakout: Channel-0-Gewichte unten = Paddle).

Danach optional Exp.4: `exp4_fix_car` Prompt + `ppo_final_seed0.npz` vs.
`edited_actor.npz` als Lesetest (Erklärung des Fixes, kein Ausführen).

## Teil B – Qwen 0.6B lokal, GGUF statt safetensors (~3h)

Ehrliche Einschränkung: Qwen3-0.6B-Q4-GGUF ≠ Qwen3.5-0.8B-bf16 aus Exp.8.
Der p50c-Diff (Layer 15/19 `down_proj`) ist NICHT übertragbar (andere Architektur,
andere Layerzahl, Q4-Quant). Teil B testet daher die Diagnose, nicht den Edit:

1. Smoke (läuft bereits im Hintergrund): `llama-cli -m qwen3-0.6b-q4_0.gguf`,
   je 3 Mathe-Prompts (`2^10+24`, GSM8K-Stil), temp 0.6 vs 0.0, `-n 300 -c 1024`.
   Messen: Loop-Rate (gleiche Phrase 3x+ / Abbruch bei `-n`), Antwort korrekt ja/nein.
2. Scripted-Kontrolle (ohne LLM): gleicher Prompt mit `--repeat-penalty 1.2`
   bzw. presence-penalty. Zeigt: Loop ist sampler-seitig mitigierbar.
3. LLM-Anteil: einem der Editor-LLMs aus Teil A die (dequantisierten) `down_proj`-
   Statistiken von 2 Layern geben (Mittel/Varianz pro Spalte, kein Full-Dump:
   GGUF + 8GB RAM geben keinen 32k-Kontext her) und fragen, welche Spalte es
   als Loop-Richtung entfernen würde. Nur Vorschlag protokollieren, NICHT anwenden
   (GGUF-Requant an einem Tag zu fehleranfällig).

Erfolg = Loop-Diagnose reproduziert (Modell loopt / loopt nicht, mit Zahlen),
plus dokumentierter Editor-Vorschlag + Random-Baseline.

## Was wir NICHT tun (bewusst)

- Keine volle MATH-500/GSM8K/IFEval-Eval (Sonnet: ~240 Mio. Tokens, ~1 Tag allein).
- Keinen 50-Pass Agent-Loop, keinen Forward-Server-Bau.
- Kein Finetuning (Exp.7-Stil) – auf 8GB + 12GB frei unrealistisch.
- Kein GGUF-Rewrite des Gewichts-Edits (Quant-Risk).

## Offene Punkte an dich

1. Welche 2-3 Editor-LLMs (API-Keys vorhanden? z.B. Claude/GPT/Lokales)?
2. Max. Token-Limit pro Prompt ok (~73k in)? Sonst auf 1 Netz kürzen.
3. Darf `logs/` so sparsam bleiben (nur exp1/exp4/exp8/harness) wegen Platte?
