# 09 — Open questions

Things the field has not solved. Things this corpus does not contain answers to. Things the developer should expect to *engineer their way around*, not look up.

## How to map document structure to prosody, generally

There is no published reference mapping from "markdown element type" to "prosodic intent." The audiobook-prosody paper ([2310.06930](../papers/2310.06930_prosody_analysis_audiobooks.pdf)) gives empirical patterns from professional narrators, but it's descriptive, not prescriptive.

You will end up writing this mapping by hand. That's fine. Document it as the system grows.

## How well any specific neural TTS engine honours SSML

Vendor claims and reality diverge. Test empirically — synthesize a document with and without SSML, listen, decide. Don't trust the README.

## Whether description-driven TTS is actually better than rule-based SSML for this use case

The natural-language description paradigm (Parler, IndexTTS2, etc.) is *theoretically* more expressive than SSML. Whether that expressiveness translates to better experience for a user reading a markdown document is an empirical question with no published answer.

The cheapest way to find out: hand-author both for the same document, A/B them.

## How to handle code in spoken form

Reading code character-by-character is not useful. Reading code word-by-word ("function authenticate user") is *sometimes* useful. Skipping code is sometimes useful. The right behaviour depends on user intent, which the system doesn't know.

Possible defaults:

- Speak the language and a one-line summary: "Python function: authenticate user."
- Speak the function/class signature only.
- Skip entirely if it's a verbatim block.

There is no consensus. Treat this as a knob the user controls.

## How to handle math

LaTeX → spoken English ("integral from zero to infinity of x squared d x") is a small NLP project. There are libraries (MathSpeak, SRE) with mixed quality. Worth a separate evaluation pass if the user reads scientific content.

## How to handle footnotes and references

Inline-read disrupts flow. Skip-and-collect breaks the reading experience. Read at end of paragraph is jarring. Read on user request is the safest default but has UX cost.

User study, not engineering, will resolve this. Pick a default; let the user override.

## Pronunciation of uncommon words

The model doesn't know how to pronounce names of people, places, or domain-specific jargon. It will hallucinate something. The user has to be able to fix it, easily, in-flow.

Per-document and per-user pronunciation dictionaries solve this *if* the UX makes corrections cheap. ([07_ux_and_controls.md](07_ux_and_controls.md) discusses the UX.)

## Multi-language documents

A markdown post that's mostly English with a few French phrases. Most TTS engines either butcher the French or speak it as English. Detection + per-span language switching is plausible but adds complexity. Defer until the user actually needs it.

## Long-form coherence

The deepest open problem. Codec-LM models drift after ~1 minute of synthesis. Description-driven models flatten over time. Sesame's CSM addresses some of this but is conversational-focused.

[Long-Form SLM 2412.18603](../papers/2412.18603_long_form_speech_generation_slm.pdf) and [Long-Context Speech Synthesis 2508.14713](../papers/2508.14713_long_context_speech_memory.pdf) describe ongoing research. None of it is an off-the-shelf solution.

Workarounds:

- Re-prime the model with a fresh speaker reference every chunk.
- Synthesize in shorter chunks with crossfades.
- Use a voice profile + named speaker to anchor identity.

These are fixes for symptoms. The underlying problem is open.

## The "presence" gap

The Sesame finding — that listeners can't tell modern TTS from human in zero-context tests, but consistently prefer human in contextual tests — defines the actual quality bar. Closing that gap is the open research problem of the next few years. Don't expect to solve it; expect to track it.

## What the developer should *not* try to solve here

This list is also an *avoid* list. Reading-mate doesn't need to solve any of these to be good. Avoid scope creep into research projects. The system can be excellent while standing on top of imperfect components, *if* it composes them deliberately and degrades gracefully when they fail.
