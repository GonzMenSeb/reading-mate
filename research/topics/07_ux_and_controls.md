# 07 — UX and controls

The system is voice-output-first. Every UX decision has to be made through the lens of "voice is ephemeral, the screen helps."

## What the user actually does

A small set of operations covers most use:

- **Start.** Hand the system content (file, URL, pasted text, browser selection).
- **Listen.** Audio plays.
- **Adjust.** Faster, slower, repeat that, jump to next section, switch voice.
- **Reference.** "What was that link?" "Show me where I am." "What did the previous paragraph say?"
- **Stop / resume.** Across days, possibly across devices.

These are the scenarios to design end-to-end before adding anything else.

## The "where am I" problem

Voice is serial; the user can't skim. The system has to make position legible. Options:

- **Visual anchor.** A live transcript with the current word/sentence highlighted. Ubiquitous in commercial readers (Speechify, Voice Dream Reader). Low cost, high payoff.
- **Audio cues.** Subtle chime at section boundaries. Risk: annoying. Tune carefully or make optional.
- **Voice cues.** "Section 3 of 7." Useful in long documents; only at navigation moments, not constantly.

The visual transcript with highlight is probably mandatory for v1. Build it.

## Speed control

Users want to control speed and the engineering choices matter:

- **Naive time-stretch (PSOLA, WSOLA).** Stretches audio without pitch shift. Cheap. Good up to ~1.5×. Beyond that, artifacts.
- **Re-synthesize at higher rate.** Ask the synthesizer to speak faster. Better quality at extreme speeds; can't be applied to already-cached audio without re-gen.
- **Hybrid.** Time-stretch for live adjustments; re-synthesize when the user picks a stable preference.

Listeners typically settle at 1.2–1.6× for non-fiction. Don't optimize for 3× — those users want a different product.

## Pause behaviour

Pause should be instant. Resume should be instant *and* should resume mid-sentence with reasonable prosody (a short rewind to a phrase boundary is acceptable; restarting the sentence sometimes is too).

Engineering note: most TTS pipelines synthesize chunks of audio, not arbitrary spans. To resume cleanly, you need either a fine-grained chunk index or a willingness to seek within a chunk. Plan the cache and chunking accordingly.

## Skip and jump

- **Forward 30s / back 30s.** Standard. Easy.
- **Next sentence / previous sentence.** Requires sentence-level chunking and an index of sentence-start times. Worth building.
- **Next section / previous section.** Same, at section level. Requires the planner to emit section boundaries.
- **Search.** "Take me to where it talks about Sesame." Voice or text input; runs against the document text; jumps to the matching audio offset. This is the sci-fi-feeling feature.

## Voice selection

The user's library of available voices should be:

- Discoverable but not overwhelming. 5–10 named voices, not a model zoo.
- Previewable. A 10-second sample of each on a known sentence.
- Pinnable per-document or globally.

Don't expose engine names or technical details by default. The user picks "Aria" or "Marcus," not "parler-tts-mini-v1 with description X."

## Pronunciation overrides

Per-document or per-user pronunciation dictionary. UX:

- After the system speaks a name, the user can say (or click) "fix that pronunciation." A dialog appears with the offending word and an input for the IPA / phonetic spelling / "speak it like THIS" voice sample.
- The override is persisted and applied to all subsequent occurrences in the document and (optionally) globally.

This single feature is the difference between "demo" and "I'd actually use this."

## Selection and "read this"

Hand-off from other apps:

- **Browser extension** that adds a "Read selection" command to the right-click menu.
- **System-level service** (where supported) that accepts text and reads it.
- **CLI** that accepts stdin and reads.

Each of these expands the surface area the user can reach. Prioritize whichever is highest-leverage for Sebastian's actual reading — likely browser, then markdown files.

## Errors and graceful degradation

What happens when:

- The synthesis model crashes mid-document. (Switch to fallback engine; warn the user.)
- The model mispronounces a word the user can't fix. (Pronunciation override flow.)
- The input is malformed or empty. (Don't read silence; tell the user via voice or visual.)
- A network call (if any) times out. (Local fallback; tell the user "I couldn't reach the better voice; using offline narrator.")
- The document is in a language the system doesn't support. (Don't read it badly. Detect and decline.)

These are all small features individually; together they're the difference between a fragile demo and something the user actually trusts.

## What to *not* build first

Resist:

- Voice cloning. ("Read in *my* voice.") Cool, niche, opens legal/UX questions.
- Multi-character dialogue. ("Different voices for different speakers.") Fascinating; defer.
- Video / image content reading. (OCR + synthesis chain; orthogonal product.)
- Live conversation mode. ("Chat with the document.") Different product entirely.

The core is "single-narrator-reads-the-document-well." Earn the right to expand by getting that right first.
