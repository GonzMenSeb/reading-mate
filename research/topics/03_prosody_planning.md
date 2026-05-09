# 03 — Prosody planning

This is the part of the system that doesn't exist as an open-source artifact you can drop in. It's also the part where most of the system's perceived intelligence lives.

**The job:** consume the structured stream from parsing, emit prosody intent that the synthesizer can faithfully execute.

## What "prosody intent" means

Prosody is the suprasegmental information of speech: pitch contour, duration, pauses, emphasis, rhythm. It's what carries *how* a sentence is said, separate from *what* is said.

Document structure is correlated with prosody but not identical to it. The mapping is what the planner has to learn or encode.

Examples:

| Structural cue | Prosodic intent |
|---|---|
| Heading | Brief upward inflection, clear pause after |
| New paragraph | Pitch reset, slightly longer pause than within paragraph |
| List item | Slight rise at end (continuation), lower at last item |
| Em-dash interruption | Quick aside, faster, slight pitch lift |
| Blockquote | De-emphasis, slight tonal shift, signal "this is reported speech" |
| Code block | Slow, deliberate, optionally summarized |
| Bold / emphasis | Slightly louder + slower on the run |
| Em-dash continuation | Held breath before the next clause |
| Question | Rising terminal contour |
| Footnote pointer | Pause + de-emphasized aside *or* skipped |

These are folk rules. Empirical analysis of professional audiobook narration ([2310.06930 Prosody Analysis of Audiobooks](../papers/2310.06930_prosody_analysis_audiobooks.pdf)) shows narrators do roughly this, with massive individual variation.

## Three families of approach

### A. Rules + SSML

The simplest shape: a deterministic mapping from element types to SSML tags. Works today with engines that respect SSML (Piper, espeak-ng, RHVoice, Mimic 3).

**Pros:** Predictable. Auditable. No model dependencies. Cheap.
**Cons:** Modern neural engines (Kokoro, CosyVoice, Parler) ignore most or all SSML. The expressivity ceiling is low. The user instinct that "the formatting is linguistic information" is honoured only crudely.

The SSML-French paper ([2508.17494](../papers/2508.17494_ssml_prosody_french.pdf)) shows MOS improvements (3.20 → 3.87) when SSML is used carefully with neural TTS — so this is not nothing. But it's not the sci-fi answer.

### B. LLM-driven natural-language description

The planner is a language model that, given the structured stream + a small style policy, emits per-segment natural-language descriptions of how to speak. Send those to a Parler-style synthesizer.

**Pros:** Expressive. Composable. Plays naturally with the rest of the LLM stack.
**Cons:** Slow if not careful. A description per paragraph multiplies token count. Latency-sensitive.

Engines this targets: Parler-TTS, IndexTTS2, CosyVoice 2/3, Qwen3-TTS — anything that takes a description as conditioning input. ([Parler-TTS overview](../articles/parler-tts_huggingface_overview.md), survey paper [2412.06602](../papers/2412.06602_controllable_speech_synthesis_llm_era_survey.pdf))

The description-per-segment approach has a known risk: voice discontinuity at description boundaries. Test this empirically; don't assume it's smooth.

### C. End-to-end LLM-coupled synthesis

Skip the intermediate IR. The synthesis model itself reads the document and emits audio, conditioned on enough context to produce coherent prosody.

This is where the field is going. Sesame's CSM, Long-Form SLM ([2412.18603](../papers/2412.18603_long_form_speech_generation_slm.pdf)), Audiobook-CC ([2509.17516](../papers/2509.17516_audiobook_cc_controllable.pdf)) are precedents.

**Pros:** Highest ceiling for naturalness. No human-designed mapping to debug.
**Cons:** Largest models, hardest to run locally, hardest to control deterministically. State of the art changes every quarter; building on it is a moving target. Most of the open-source models in this category are not yet good enough to ship as a default narrator.

## A practical hybrid

The configuration most likely to work today, **as one possible starting point**, is hybrid:

1. **Rule-based structural prosody** — paragraph pauses, heading inflection, list pacing — emitted as a typed IR.
2. **LLM-driven style annotations** — per-section natural-language description of the intended register ("explain this calmly," "this is a story segment, narrate it"), informed by the surrounding text.
3. **Synthesizer that consumes both** — either an SSML-honouring engine for the structural part *plus* a description-driven engine for the stylistic part, or a single engine that can be wrapped to honour both.

The developer should be skeptical of this recommendation. Test before committing.

## The intermediate representation

Whatever the planner emits, write it down. The IR is the contract between planner and synthesizer; everything depends on it.

Properties a good IR should have:

- **Typed.** A pause is not a fragment of XML; it's a `Pause(duration_ms=350, reason="paragraph_break")`.
- **Composable.** A heading-with-emphasis is a Heading containing emphasized runs, not a flat string.
- **Round-trippable.** You should be able to serialize, deserialize, and produce the same audio. Necessary for caching and for testing.
- **Engine-agnostic.** Adapters convert IR → SSML or IR → Parler descriptions per engine.

A reasonable shape:

```
Utterance(
    voice_profile_id="narrator-1",
    style_description="calm, deliberate, slightly warm",
    runs: [
        TextRun(text="...", emphasis="none"),
        Pause(duration_ms=200, reason="comma"),
        TextRun(text="...", emphasis="strong"),
        Pause(duration_ms=400, reason="paragraph"),
    ]
)
```

This is a sketch, not a spec. The spec is yours to design.

## What to read first

If the developer reads exactly two things to understand prosody planning:

1. The 2024 survey: [2412.06602](../papers/2412.06602_controllable_speech_synthesis_llm_era_survey.pdf). Maps the whole landscape.
2. Sesame's article: [research/articles/sesame_crossing_uncanny_valley_of_voice.md](../articles/sesame_crossing_uncanny_valley_of_voice.md). Reframes the goal in a useful way — it's not about the prosody, it's about *presence*.

Read the audiobook-prosody paper ([2310.06930](../papers/2310.06930_prosody_analysis_audiobooks.pdf)) before deciding what good prosody looks like. Empirical grounding in human narration prevents over-engineering.

## A suggested experiment

Before committing to an approach, run a small study:

1. Take 5 markdown documents of varying type (essay, technical doc, fiction excerpt, news article, code-heavy tutorial).
2. Hand-author "ideal" SSML or hand-author "ideal" Parler descriptions per segment.
3. Synthesize with the candidate engine.
4. Listen. Compare to plain-text-to-engine output.

If the gap between hand-authored and plain is small, the planner won't matter much; the engine is the bottleneck. If the gap is large, you've found the lever and the rest of the project is figuring out how to *automate* what you did by hand.

This experiment is the cheapest way to avoid building the wrong thing. Do it before writing any planner code.
