# 06 — Voice continuity

Listeners notice when the narrator changes. They notice when the same narrator's voice drifts over time. They notice when the prosody flattens after the first paragraph because the model lost context.

This topic is about all three failure modes.

## Within-document continuity

The same narrator, the same voice, from word 1 to word 50,000.

**Failure modes observed in the field:**

- **Speaker drift in codec-LM TTS.** Long-context generation loses speaker identity over time. Documented in [Long-Form SLM 2412.18603](../papers/2412.18603_long_form_speech_generation_slm.pdf).
- **Prosody collapse.** First paragraph sounds animated; tenth sounds flat. Common in models without document-level conditioning.
- **Cross-chunk discontinuity.** Audible "seams" where one synthesis call ends and the next begins. Especially when chunks are short.

**Mitigations the field uses:**

- **Speaker conditioning every chunk.** Pass the reference audio (or speaker name/embedding) on every synthesis call, not just the first.
- **Cross-chunk context.** Pass the last N seconds of audio (or its tokens, in codec-LM models) as conditioning for the next chunk. Audiobook-CC and ContextSpeech do versions of this.
- **Crossfade at chunk boundaries.** A short overlap-fade hides spectral discontinuities. Cheap, effective.
- **Single-engine, single-speaker invariant.** Don't switch engines or speakers mid-document.

## The voice profile abstraction

Define the voice once, reference it everywhere:

```
VoiceProfile(
    id="default-narrator",
    engine="parler-tts-mini-v1",
    description="A male narrator with a calm, deliberate pace and slight warmth. Clear high-quality recording, very close up.",
    speaker_name="Jon",  # Parler's named-speaker affordance
    fallback_engine="kokoro-en-us",
    fallback_voice="am_michael",
)
```

Every synthesis call takes a voice profile. The IR carries the profile ID; the adapter uses it. Switching narrators is a profile swap; multi-voice features (different voices for quotes, for example) are profile compositions.

This abstraction is small and worth getting right early.

## Cross-document continuity

The user has a "default narrator." They've grown used to it. Don't change it across documents without their knowledge.

**Implication:** voice profile is a user setting, not a per-call parameter. Persist it. Let the user override per document if they want.

## Cross-section register

Within a document, prosodic *register* should adjust with content (a fiction excerpt should sound different from a code tutorial), but it should not whiplash. Adjacent sections shouldn't sound jarringly different in the same document.

**Heuristics:**

- Smooth description transitions. If the planner emits per-section descriptions, *interpolate* nearby descriptions or pick a single description per coarse-grained section, not per paragraph.
- Detect register changes explicitly. A section labeled `> Quote` legitimately *should* sound different. Other transitions probably shouldn't.
- A/B test discontinuity. Generate the same document with vs without per-section register changes; have a listener vote.

## Pronunciation continuity

Named entities should be pronounced the same way every time they appear. This is harder than it sounds:

- Most TTS engines have no inter-call memory. The model that pronounces "Eurusaci" one way at the start of paragraph 1 might pronounce it differently in paragraph 5.
- The fix: a **pronunciation dictionary** scoped to the document (or user). Detect proper nouns at parse time, pin a pronunciation, force-substitute on every occurrence.

Voice Dream Reader's pronunciation dictionary is a good UX precedent. ([NN/g voice-first article](../articles/nngroup_voice_first.md))

## When the user changes the voice

Mid-document voice switch is a real product case ("hmm, this voice is annoying me, swap"). Two failure modes to design around:

- **In-flight playback.** Cut over at the next chunk boundary; don't restart the whole document. Provide a brief "switching voice…" cue if the cutover is audible.
- **Cache invalidation.** The audio cache is keyed on voice profile. Don't just dump it — keep both cached, in case the user switches back.

## The end-of-document affordance

Long sessions need natural endings. Don't just stop. Slow the final sentence slightly, let the last syllable breathe. The empirical [audiobook prosody paper](../papers/2310.06930_prosody_analysis_audiobooks.pdf) documents this in human narrators; you can encode it as a planner rule.

Same goes for chapter ends, section ends. Audible structure helps the listener know "we just finished a thing."
