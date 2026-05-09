# Papers manifest

Each entry: filename, title, authors, year, source URL, why it matters here.

## Tier 1 — load-bearing

- `2402.01912_parler-tts_natural_language_guidance.pdf`
  *Natural Language Guidance of High-Fidelity Text-to-Speech with Synthetic Annotations* — Lyth & King (Stability AI / Edinburgh), 2024. https://arxiv.org/abs/2402.01912
  Foundation paper for the Parler-TTS family. Defines the "describe the voice in natural language" paradigm: gender, pitch, pace, expressiveness, room characteristics. Directly relevant to the "format-as-linguistic-input" goal — the description text is what carries prosody intent.

- `2412.06602_controllable_speech_synthesis_llm_era_survey.pdf`
  *Towards Controllable Speech Synthesis in the Era of Large Language Models* — survey, 2024. https://arxiv.org/abs/2412.06602
  Read this first. Maps the landscape: prompt-based TTS, instruction-tuned TTS, codec-based LMs, style transfer, evaluation. Cites all the others.

- `2307.00782_contextspeech_paragraph_reading.pdf`
  *ContextSpeech: Expressive and Efficient Text-to-Speech for Paragraph Reading* — 2023. https://arxiv.org/abs/2307.00782
  Paragraph-level TTS trained on audiobook narration. Direct precedent: how to make a model that reads documents, not just sentences.

- `2412.18603_long_form_speech_generation_slm.pdf`
  *Long-Form Speech Generation with Spoken Language Models* — Google research, 2024. https://arxiv.org/abs/2412.18603
  17-page deep dive on what breaks at minute-long synthesis (semantic drift, speaker drift, paralinguistic incoherence). Defines the problem class this project is in.

- `2308.14430_textrolspeech_corpus.pdf`
  *TextrolSpeech: A Text Style Control Speech Corpus With Codec Language Text-to-Speech Models* — 2023. https://arxiv.org/abs/2308.14430
  Public dataset (236h, 1,500 speakers) with text style labels. If anyone considers training/finetuning, this is the canonical corpus.

- `2301.13662_instructtts.pdf`
  *InstructTTS: Modelling Expressive TTS in Discrete Latent Space with Natural Language Style Prompt* — 2023. https://arxiv.org/abs/2301.13662
  Earlier work that pioneered free-form natural-language style prompts. Worth reading for what the field tried before Parler-TTS converged on annotations.

- `2211.02336_audiobook_tts_prosody_context.pdf`
  *Improving Speech Prosody of Audiobook Text-to-Speech Synthesis with Acoustic and Textual Contexts* — 2022. https://arxiv.org/abs/2211.02336
  Empirical study of how much prior context (textual and acoustic) actually improves prosody. Use to reason about what context window the system should maintain when reading long documents.

- `2508.14713_long_context_speech_memory.pdf`
  *Long-Context Speech Synthesis with Context-Aware Memory* — 2025. https://arxiv.org/abs/2508.14713
  Specifically targets paragraph-level synthesis with a memory mechanism for cross-utterance dependencies. State-of-the-art for the document-reading shape.

- `2301.02111_valle_neural_codec_lm.pdf`
  *Neural Codec Language Models are Zero-Shot Text to Speech Synthesizers (VALL-E)* — Microsoft, 2023. https://arxiv.org/abs/2301.02111
  The codec-LM paradigm that underlies Parler, CosyVoice, Sesame CSM. Read for architectural literacy: tokens-of-audio + LM = TTS.

- `2505.19206_speakstream_streaming_tts.pdf`
  *SpeakStream: Streaming Text-to-Speech with Interleaved Data* — 2025. https://arxiv.org/abs/2505.19206
  How to make codec-LM TTS run in real time with token interleaving. Directly relevant for "respond now, finish synthesizing as the user listens."

- `2310.06930_prosody_analysis_audiobooks.pdf`
  *Prosody Analysis of Audiobooks* — 2023. https://arxiv.org/abs/2310.06930
  Empirical analysis of how professional narrators actually use prosody — pauses at section breaks, pitch reset for new paragraphs, etc. Use as ground truth for what "good reading prosody" looks like.

- `2307.16012_msstyletts_hierarchical_style.pdf`
  *MSStyleTTS: Multi-Scale Style Modeling with Hierarchical Context Information* — 2023. https://arxiv.org/abs/2307.16012
  Hierarchical style modeling: utterance-level, paragraph-level, document-level. The structural shape this project's system likely needs.

## Tier 2 — supporting

- `2406.06406_emotion_natural_language_tts.pdf`
  *Controlling Emotion in Text-to-Speech with Natural Language Prompts* — 2024. https://arxiv.org/abs/2406.06406
  Tighter focus on emotion-via-text-prompt. Useful when designing how the upstream component decides what emotional cue to pass along.

- `2406.05370_valle_2.pdf`
  *VALL-E 2: Neural Codec Language Models are Human Parity Zero-Shot Text to Speech Synthesizers* — Microsoft, 2024. https://arxiv.org/abs/2406.05370
  Successor paper. Important context for the codec-LM trajectory.

- `2509.20485_objective_prosody_intelligibility_evaluation.pdf`
  *Objective Evaluation of Prosody and Intelligibility in Speech Synthesis* — 2025. https://arxiv.org/abs/2509.20485
  Evaluation methodology — automated metrics that correlate with human judgment. Use to set up CI quality gates instead of subjective spot-checks.

- `2509.17516_audiobook_cc_controllable.pdf`
  *Audiobook-CC: Controllable Long-Context Speech Generation for Audiobook* — 2025. https://arxiv.org/abs/2509.17516
  Pipeline for chapter-segmenting fiction, character analysis, narration vs dialogue routing. Architectural inspiration for the document-pipeline shape.

- `1808.01410_predicting_expressive_speaking_style.pdf`
  *Predicting Expressive Speaking Style from Text in End-to-End Speech Synthesis* — Google, 2018. https://arxiv.org/abs/1808.01410
  Foundational early paper on inferring style purely from input text. Predates LLM-driven approaches but still cited for the basic problem framing.

- `2407.12707_ttsds_evaluation.pdf`
  *TTSDS: Text-to-Speech Distribution Score* — 2024. https://arxiv.org/abs/2407.12707
  Distribution-distance evaluation that catches things MOS misses (mode collapse, prosody flattening). Use alongside CMOS for evaluation.

- `2508.17494_ssml_prosody_french.pdf`
  *Improving French Synthetic Speech Quality via SSML Prosody Control* — 2025. https://arxiv.org/abs/2508.17494
  Concrete demonstration that SSML still meaningfully improves modern TTS (MOS 3.20 → 3.87). Relevant if SSML is chosen as the intermediate representation.

- `2021_acl_demo_15_trafilatura.pdf`
  *Trafilatura: A Web Scraping Library and Command-Line Tool for Text Discovery and Extraction* — Barbaresi, ACL 2021. https://aclanthology.org/2021.acl-demo.15/
  Canonical reference for high-quality main-content extraction. Empirical winner across benchmarks. Relevant for the "ignore everything that's not important" web-page case.

## Reading order

If the next developer reads only one: **2412.06602** (the LLM-era TTS survey).
If they read three: add **2307.00782** (ContextSpeech) and **2402.01912** (Parler-TTS).
If they read five: add **2412.18603** (long-form SLM) and **2310.06930** (prosody of audiobooks).
