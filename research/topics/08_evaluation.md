# 08 — Evaluation

If "sounds good" is the bar and there are no objective tests, the system rots silently. Build the evaluation harness before the system is "done."

## Subjective vs objective

Both have real uses; neither alone suffices.

### Subjective

- **MOS (Mean Opinion Score).** 1–5 naturalness rating from human listeners. Industry default. Expensive at scale, cheap at small scale.
- **CMOS (Comparative MOS).** Listener picks A vs B on a 7-point preference scale. Sesame's eval ([article](../articles/sesame_crossing_uncanny_valley_of_voice.md)) uses this. More sensitive than MOS for "is X better than Y."
- **Long-listening test.** Have a human listen for 20+ minutes. Note when they checked out, when they got annoyed, when something jarred them. Hard to quantify but uniquely revealing.

### Objective

- **WER (Word Error Rate).** Run output through ASR; compare to input. Catches mispronunciations, dropped words, hallucinations. Cheap. Mandatory.
- **MCD (Mel-Cepstral Distortion).** Spectral distance to a reference recording. Useful for *cloning* fidelity, less useful for general quality.
- **F0 RMSE.** Pitch contour error vs reference. Useful when you have ground-truth recordings.
- **TTSDS** ([2407.12707](../papers/2407.12707_ttsds_evaluation.pdf)). Distribution-distance metric. Catches mode collapse and prosody flattening that MOS often misses.
- **Prosody-specific** ([2509.20485](../papers/2509.20485_objective_prosody_intelligibility_evaluation.pdf)). 2025 paper proposing better correlation with human judgment.

## What to measure for *this* product

Reading-mate is unusually long-form. Build the eval around that.

### Tier 1: regression-prevention

Run on every PR / nightly build:

1. **WER on a fixed corpus.** 10–20 documents (varied: technical, fiction, news, code, mixed). Synthesize, ASR, compare. Threshold: WER doesn't get worse than baseline.
2. **Latency.** TTFA and RTF on the same corpus. Threshold: TTFA stays under 500ms, RTF under 0.5.
3. **Voice-consistency check.** Synthesize the same document twice. Compute embedding distance per chunk. Threshold: variance under a calibrated value.

These run automatically. They catch *regressions*. They don't tell you if the system is good.

### Tier 2: quality assessment

Run periodically (weekly, before releases):

1. **CMOS vs reference.** Compare current build to last release on 10 documents. Aim for non-negative preference.
2. **Long-form drift study.** Synthesize a 30-minute document. Spot-check 5 segments at minutes 1, 7, 15, 22, 28. Compute embedding distance from minute 1. Drift > threshold = regression.
3. **Specific failure mode probes.** Curate a set of "things the system shouldn't do." Mispronouncing common names. Reading code character-by-character. Cutting off final words. Each probe is a documented document + expected behaviour.

### Tier 3: human listening

Run before major releases:

1. **The 20-minute test.** A human listener listens to a 20-minute synthesis end-to-end. After, they annotate where they noticed *anything*. The signal is denser at the start; fatigue effects are real.
2. **Comparison listen.** Same document on a competitor (Speechify, ElevenLabs). Same human notes preference and reasons.
3. **Open feedback channel.** From any user, ever. Logged and triaged.

## The corpus

A small, fixed, representative corpus is more useful than a large, varying one. Suggested 10:

1. A markdown technical blog post (~1500 words, with code and emphasis).
2. A scientific paper abstract + intro (~800 words, dense with abbreviations).
3. A short fiction excerpt (~1000 words, dialogue and narration).
4. A news article (~600 words, quotations).
5. A list-heavy how-to article (~800 words).
6. A page with named entities (people, companies, places).
7. A page with numbers, dates, prices.
8. A page with a non-English term (graceful degradation test).
9. A long narrative (~5000 words, drift test).
10. A pathological page: nested lists, math, code, footnotes.

Version-control the corpus. Re-running synth on a stable corpus is the only way to know if the system improved between releases.

## The "presence" axis

Sesame's framing — that the goal is *presence*, not just naturalness — implies an axis that the standard metrics don't capture. There isn't a great objective measure for presence. Closest proxies:

- **Listener engagement length.** How long do listeners listen before pausing or switching off? Self-reported, voluntary.
- **CMOS in *contextual* settings.** Per Sesame's protocol: provide 90 seconds of context before the test sample. Without context, modern TTS is at parity with human; with context, the gap shows up. ([sesame article](../articles/sesame_crossing_uncanny_valley_of_voice.md))

For reading-mate specifically, design CMOS tests where listeners hear:

> Sample A: minutes 5–6 of a 30-minute synthesis.
> Sample B: minutes 5–6 of a 30-minute human reading.

If listeners can't tell at minute 5, you're past the bar. If they can, you have a target.

## Don't:

- Rely solely on MOS. It plateaus; new models all score 4.0+ and you can't tell which is actually better.
- Compare engines on different corpora. One sentence isn't enough; ten different sentences across systems isn't apples-to-apples. Pin the corpus.
- Believe the engine vendor's metrics without re-running them.
