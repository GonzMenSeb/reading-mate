# Principles — be deliberate with every choice

Reading these instructions, you might notice they're terse. That's the point. The repository deliberately does not contain a plan. It does not pick your stack, your model, your IR, or your runtime. *Those choices are the work.*

The next person to touch this repo (you) is expected to make those choices like a professional, defend them in writing, and be willing to be wrong publicly. This document tells you what we expect from that process.

## The rule

**No choice is allowed to be a default.**

Not the language. Not the framework. Not the TTS engine. Not the LLM. Not the IR shape. Not the storage layer. Not the test framework. Not the formatter, the linter, the build tool, the directory layout.

Every one of those choices has alternatives. You will pick one. When you pick, you must:

1. Have read enough of the alternatives to be honest about why this one.
2. Write it down in `decisions/NNN-decision-name.md` (see template below) before committing.
3. Be explicit about what you'd revisit if you were wrong.

If a choice doesn't get a decision record, it shouldn't be made yet.

## What a "decision record" looks like

We don't dictate a format, but the file should answer at minimum:

- **What did you decide?** (One sentence.)
- **What were the alternatives you actually considered?** (Not "X, Y, Z were possibilities" — what you read, tested, looked at the docs of.)
- **Why this one?** (Concrete reasons. "Faster" doesn't count; "synthesizes 12-word chunk in 180ms vs 410ms on my laptop" does.)
- **What would change your mind?** (Falsifiable. "If the latency target moves below 100ms, we'd switch to X.")
- **What does this choice constrain downstream?** (E.g. picking Python here means avoiding language-bridges later.)

ADR (Architecture Decision Record) format works fine, but the spirit matters more than the format. We want to read these in a year and understand both *what* and *what was almost picked instead*.

## The reverse rule

**You are also responsible for *not* deciding things prematurely.**

If a choice can be deferred without blocking work, defer it. Examples of choices that should not be made on day one:

- The exact final TTS engine. Pick one for prototyping; commit later.
- The deployment topology (single binary vs services).
- The persistence layer for caches.
- Whether to support a browser extension.

A choice deferred is a choice you can make better later. A choice made early is a constraint you may not realize you're paying for.

## Process expectations

### Read the corpus before writing code

The `research/` directory is the brief. Read at least:

- `research/topics/00_problem_shape.md` through `09_open_questions.md`.
- The "if you read only one paper" recommendations in `research/papers/MANIFEST.md`.
- The article snapshots in `research/articles/`.

Skim the rest. The references exist because they actually informed the requirements. If a requirement seems wrong, read its source before pushing back.

### Plan before you build

The first artifact you produce should be a plan. Not a roadmap. A *technical plan*: what will exist, in what order, talking to what, with what contracts. Get it reviewed (by Sebastian or by another agent) before writing implementation code. If the plan can't be written down clearly, it's not ready to be implemented.

### Build the eval before the system

Section 08 of the research topics describes the evaluation harness. **Build a basic version of it before you build the system.** It will be ugly and partial. That's fine. The eval is what tells you whether your changes are improvements; without it, you're flying blind.

### Smallest viable thing first

The shortest path to a working artifact is: parse markdown → synthesize with a single off-the-shelf engine → play through OS audio. Get *that* working end-to-end. Then introduce the prosody planner, the IR, the streaming pipeline. Premature integration of all the moving parts will produce a system you can't debug.

### When you change your mind, write a new decision record

Don't edit the old one. Append a new one that supersedes it. The history is the value.

## Specific things you should *not* do

- Adopt a framework before you have a working prototype.
- Build microservices on day one.
- Write a custom TTS model.
- Write a custom HTML extractor.
- Implement features in `requirements.md`'s "explicitly not required" list.
- Add comments that restate what the code does.
- Build error handling for situations that can't occur.
- Write a test before you have a function.
- Skip the decision record because "it's obvious."

## Specific things you should do

- Measure latency from day one.
- Cache aggressively, because re-synthesis is expensive.
- Make the IR explicit and round-trippable.
- Use real documents (your own reading material) as the dev corpus.
- Run the eval on every meaningful change.
- Tell the user when something fails. Never silent failure.

## On the AI-augmented-developer angle

You are likely an AI-augmented developer. That changes some things and doesn't change others.

What it changes:

- You can read more research faster. Use that. Don't skip the corpus to save time.
- You can prototype in two languages cheaply. Use that to *test* a choice before committing.
- You can generate decision records quickly. *Don't* let that make them perfunctory. The thinking is the value, not the artifact.

What it doesn't change:

- You still have to listen to the system. Audio quality is not a metric your model can score for you. You and Sebastian have to actually listen, with attention, multiple times.
- You still have to ship. A research-quality prototype that nobody uses is a research-quality prototype.
- You still have to be wrong sometimes. Don't paper over uncertainty with confident prose.

## What "done" looks like

The acceptance test in `requirements.md` is the bar. Do not declare done until those five steps work end-to-end with no asterisks. "It works for the happy path" is not done; "it crashes only rarely" is not done; "it works on my hardware" is not done.

When you think it's done, sit with it for a week. Read your own things with it. Listen in the morning, in noise, while distracted. Find what bugs you. *That's* what's not done yet.

Then, and only then, ship.
