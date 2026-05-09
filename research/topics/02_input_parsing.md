# 02 — Input parsing

The input modalities reading-mate plausibly has to handle:

| Modality | Where it comes from | Hardest aspect |
|---|---|---|
| Markdown | File, paste | Code blocks, math, link rendering |
| HTML | URL, browser selection | Boilerplate stripping, dynamic content |
| Plain text | Paste, file | Inferring structure that isn't marked |
| PDF | File, URL | Layout vs reading order, hyphenation |
| Rich-text / RTF / DOCX | File | Format zoo; OOXML particularly |
| Browser article | Reader-mode handoff | Already-cleaned but variable shape |

**Don't try to handle all of these in v1.** Pick the modality with the highest signal-to-effort ratio and design the system so adding modalities is incremental.

## Markdown

The cleanest input. AST-based parsers exist in every language. Choices the developer must make:

- **CommonMark vs GFM vs MDX** — does the system have to handle GitHub-flavoured tables, task lists, code fences with languages? Almost certainly yes.
- **Frontmatter** — YAML front-matter has metadata that should bypass synthesis (don't read the YAML; use it).
- **Math** — LaTeX or KaTeX-syntax math should arguably be *spoken* in human form ("integral from zero to infinity of x squared dx"). Non-trivial.
- **Code blocks** — the canonical example of "read this differently." Speak the language hint, don't read the code character-by-character. Probably summarize.

## HTML

Two sub-problems:

**Boilerplate stripping** — solved problem. Use Trafilatura or a peer; don't reinvent. ([trafilatura summary](../articles/trafilatura_paper_summary.md))

**Dynamic content** — JavaScript-rendered pages. A library that parses static HTML will see a skeleton on SPA-heavy sites. The developer must decide whether to:

1. Punt: only support static HTML. Ask the user to use the browser's reader-mode and paste. Cheap.
2. Use a headless browser (Playwright, Puppeteer). Heavy but complete.
3. Use the browser as the upstream — a browser extension that hands the system *already-rendered DOM*. This is probably the right answer for "I want to read this article" use cases.

The choice is product-shape, not just engineering.

## Plain text

Underrated case. Most text the user wants read is already plain. Challenge: there *is* structure (paragraphs separated by blank lines, lists, sometimes ASCII headings), but it's underspecified.

Heuristics that work:

- Paragraphs = double-newline-separated blocks.
- Lists = consecutive lines starting with `- `, `* `, `1. `, etc.
- Sentences = naive split, then refined.
- All-caps short lines surrounded by blank lines = probable heading.

Don't try to do *too much* heuristic recovery. If the input is plain text, treat it as prose with paragraph breaks; that's the contract.

## PDF

Hard. PDFs encode visual layout, not reading order. A two-column academic paper, a magazine, a PDF with footnotes — each requires reasoning about which text is in what order, what's a sidebar, what's a caption.

Tools:

- `pdfminer.six` (Python) — gives you positioned text. Cheap.
- `unstructured` (Python) — heuristic structuring.
- `pdftotext` from poppler — fastest but loses structure.
- Visual / OCR-based tools (Marker, MinerU, GROBID for academic) — slow but produce structured output.

If reading-mate is "for the user's day-to-day reading," PDF support may be lower priority than markdown + HTML. But academic users will want it. **Decide the use case before committing to the parsing depth.**

## Rich-text formats (DOCX, RTF, ODT)

Use `pandoc` as a universal front-end. Pandoc reads ~30 formats and emits to a structured AST. Nobody should write a DOCX parser from scratch in 2026.

## Browser-handoff modality

The cleanest UX is "I'm reading this article in my browser, hand it off to reading-mate." Two variants:

1. **Browser extension** — extension reads the rendered DOM, sends to local reading-mate API. Best content quality.
2. **System reading-mode share** — OS-level share intent (Linux: not a thing natively; macOS/iOS/Android: yes). Ask the user to install reader-mode, then paste cleaned content.

If the developer wants this product to *feel* sci-fi, an extension that lets you select text in any tab and have it read in a chosen voice is probably the killer demo.

## Output of parsing: the structured stream

Whatever the input format, the output of this stage is a typed tree. Suggested element types:

```
Document(metadata: {title, author, lang, ...})
├── Section(level: 1..6, title, anchor)
│   ├── Paragraph(inline_runs)
│   ├── List(ordered, items)
│   ├── CodeBlock(lang?, content, summary?)
│   ├── Quote(attribution?, content)
│   ├── Figure(caption, alt, src)
│   ├── Table(caption?, rows)
│   └── Math(latex?, mathml?)
└── Footnote(id, content)
```

`inline_runs` carry emphasis, code spans, links, citations. They matter at the planner stage.

## What parsing should *not* do

- Not decide pacing.
- Not decide pauses.
- Not normalize abbreviations or numbers (that's the planner's job; doing it here destroys information).
- Not strip "non-essential" elements heuristically — let the planner decide what to skip.

Parsing is information-preserving. The planner is information-reducing. Keep that boundary clean.
