# 05 — Streaming and latency

A reader that pauses for 10 seconds after "play" before saying anything is not a sci-fi reader. Latency is a product feature, not just an engineering concern.

## The targets

From engineering blog posts and our own product framing:

| Metric | Target |
|---|---|
| Time-to-first-audio (TTFA) | < 500 ms is conversational; < 250 ms is impressive |
| Real-time factor (RTF) | < 1.0 absolutely required (otherwise you fall behind) |
| Buffer underruns under load | 0 |
| Tail latency (p99 chunk synthesis) | < 2× median |

The SpeakStream paper ([2505.19206](../papers/2505.19206_speakstream_streaming_tts.pdf)) and Deepgram's chunking guidance ([article](../articles/deepgram_text_chunking_for_streaming_tts.md)) are the relevant precedents.

## The chunking question

You can't wait for "the document" to finish before synthesizing. You also can't synthesize letter-by-letter. The unit of work has to be chosen.

Practical chunk strategies:

- **Sentence boundary** — split on `. ! ?`. Simplest. Variable latency. Best balance of prosody and speed.
- **Fixed character window** — predictable, can break mid-sentence (bad for prosody).
- **Phrase / clause boundary** — better prosody than fixed, more complex to detect.
- **Adaptive** — small chunks early (fast first audio), larger chunks later (better intonation).

Deepgram-style guidance:

| Use case | Recommended chunk size |
|---|---|
| Real-time conversational | 50–100 chars |
| Document narration | 200–400 chars or full sentences |
| Long-form (audiobook) | Larger; intonation is paramount |

For reading-mate, default to sentence-or-paragraph chunking with fast-start optimization for the first chunk. The first sentence can be small (faster TTFA); subsequent sentences can be normal.

## Pipelining the stages

Three stages need to overlap:

```
t=0   ─── parse + plan section 1 ──┐
t=100      synthesize chunk 1 ─────┐│
t=300        play chunk 1 ─────────┤├── parse + plan section 2 (in parallel)
t=500        synthesize chunk 2 ───┘└── play chunk 2
...
```

Buffer-managed playback hides any per-chunk variance from the listener. The buffer level is the system's health signal: if it dips, the synth pipeline is falling behind.

## Caching

Aggressive caching changes the latency game. If the user re-reads, scrubs, or replays:

- **Audio cache** keyed on `(document_hash, segment_id, voice_profile)` — instant playback.
- **Plan cache** keyed on `(document_hash, planner_version)` — the prosody planning, especially if it includes an LLM call, is expensive. Cache the plan, regen audio.

A reasonable design: SQLite + filesystem for audio blobs. Don't over-engineer; cold storage is fine.

## Streaming-token edge case

If the upstream input is *itself* streaming (e.g., the user is dictating text, or an LLM is producing text token-by-token), the chunker has to buffer until a sentence boundary. This is mentioned in the Deepgram doc and is a common bug source.

## The first-chunk problem

The first chunk is the most latency-critical, and also the most prosody-critical (it sets the listener's first impression of the voice). These two pressures conflict.

Heuristic: for the first chunk, use a *very* small fragment (5–10 words, ending at a clean phrase boundary), get audio out fast, and start the next chunk's synthesis in parallel before the first finishes playing. By chunk 3, you're caught up.

## Measurements you must take

Build the timing instrumentation early. Track:

- TTFA per request.
- RTF per chunk.
- Buffer level over time.
- Cache hit rate (audio + plan).
- Per-stage time: parse, plan, synthesize, encode, deliver.

If you don't measure these from day one, the system gets slow and you can't tell why. If you do, you'll catch regressions before they ship.

## Don't optimize prematurely

A common failure mode: spending a week shaving 50ms off TTFA on a system whose dominant latency is "user clicks play, browser takes 200ms to dispatch the event." Profile first; optimize the slowest stage.

The other failure mode: optimizing for low TTFA at the cost of prosody. A 100ms TTFA with bad first-chunk prosody is worse than a 400ms TTFA with good first-chunk prosody for *long-form listening*. Don't trade the wrong way.
