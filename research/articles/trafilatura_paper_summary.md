# Trafilatura — paper summary

**Source paper:** https://aclanthology.org/2021.acl-demo.15/
**Author:** Adrien Barbaresi (BBAW), ACL 2021
**License:** Apache 2.0 (Python library)
**PDF in repo:** `research/papers/2021_acl_demo_15_trafilatura.pdf`
**Why it matters here:** Solves the "given a URL, give me the article text" problem. Empirical winner across benchmarks. Relevant any time the input is a webpage and the system has to "ignore navigation, ads, footers, comments."

---

## Problem

Web pages are wrapped in boilerplate: nav, ads, share buttons, related-articles widgets, cookie banners, footers. For a reader to speak only the *article*, all of that has to be stripped — without losing structure (headings, paragraphs, lists, code).

## How it works (in brief)

Heuristic-first pipeline:

1. **Tree pruning** — strip `<nav>`, `<footer>`, `<aside>`, `<script>`, etc.
2. **XPath cascade** — try targeted selectors (article body, main content) first.
3. **Heuristic scoring** — text density, link density, element type.
4. **Fallbacks** — `jusText` and Readability if the primary cascade fails.
5. **Metadata extraction** — title, author, date, description.

Output formats include plain text, TXT, CSV, XML, TEI.

## Empirical evaluation

From the Bevendorff et al. SIGIR 2023 benchmark (14 extractors compared):

- **Trafilatura: best mean ROUGE-LSum F1 (0.883).** Best overall robustness.
- **Readability: highest median (0.970).** Most predictable on "easy" pages but fails harder on edge cases.
- **Heuristic extractors generally beat large neural models.** Surprisingly, throwing an LLM at this problem is not a free win.

This is not a paper to skim. The takeaway is that web extraction is a *solved* problem, and the solution is unsexy. Don't reinvent it. Consume it as a library.

## Implications for reading-mate

When the input modality is "a URL" or "an HTML blob":

- Use a battle-tested extractor; don't write a new one.
- Preserve structure — emit headings, lists, code blocks as distinct typed elements, not flattened text. The downstream prosody planner needs this.
- The "neural extractors lose to heuristics" finding is a useful sanity check. Pure-LLM approaches to extraction are tempting; this benchmark says don't.
