# 00 — Problem shape

The shortest summary of what reading-mate is: **a system that turns structured documents into speech that a human would actually want to listen to for a long time.**

That sentence hides four claims. Each one rules out a class of "obvious" solutions.

## "Structured documents"

The input is not prose. It's markdown, HTML, PDFs, code-with-prose, technical articles, slides, papers. Structure is everywhere: headings, lists, tables, code blocks, blockquotes, footnotes, links, emphasis, math, captions. **Most of that structure is linguistic information** — it tells you something about how the content should be heard, not just rendered.

A heading isn't text-with-bigger-font; it's a topic boundary. A blockquote isn't text-with-a-margin; it's reported speech. A code block isn't text-in-a-font; it's something the reader's voice should slow down for and de-emphasize.

This rules out: "feed plain text into a TTS." Plain-text-only systems aren't bad; they're answering a different question.

## "Speech"

Output is audio, not annotated text. The system commits to a particular reading at synthesis time. There's no "press play to hear it" abstraction layer that hides decisions; every decision about pace, pitch, pause, emphasis is *materialized as samples*.

This is what makes the problem hard. Annotating "pause here" is cheap. Producing a pause that sounds natural in *this voice*, *in this section*, *after this sentence* is not.

This rules out: SSML-as-the-product. SSML is an intermediate representation, and a useful one. But "we generate SSML" is not a complete answer — someone still has to choose a TTS that honours it well, and most modern neural TTS engines don't honour SSML at all.

## "A human would actually want to listen to"

Subjective bar, but real. There's a difference between "intelligible" and "tolerable for an hour." Speechify / Voice Dream Reader / NaturalReader / iOS VoiceOver are all intelligible. People still pay ElevenLabs for the difference.

The Sesame CMOS finding is the load-bearing data point: in zero-context A/B tests, listeners can't tell modern TTS from human. *In context*, they can. Long-form listening is the regime where the gap shows up. ([sesame article in repo](../articles/sesame_crossing_uncanny_valley_of_voice.md))

This rules out: "good enough" engineering. Reading-mate isn't a new wrapper around Piper. It's a system whose value is concentrated in the parts that the existing wrappers don't do well.

## "For a long time"

Sessions are minutes to hours. Documents are multi-paragraph, multi-section, sometimes multi-chapter. The system has to maintain:

- **Speaker identity** — same narrator throughout.
- **Prosodic register** — same general "feel" (calm, deliberate, upbeat) within a section.
- **Semantic coherence** — pronoun resolution, named-entity pronunciation consistency, tonal fit between paragraphs.
- **The user's place** — pause/resume/jump/scrub without losing context.

Long-form is where most current models break. ([Long-Form SLM paper, 2412.18603](../papers/2412.18603_long_form_speech_generation_slm.pdf) is the canonical statement of this.)

This rules out: per-sentence inference. The unit of work cannot be a single sentence in isolation; the system needs context windows that span at least a paragraph and probably a section.

## What this leaves

A pipeline. Roughly:

```
input → parser → structured stream → prosody planner → TTS synthesizer → audio
                                            ^                  ^
                                            |                  |
                                       linguistic         voice/style
                                       intent (pauses,    profile
                                       emphasis, pace)
```

The interesting parts are not the endpoints. The parser is a known problem (Trafilatura, pandoc, tree-sitter). Modern TTS engines are commodity. **The prosody planner is the part of the system that doesn't exist yet** — at least not as an open-source artifact you can pick up off the shelf.

The next developer's primary engineering question is: *what does the prosody planner look like, and what shape of intermediate representation does it emit?* Most other choices follow from that.

## What "sci-fi-worthy" actually constrains

Sebastian asked for sci-fi-worthy. Translating that into engineering constraints:

1. **Latency feels conversational.** First word starts within ~500 ms of "play." The Deepgram chunking guidance (50–400 char chunks) and SpeakStream-style interleaving ([2505.19206](../papers/2505.19206_speakstream_streaming_tts.pdf)) are the relevant precedents.
2. **The voice is a presence, not a tool.** Picks up on context. Adjusts. Doesn't sound like the same neutral assistant for every document. This is the Sesame brief.
3. **It does the right thing without configuration.** Hand it a markdown file, hand it a URL, hand it a PDF — it just reads. No "select your engine" dialog. No SSML hand-tuning.
4. **It's reliable.** Sci-fi systems work. They don't crash, they don't mispronounce names, they don't lose your place. Reliability is part of the magic.

If any of those four feels optional, the bar has slipped. Sebastian asked for both *sci-fi* and *reliable* — together, on purpose.
