# Knowledge namespace

These are the topic-organized notes that compile what we found into a working understanding of the problem space. Read them in order if you're new; jump in topically if you're not.

The notes deliberately avoid prescribing a stack. They explain what the *problem* looks like, what *the field has tried*, and what *seems to work*. The choices about language, framework, model, and topology are explicitly yours to make.

## Reading order

1. `00_problem_shape.md` — what you're actually building, beyond "TTS for documents."
2. `01_pipeline_architecture.md` — the conceptual stages every system in this space ends up having.
3. `02_input_parsing.md` — turning markdown / HTML / PDF / plain text into a structured stream.
4. `03_prosody_planning.md` — *the* hard part. Structure → prosody intent.
5. `04_synthesis_engines.md` — landscape of TTS engines without picking one.
6. `05_streaming_and_latency.md` — making it feel real-time.
7. `06_voice_continuity.md` — same narrator across the whole document.
8. `08_evaluation.md` — how to know it's actually good.
9. `09_open_questions.md` — what nobody has solved well yet.

> The UI/UX layer is owned outside this repository. These notes intentionally focus on the system: pipeline, prosody, synthesis, evaluation. If you need the UI brief, ask Sebastian; it's maintained alongside the design work.

## How these notes were assembled

A research pass over arxiv (TTS, prosody, streaming, long-form synthesis), engineering blogs (Sesame, Deepgram, NN/g), the Trafilatura paper, and primary sources for the open-source TTS families (Parler, Kokoro, Piper, Sesame CSM, CosyVoice, Bark). PDFs are in `../papers/`; article snapshots are in `../articles/`.

Anything in here that looks like a fact has a citation back to the corpus. Anything that looks like an opinion is flagged as such.
