# 01 — Pipeline architecture

Every system in this space, when you strip vocabulary differences, has the same conceptual stages. Understanding them makes choosing tools easier.

## The stages

```
[Source] → [Acquisition] → [Parsing] → [Structuring] → [Planning]
                                                          ↓
                                                  [Annotation/IR]
                                                          ↓
                                                    [Synthesis]
                                                          ↓
                                                  [Stream/Cache]
                                                          ↓
                                                   [Playback]
                                                          ↑
                                                  [Control surface]
```

### Acquisition

Where does the input come from? Pasted text, file, URL, clipboard, browser selection, document already open in another app. Each source has a different "front door" and different metadata available. Don't conflate them.

### Parsing

Source-format-specific. Markdown → AST. HTML → DOM. PDF → text + layout heuristics. The output is a *typed tree* of structural elements. **Type information matters more than content for the next stage.**

For HTML specifically, a heuristic extractor like Trafilatura is still SOTA; neural extractors lose to heuristics on this benchmark. ([trafilatura summary](../articles/trafilatura_paper_summary.md))

### Structuring

Take the parser output and normalize across formats. A heading is a heading whether it came from `# H1` or `<h1>` or a PDF heuristic. The downstream system shouldn't care about input format.

This stage emits an internal representation — a sequence of typed elements with hierarchical position. Most reasonable systems will end up with something like:

```
Section(level=1, title="...")
Paragraph(text="...", references=[...])
List(ordered=False, items=[...])
CodeBlock(lang="python", text="...")
Quote(source=..., text="...")
Figure(caption="...", url="...")
...
```

Choice of internal IR is a choice. The developer should justify it explicitly: this is the shape the rest of the system will commit to.

### Planning

The hard stage. Takes the structured stream and emits **prosody intent** — what the speech should *do* at each point, expressed in a form the synthesis layer can consume.

This is its own topic. See `03_prosody_planning.md`.

The output of planning could be:
- Plain prose enriched with marker tokens.
- SSML-like XML.
- A natural-language description-per-segment (for Parler-style engines).
- A typed IR that the synthesis layer interprets.

There is no industry-standard answer. ([survey paper 2412.06602](../papers/2412.06602_controllable_speech_synthesis_llm_era_survey.pdf) maps the space.) **This is the most consequential design choice.**

### Synthesis

Convert annotated text → audio samples. Many possible engines (see `04_synthesis_engines.md`). Critical properties to evaluate:

- Honors the IR the planner emits (or can be wrapped to).
- Produces consistent voice across calls.
- Acceptable latency.
- Runs locally.

### Stream / cache

The synthesizer's output is streamed to the user, not held in memory until done. Sentence-or-paragraph chunks ([Deepgram chunking notes](../articles/deepgram_text_chunking_for_streaming_tts.md)). Cache aggressively — re-reading the same passage shouldn't re-synthesize.

### Playback

Audio output, with controls. Treat as its own component, even if it's just a wrapper around a system audio API initially. Decoupling it makes "headless mode" (synthesize-to-file) cheap to add. The control surface (play/pause/jump/speed/repeat/stop) is exposed as a programmatic API; the UI layer that surfaces those controls is owned outside this repo.

### Control surface

User actions: play, pause, jump, slow down, speed up, repeat last, switch voice, "what was that?", "spell that name". Some of these alter playback only; some have to flow back to planning (e.g. "read this section like a story" might re-plan). Decouple cleanly.

## Concurrency model

These stages don't run sequentially. A working system has:

- **Acquisition + parsing**: synchronous on user action.
- **Structuring**: synchronous, fast.
- **Planning**: streaming. Plan section N while section N-1 is being synthesized.
- **Synthesis**: streaming. Synthesize chunk M while chunk M-1 is being played.
- **Playback**: continuous, decoupled buffer.

The audio buffer is what keeps everything else hidden from the listener. If the buffer underruns, the listener hears it. Buffer-management is therefore a top-level concern, not an afterthought.

## Coupling rules

The mistake to avoid: tying the planner to a specific synthesizer's IR. If the planner emits SSML, swapping in a model that wants natural-language descriptions becomes a rewrite.

**Decouple via a typed internal IR.** Adapters convert from IR → SSML or IR → Parler description per synthesizer. The IR is the contract; everything below it is replaceable.

## Storage

The system has two persistent stores worth thinking about:

- **Document cache** — parsed and structured documents, so re-opening a file is instant.
- **Audio cache** — synthesized audio per (document hash, voice profile, chunk). Lets the user replay sections without re-synthesis cost.

These are both "obvious" but easy to forget. Plan for them at architecture time.

## What deliberately isn't in this diagram

- **A "model" box.** There can be multiple models — one for parsing, one for prosody planning, one for synthesis. There may be zero models in some boxes (Trafilatura is heuristic). Don't pre-commit to "the model" as a singleton.
- **A specific service boundary.** Whether this runs in-process, as separate services, or as a library is a deployment choice, not an architecture choice. The pipeline is the same.
- **A specific async runtime.** That's a language/framework choice. The pipeline doesn't care.
