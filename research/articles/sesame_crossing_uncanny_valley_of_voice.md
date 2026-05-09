# Crossing the Uncanny Valley of Conversational Voice

**Source:** https://www.sesame.com/research/crossing_the_uncanny_valley_of_voice
**Author:** Sesame AI
**Captured:** 2026-05-09
**Why it matters here:** Sesame defines "voice presence" as the goal — what separates a voice that lives with you from one that doesn't. Their CSM is the open-weight (Apache 2.0) reference implementation closest to what reading-mate ultimately wants to feel like.

---

## Problem framing

Today's voice assistants lack the qualities that make voice useful for sustained collaboration. Sesame's framing: *"A personal assistant who speaks only in a neutral tone has difficulty finding a permanent place in our daily lives."* Emotional flatness becomes exhausting. The bar isn't "sounds like a human" — it's *presence*: a sense that something is actually there.

## Four components of voice presence

1. **Emotional intelligence** — interpreting and responding to emotional context.
2. **Conversational dynamics** — natural timing, pauses, interruptions, emphasis.
3. **Contextual awareness** — adjusting tone and style to match situation.
4. **Consistent personality** — coherent, reliable, appropriate over time.

For reading-mate, (3) and (4) translate most directly: the voice should adjust style to "this is a heading" / "this is a code block" / "this is a quote," and it should feel like *the same reader* throughout the document.

## CSM architecture

Two autoregressive transformers based on Llama:

- **Multimodal backbone** processes interleaved text and audio tokens, models the zeroth codebook.
- **Smaller audio decoder** reconstructs remaining codebooks.

Split design improves efficiency and expressivity vs two-stage approaches, enabling lower latency while staying end-to-end trainable. The system leverages conversation history to produce more coherent speech.

## Training scale and sizes

- ~1M hours of publicly available English audio (transcribed, diarized, segmented).
- Three sizes: Tiny (1B / 100M), Small (3B / 250M), Medium (8B / 300M).
- 2048 sequence length, 5 epochs.
- Compute amortization: audio decoder trained on 1/16th of frames; zeroth codebook on all frames.

## Evaluation methodology

CMOS (Comparative MOS) with Expresso dataset, 80 evaluators.

Critical finding: **without context, listeners showed no preference between generated and human speech.** With 90 seconds of conversation context, evaluators consistently preferred humans. Sesame's read: "a noticeable gap remains between generated and human prosody" — and the gap *only shows up when context matters*.

For reading-mate this is huge: a sentence-level eval will never reveal whether the system is reading a *document* well. Evaluation must include long-context tests.

## Limitations Sesame calls out

- English-focused; multilingual underdeveloped.
- Cannot model conversation structure itself, only text+speech content.
- Doesn't use pre-trained LM weights.
- Cannot implicitly learn turn-taking, pacing, conversation dynamics.

## Quotes worth remembering

> "There are countless valid ways to speak a sentence, but only some fit a given setting."

> "Human conversations are a complex process involving turn taking, pauses, pacing, and more."
