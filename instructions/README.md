# Instructions for the next developer

Welcome. Three documents matter here:

1. **`requirements.md`** — what the system must do. Capabilities, not implementation.
2. **`principles.md`** — how to make decisions during the work. Read this *before* writing code.
3. **`../research/`** — the knowledge namespace this brief was assembled from. Read `research/topics/README.md` first.

## What's expected from you

You are not picking up a project that has been planned and partially built. You are picking up the *seed* of a project. The planning is yours. The implementation is yours. The tool choices are yours.

What's been done for you:

- The problem space has been mapped (see `research/topics/`).
- The relevant academic literature has been read and indexed (see `research/papers/`).
- Engineering precedents and UX precedents have been snapshotted (see `research/articles/`).
- The capabilities you have to deliver have been written down (see `requirements.md`).
- The way to *behave* during the work has been written down (see `principles.md`).

What's *not* been done for you, and is your job:

- Choose a language. Justify it.
- Choose a runtime / framework. Justify it.
- Choose an architecture. Defend it in a written plan.
- Choose your initial TTS engine. Test alternatives. Justify the pick.
- Choose your prosody-planner approach (rules, LLM-driven descriptions, hybrid, something else). Justify it.
- Choose your IR shape. Document it.
- Choose your evaluation harness shape. Build it before the system.
- Choose your storage strategy. Justify it.
- Choose how to ship (CLI? GUI? service? tray? all of the above?). Justify it.

Each of these choices needs a *decision record*. See `principles.md` for the format.

## What success looks like

If, six months from now, someone reads the `decisions/` directory in this repo, they should be able to:

1. Understand why every component exists.
2. Understand what alternatives were considered for each.
3. Identify which choices were made early and would be revisited.
4. See the chain of reasoning back to the requirements.

If, after using the system for a week, Sebastian:

- Trusts it to read his daily reading,
- Doesn't switch back to commercial TTS readers,
- Forgets it's a piece of software and starts referring to it like a colleague,

then it's working.

## Order of operations

A suggested sequence (not prescriptive — make your own plan):

1. Read the research namespace.
2. Write a draft technical plan. Get it reviewed.
3. Write the first three decision records: language, IR shape, initial TTS engine.
4. Build the evaluation harness skeleton. Run it on a baseline (e.g., raw plain-text-to-engine output).
5. Build the smallest viable end-to-end pipeline: input → parse → synthesize → play. No prosody planner yet.
6. Add the prosody planner. Measure improvement against the eval baseline.
7. Add streaming, voice continuity, controls, persistence.
8. Sit with it. Ship.

## A note on scope

The default impulse will be to make this bigger than it should be. Resist. The requirements list explicitly enumerates what's *not* in v1. Cut against your own ambition; small reliable systems compound, large fragile systems don't.

Sebastian asked for "sci-fi-worthy while being reliable and working well." Both halves were on purpose. A demo that wows once is not the goal; a tool that earns daily use *and* impresses on day one is.

## Communication

When you need to make a hard call:

- Write the alternatives down.
- Write your recommendation down.
- Bring it to Sebastian as a decision-to-make, not as a thing-already-done.

When something doesn't work:

- Say so explicitly.
- Don't dress it up. Don't hide the failure under a "tradeoff."
- Have a hypothesis about *why* it didn't work before discussing what to try next.

When you finish a meaningful piece:

- The decision record exists.
- The eval is updated.
- A user-visible smoke test exists.

That's the rhythm.

Good luck. The corpus is the gift; the quality of the work is yours to make.
