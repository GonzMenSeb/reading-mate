# Source index

Quick lookup: which papers/articles cover which question. Use to find primary sources without re-reading every topic note.

## On long-form / document-level synthesis

- [2412.18603 Long-Form Speech Generation with Spoken Language Models](../papers/2412.18603_long_form_speech_generation_slm.pdf) — Google research. Defines the long-form problem.
- [2307.00782 ContextSpeech: Expressive and Efficient TTS for Paragraph Reading](../papers/2307.00782_contextspeech_paragraph_reading.pdf) — paragraph-level TTS.
- [2508.14713 Long-Context Speech Synthesis with Context-Aware Memory](../papers/2508.14713_long_context_speech_memory.pdf) — paragraph-level memory mechanism.
- [2509.17516 Audiobook-CC: Controllable Long-Context Speech Generation](../papers/2509.17516_audiobook_cc_controllable.pdf) — chapter-segmenting + character routing pipeline.
- [2211.02336 Improving Speech Prosody of Audiobook TTS](../papers/2211.02336_audiobook_tts_prosody_context.pdf) — empirical: how much context helps.
- [2310.06930 Prosody Analysis of Audiobooks](../papers/2310.06930_prosody_analysis_audiobooks.pdf) — empirical study of professional narration.
- [2307.16012 MSStyleTTS: Multi-Scale Style Modeling](../papers/2307.16012_msstyletts_hierarchical_style.pdf) — hierarchical style at multiple levels.

## On instruction / description / prompt-based TTS

- [2402.01912 Parler-TTS / Lyth & King](../papers/2402.01912_parler-tts_natural_language_guidance.pdf) — natural-language guidance with synthetic annotations.
- [2301.13662 InstructTTS](../papers/2301.13662_instructtts.pdf) — discrete-latent + style prompt.
- [2308.14430 TextrolSpeech](../papers/2308.14430_textrolspeech_corpus.pdf) — corpus + codec model.
- [2406.06406 Controlling Emotion in TTS with Natural Language](../papers/2406.06406_emotion_natural_language_tts.pdf) — emotion-via-prompt.
- [2412.06602 Towards Controllable Speech Synthesis in the LLM Era](../papers/2412.06602_controllable_speech_synthesis_llm_era_survey.pdf) — survey. Read first.

## On codec-LM TTS architecture

- [2301.02111 VALL-E](../papers/2301.02111_valle_neural_codec_lm.pdf) — original codec-LM TTS paper.
- [2406.05370 VALL-E 2](../papers/2406.05370_valle_2.pdf) — successor with quality at human parity (zero-shot).

## On streaming / real-time

- [2505.19206 SpeakStream](../papers/2505.19206_speakstream_streaming_tts.pdf) — interleaved-token streaming.
- [Deepgram chunking guide](../articles/deepgram_text_chunking_for_streaming_tts.md) — engineering chunk-size guidance.

## On structure markup (SSML)

- [2508.17494 Improving French Synthetic Speech via SSML](../papers/2508.17494_ssml_prosody_french.pdf) — SSML still helps modern neural TTS (MOS 3.20 → 3.87).

## On evaluation

- [2407.12707 TTSDS](../papers/2407.12707_ttsds_evaluation.pdf) — distribution-distance evaluation.
- [2509.20485 Objective Evaluation of Prosody and Intelligibility](../papers/2509.20485_objective_prosody_intelligibility_evaluation.pdf) — better-correlated objective metrics.

## On extraction / preprocessing

- [Trafilatura — ACL 2021](../papers/2021_acl_demo_15_trafilatura.pdf) — main-content extraction. SOTA across benchmarks.

## On product framing / UX

- [Sesame: Crossing the Uncanny Valley of Voice](../articles/sesame_crossing_uncanny_valley_of_voice.md) — the "voice presence" framing.
- [NN/g: Voice First](../articles/nngroup_voice_first.md) — voice-output UX rules.

## On engine choices

- [Parler-TTS engineering overview](../articles/parler-tts_huggingface_overview.md) — how to use Parler programmatically.

## Earlier expressivity work

- [1808.01410 Predicting Expressive Speaking Style from Text](../papers/1808.01410_predicting_expressive_speaking_style.pdf) — pre-LLM era foundational work.
