# Text Chunking for Streaming TTS — Deepgram docs

**Source:** https://developers.deepgram.com/docs/tts-text-chunking
**Captured:** 2026-05-09
**Why it matters here:** Engineering-grade guidance on the latency/prosody tradeoff that any reading system has to face. Specific numbers, not vibes.

---

## Core principle

Text chunking exists to start playing audio before generation completes. Split at sentence boundaries while preserving natural speech patterns.

## Concrete chunk-size guidance

| Use case | Recommended chunk |
|---|---|
| Voice assistants (responsive feel) | 50–100 chars |
| Call-center bots (natural sound) | Complete sentences |
| Long-form content (intonation preservation) | 200–400 chars |

For reading-mate, "long-form content" is the dominant case. Default toward sentence-or-larger chunks; only drop to 50–100 char fragments if a specific sub-feature (preview line, hover-read) demands it.

## Sentence boundary detection

Primary method: regex split on `. ! ?` while preserving punctuation. Strip empty chunks. Maintain grammatical structure. The doc calls this approach "simplest and most effective" — sentence-level chunking offers the best balance of latency and prosody.

## Streaming text from upstream LLM

When the producer is itself streaming (an LLM emits tokens):

- Collect tokens into complete sentences before sending to TTS.
- Queue incoming paragraphs as they arrive.
- Process sentences sequentially, not in parallel (concurrent generation breaks prosody continuity).

## Latency vs prosody tradeoff

- Smaller chunks → faster initial playback, riskier intonation.
- Larger chunks → better intonation, slower start.

Doc mentions clause-based, NLP-based, and adaptive chunking as alternatives but doesn't detail thresholds.

## Implications for reading-mate

The system will likely have **two upstream producers**: a planner/preprocessor (LLM) emitting prose-with-cues, and a real-time path for "read selection now." Both feed the same TTS chunker. Decouple chunking from generation so the strategy can be tuned without retraining anything.
