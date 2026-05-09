# reading-mate — broad requirements

These are *capability* requirements. They describe what the system must be able to do, not how. Choices about language, framework, models, and architecture belong to the developer who plans and implements.

If a requirement here forces a particular tool, treat it as a bug in the requirement, not a hint about the implementation. Push back.

**Scope note.** This document owns *system* requirements — the engine, the pipeline, the synthesis quality, the runtime surface. UI requirements (visual layout, interaction patterns, what the user sees and touches) are owned separately and supplied to you alongside this brief. The system is responsible for *exposing the right capabilities* so any UI can wire to them; what the UI looks like is not specified here.

## Vision (the thing the requirements have to serve)

Reading-mate reads structured documents aloud in a way that's worth listening to for a long time. It treats the document's structure as linguistic information, not visual decoration. It feels like a presence, not a tool. It works locally, reliably, and without configuration ceremony.

The bar is *sci-fi-worthy reliability* — both halves of that, on purpose. A flashy demo that fails in the long tail is not the goal. A pedestrian, dependable system also is not the goal.

## Functional requirements

### F1. Accept structured input from common sources

The system must accept, at minimum:
- Markdown files and content.
- HTML pages (URL or already-rendered DOM).
- Plain text.

It should be possible to add input modalities (PDF, DOCX, browser-handoff, file watcher) without rewriting the system. The architecture must make new modalities cheap.

### F2. Preserve and use document structure

Headings, paragraphs, lists, code blocks, blockquotes, emphasis, and links must be available to downstream stages as *typed* elements. Stripping structure to a flat character stream before synthesis is forbidden.

### F3. Speak documents with prosody that reflects their structure

The system must produce audio in which a listener can perceive section boundaries, paragraph breaks, list rhythm, and emphasis without seeing the text. The exact prosodic mapping is the developer's design problem.

The bar is empirical: a listener hearing the audio without the text should be able to identify where major structural transitions occur — not from chimes, but from the speech itself.

### F4. Maintain narrator continuity within a document

The same voice from word 1 to the last word. No audible drift in pitch, timbre, or pacing across paragraphs. No discontinuities at chunk boundaries.

### F5. Handle long documents

The system must reliably read documents of at least 30 minutes of synthesized audio without quality degradation that a human listener would call out. This is a real, measurable requirement and a known weakness of current TTS — the system has to engineer around it.

### F6. Stream audio with low latency

Time from "play" to first audible word must be under 500 milliseconds for typical documents on commodity hardware. Re-synthesis of cached content should be near-instant.

### F7. Expose a programmatic control surface for playback

The system must expose, through a stable internal API, the operations: play, pause, resume, stop, jump-by-sentence (both directions), jump-by-section (both directions), continuous speed adjustment, and repeat-last-passage. Any UI layer wiring to these operations must observe a response within 100 ms. Granular indexing (current sentence, current section, time offset) must be queryable in real time.

### F8. Persist user state across sessions

If the user pauses a document and returns later, position is preserved. Voice profile is preserved. Per-document settings (pronunciation overrides, custom pacing) are preserved.

### F9. Support pronunciation overrides

The system must support per-document and per-user pronunciation overrides for arbitrary tokens. Overrides apply deterministically to every subsequent occurrence within their scope. The override store must be persistable, exportable, and inspectable.

### F10. Run locally by default

Default operation requires no network. A network mode for higher-quality voices is acceptable as an opt-in, but the offline experience must be the canonical experience and must be sufficient for daily use.

### F11. Degrade gracefully

Component failures (model crash, OOM, malformed input, network outage) must not crash the application. The system must emit a typed, observable failure event with a human-readable description; an automatic fallback path takes over wherever one exists. Silent failure is forbidden.

## Non-functional requirements

### N1. Reliability

The system must run for hours without leaks, freezes, or audio dropouts. Audio buffer underruns are bugs, not allowable transient events.

### N2. Performance budget

Real-time factor (RTF) must remain under 0.5 (synthesis at least twice as fast as playback) on the developer's target hardware. Specify the target hardware explicitly in the planning phase.

### N3. Reproducibility

Given the same input and the same voice profile, the system produces the same audio. No silent randomness in production paths. (Random seeds must be explicit and configurable.)

### N4. Observability

Per-stage latency, error rates, cache hit rates, and audio buffer health must be exposed to the developer in real time during development and to the user (in summary form) in the application.

### N5. Privacy

User documents are not transmitted to third parties without explicit per-action opt-in. Any optional online mode must make this contract clear.

### N6. License compliance

All bundled models, datasets, and code must be under licenses that permit personal use *and* be documented per-component. If any component is non-OSI-open-source (e.g., research-only weights), it is opt-in and clearly labeled.

### N7. Voice catalog as a system concept

Voices must be addressable by stable, human-meaningful identifiers — not by underlying model names or version strings. The voice catalog (preview audio, attribution, license, capability metadata) is part of the system, queryable through the API; how it's presented is the UI's concern.

### N8. Empty-input safety

The system must never produce silent output for non-empty input, and must never start synthesis for empty or malformed input. Both cases emit a typed event for the UI to handle.

## What's explicitly *not* required (yet)

These are deferred. Building any of them in v1 is scope creep:

- Voice cloning ("read in my voice").
- Multi-character dialogue (different voices for different speakers).
- Video / image content.
- Live conversational mode.
- Multi-language synthesis within a single document.
- Mobile platform support (desktop / Linux first; mobile is its own product question).
- Cloud sync of state across devices.

If the developer believes any of these is *necessary* for the v1 product, they should make the case explicitly before adding it. The default answer is no.

## Quality bars

### Latency

| Metric | Bar |
|---|---|
| Time to first audible word | ≤ 500 ms |
| Real-time factor | ≤ 0.5 |
| Pause→silent latency | ≤ 100 ms |
| Resume→audible latency | ≤ 200 ms |

### Audio

| Metric | Bar |
|---|---|
| Sample rate | ≥ 22050 Hz, preferably 24000 |
| Buffer underruns / hour at idle | 0 |
| Audible discontinuity rate at chunk boundaries | indistinguishable to a casual listener |

### Comprehension

| Metric | Bar |
|---|---|
| WER (synthesized → ASR'd → compared to source) | ≤ 5% on the standard corpus |
| Mispronunciation rate of fixed pronunciations | 0 (these are *fixed* — they should not vary) |

### Long-form

| Metric | Bar |
|---|---|
| Embedding distance of voice timbre, minute 1 vs minute 30 | within calibrated threshold |
| Listener notices voice change during 20-minute listen | < 30% of listeners |

## Acceptance test (informal)

The integrated product (system + UI) passes when Sebastian can:

1. Hand reading-mate a long markdown document or web article.
2. Listen end-to-end without wanting to switch to a commercial alternative.
3. Correct a mispronounced name and have the correction stick for the remainder of the document.
4. Pause and return — minutes, hours, or days later — and resume from the same place with the same voice and settings.
5. Trust the system enough to listen ambiently while doing other work.

The system's responsibility, specifically, is that *none of those five outcomes is blocked by a system-side limitation*: the capabilities, performance, persistence, and reliability described in the requirements above are present. Whether the experience *feels* right is a joint property of the system and the UI; this document only commits to the system half.
