# 04 — Synthesis engines

The TTS landscape changes faster than any other part of this system. This note maps the *families* of approaches, with concrete examples as of mid-2026, but the developer must verify current state at decision time. **No engine is recommended.** The goal is to make the choice deliberate.

## Families

### 1. Lightweight neural (CPU-class)

VITS-family models exported to ONNX. Two-stage: text → phonemes → audio. Sub-second on CPU. No SSML or rudimentary SSML. No style prompts.

**Example artifacts:** Piper, Kokoro-82M.

**When this is the right family:**
- Hard latency requirements.
- Low-resource hardware.
- Predictable, reliable output is more important than expressiveness.

**Tradeoffs:**
- Prosody is whatever the model learned. You shape it via punctuation and training data, not at inference.
- Voice cloning, style prompts, instruction-following: none.

### 2. Codec-LM TTS (GPU-class, mostly)

Audio is tokenized via a neural codec; an LM predicts tokens conditioned on text. Often supports zero-shot voice cloning (give it 3–10 seconds of reference audio).

**Example artifacts:** VALL-E ([2301.02111](../papers/2301.02111_valle_neural_codec_lm.pdf)), VALL-E 2, XTTS-v2 (CPML, *not* OSI open-source), Bark, IndexTTS2.

**When this is the right family:**
- Voice cloning is core to the product.
- Quality bar is high.
- GPU is acceptable.

**Tradeoffs:**
- Inference is heavier; streaming requires careful engineering.
- Some popular members of this family (XTTS-v2) are not open-source; verify before adopting.
- Long-form coherence is a known weakness ([2412.18603 long-form SLM](../papers/2412.18603_long_form_speech_generation_slm.pdf)).

### 3. Description-conditioned TTS

Takes a natural-language description of the voice as a second input. Closest fit to "use the structure as linguistic information about how to read it."

**Example artifacts:** Parler-TTS, IndexTTS2, CosyVoice 2/3, Qwen3-TTS.

**When this is the right family:**
- The product wants per-section style control.
- An LLM is already in the stack producing the descriptions.
- Open-license requirement (Parler is Apache-2.0).

**Tradeoffs:**
- Voice consistency across description changes is uneven; test empirically.
- Some of these are large; latency may be a problem.
- Description quality matters a lot — bad descriptions make bad audio.

### 4. Conversation-context-aware

Conditions on prior conversation/document audio, not just text. Targets the long-form coherence problem directly.

**Example artifacts:** Sesame CSM-1B (Apache-2.0 as of 2026), MOSS-TTSD.

**When this is the right family:**
- The dominant failure mode in your prototype is "voice drifts over time" or "prosody flattens after the first paragraph."
- You want the system to feel like *one narrator* across long content.

**Tradeoffs:**
- Most context-aware models target *conversation*, not document narration. Adapt at your own risk.
- Larger context = more compute per chunk.

### 5. Classical / non-neural

eSpeak-ng, Festival, Pico. Tiny, fast, intelligible-but-robotic.

**When this is the right family:**
- Fallback path when neural models fail.
- Accessibility minimum.
- Embedded / extreme-low-resource.

These are also useful as a *reference baseline* during evaluation: "is the neural model actually beating eSpeak-with-good-SSML?"

## Engine selection criteria

When evaluating a candidate, score it on:

1. **License.** OSI-approved or not. Verify both code and *weights* — they often differ.
2. **Latency.** Time to first byte and real-time factor.
3. **Long-form behaviour.** Synthesize a 10-paragraph passage; listen for drift, repetition, mispronunciation.
4. **Honor for prosody hints.** Does it respect SSML? Description? Pause hints? Test.
5. **Voice consistency.** Same input twice = same output? Same speaker across paragraphs?
6. **Pronunciation control.** Can you fix the model's pronunciation of a name without retraining?
7. **Streaming support.** Native or wrappable?
8. **Hardware.** CPU adequate or GPU required?
9. **Project liveness.** Last commit, open issues, response time. Dead repos rot.
10. **Inference framework.** ONNX, PyTorch, MLX. Affects what runtime you commit to.

Build a small evaluation harness once and run every candidate through it.

## What this project doesn't need to commit to

- A single engine forever. The IR-driven architecture (see `01_pipeline_architecture.md`) means engines are swappable.
- An online engine. Local-first is a goal, but a fallback to a remote engine for "premium quality" mode is not categorically wrong — it's a product decision.
- A custom-trained model. None of the project's stated goals require training. Fine-tuning, maybe. Training from scratch, no.

## Evaluation samples to use

When testing engines, use these passages (or similar) as your battery. They're chosen to surface specific failure modes.

1. **A markdown technical article with code blocks.** Tests: code-block handling, inline code, list pacing.
2. **A long fiction excerpt with dialogue.** Tests: speaker change handling, expressive prosody.
3. **A news article with quotes.** Tests: blockquote prosody, attribution flow.
4. **A scientific abstract.** Tests: numeric and abbreviation normalization, technical pronunciation.
5. **A page of pure prose, 1000+ words.** Tests: long-form drift.
6. **A page with named entities (people, places).** Tests: pronunciation consistency and correctness.
7. **A page in a non-primary language.** Tests: graceful degradation.

Synthesize each with the candidate engine, listen end-to-end, and rate against the failure modes. Rinse. Repeat.
