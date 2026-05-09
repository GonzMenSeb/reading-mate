# reading-mate

A seed, not a project. Yet.

## What this is

The vision: a system that reads structured documents — markdown, articles, papers — out loud in a way that's worth listening to for a long time. Not a wrapper around an off-the-shelf TTS. A system that treats document structure as linguistic information, that maintains a single coherent narrator across a 30-minute reading session, that runs locally, and that *feels* like a presence rather than a tool.

The bar is sci-fi-worthy *and* reliable. Both halves, on purpose.

## What this is *not*

There is no implementation here. No stack has been chosen. No model has been picked. No architecture has been committed.

That's deliberate. The job of choosing those things, justifying them, and building the system is the work that comes next. This repository is the brief that defines that work.

## What's in here

```
reading-mate/
├── README.md                  ← you are here
├── instructions/              ← what the next developer is asked to do
│   ├── README.md              ← start here
│   ├── requirements.md        ← capability requirements (what the system must do)
│   └── principles.md          ← how to make decisions during the work
└── research/                  ← the knowledge namespace
    ├── topics/                ← synthesized notes by topic
    ├── papers/                ← downloaded primary sources (PDFs)
    ├── articles/              ← snapshotted blog posts and engineering docs
    └── notes/                 ← meta indexes and lookup helpers
```

## How to use this repo

If you're the next developer:

1. Read `instructions/README.md`.
2. Read `instructions/principles.md` *before* writing code.
3. Read `research/topics/README.md` and follow the suggested reading order.
4. Read `instructions/requirements.md` and check that you understand the bar.
5. Then plan. Then build the smallest end-to-end thing. Then iterate.

If you're a reviewer or curious passer-by:

- `instructions/requirements.md` tells you what the goal is.
- `research/topics/00_problem_shape.md` tells you why it's hard.
- `research/topics/09_open_questions.md` tells you what's still unsolved.

## Why a "seed" repository at all

A few projects deserve more thought than the usual "open the editor and start typing." The mistake to avoid here is over-confident early commitment to the wrong stack, the wrong model, the wrong intermediate representation. Each of those is a one-way door.

This repo exists so that the next person to start the work has a real brief in front of them — not a vague description, not "build me a TTS reader," but a defined problem space, a curated literature, and an explicit set of capabilities and quality bars. That gives them a fighting chance to build the thing the bar describes, and to defend their choices when those choices matter.

## License

To be determined by the implementing developer, in line with the dependencies they choose. The research corpus contained in `research/papers/` and `research/articles/` consists of third-party publications and snapshots; their respective licenses apply to those files individually. See the manifests in those directories for source attribution.

## Acknowledgements

The research corpus draws on academic publications from arXiv (TTS, prosody, streaming synthesis, content extraction, evaluation) and engineering documentation from Sesame, Deepgram, Hugging Face, Trafilatura, and Nielsen Norman Group. Full attribution is in `research/papers/MANIFEST.md` and `research/articles/MANIFEST.md`.
