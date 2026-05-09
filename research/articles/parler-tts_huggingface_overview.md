# Parler-TTS — engineering overview

**Source:** https://github.com/huggingface/parler-tts
**Captured:** 2026-05-09
**License:** Apache-2.0 (code and weights)
**Why it matters here:** This is the canonical open-source reference implementation of "describe the voice in plain English, get audio." Functionally the closest thing to a programmatic prosody control surface that doesn't need SSML.

---

## How it works

Two inputs:

1. **Speaker description** ("how to say it") — natural-language prompt that characterizes vocal qualities.
2. **Text prompt** ("what to say") — the actual sentence/passage to read.

The model is conditioned on both and produces audio matching them.

## What goes in the description

- **Gender** — male / female.
- **Pitch** — high / moderate / low.
- **Speaking rate** — slow / moderate / fast.
- **Expressiveness** — monotone vs animated.
- **Recording quality** — "very clear audio" / "very noisy audio".
- **Reverberation** — room characteristics, proximity.

Example:
> "A female speaker delivers a slightly expressive and animated speech with a moderate speed and pitch. The recording is of very high quality, with the speaker's voice sounding clear and very close up."

## Speaker consistency

Trained on 34 named speakers. Putting a name in the description (e.g. "Jon's voice is monotone yet slightly fast in delivery, with a very close recording...") binds outputs to that speaker's timbre. Useful for: same narrator across an entire document.

## Programmatic shape

```python
from parler_tts import ParlerTTSForConditionalGeneration
from transformers import AutoTokenizer

model = ParlerTTSForConditionalGeneration.from_pretrained("parler-tts/parler-tts-mini-v1")
tokenizer = AutoTokenizer.from_pretrained("parler-tts/parler-tts-mini-v1")

input_ids = tokenizer(description, return_tensors="pt").input_ids
prompt_input_ids = tokenizer(prompt, return_tensors="pt").input_ids

generation = model.generate(input_ids=input_ids, prompt_input_ids=prompt_input_ids)
```

## Implications for reading-mate

The "description" slot is the natural target for an upstream component that maps document structure → prosody intent. Headings get one description ("speaker reads with deliberate pacing, slight emphasis on key terms"); body gets another; quoted passages get another. Speaker name pinned for consistency.

Two questions the developer must investigate:

1. **How well does Parler hold prosody across long inputs?** The HF README is silent on long-form behaviour — and the long-form-SLM and ContextSpeech papers in this corpus suggest it likely degrades. Test before committing.
2. **Is per-segment description switching jarring?** Parler doesn't carry state across calls. Switching descriptions mid-document might cause audible discontinuities. Validate empirically.
