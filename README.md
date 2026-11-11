# visual-stack-review

A Claude Code skill that acts as a senior document-engineering consultant. It detects the technology stack being used (or planned) to produce a document, classifies the document type, and gives honest recommendations on whether to stay-and-fix or migrate to a better-suited tool.

---

## Overview

`visual-stack-review` analyses any document-production pipeline — PDF rendering, static-site tooling, programmatic PDF generation — and answers the question: *"Is this the right tool for what I'm building?"*

It works in three modes:

- **Mode A — Render + source review:** You have both a PDF output and the source code that produced it. The skill detects the stack from both, cross-corroborates the signals, and advises.
- **Mode B — Source-only advisory:** You have source (HTML, Markdown, LaTeX, Typst, etc.) but have not rendered yet. Useful for catching tool-mismatch problems before committing to a pipeline.
- **Mode C — Prospective intake:** No artifact yet. You describe what you want to produce and the skill runs a short intake interview, then recommends a stack.

The skill always produces a **stay-and-fix assessment** first — an honest accounting of what your current tool costs to maintain and what can be fixed within it. Full alternative comparisons (top 2–4 tools) are produced on request, or automatically when the current stack has structural limitations that cannot be resolved.

---

## When to use it / Triggers

Invoke this skill when:

- You ask "is this the right tool?" about any document-production stack
- You ask "what should I use to build X?" for a new document pipeline
- You have a specific rendering bug (broken table pagination, broken footnotes, header/footer issues) and want to know whether it is fixable or structural
- You want a technology stack recommendation for any document-production pipeline
- The `visual-review` skill surfaces a problem that requires a stack-level decision

Example prompts that trigger it:
- "Why is my table pagination broken in Paged.js?"
- "Should I use WeasyPrint or Prince for this report?"
- "I'm going to build a monthly PDF invoice system — what stack should I use?"
- "Is wkhtmltopdf still a viable choice?"

---

## What the skill produces

### Detection (Phase 1)

Detects the current renderer from three signal classes:

- **PDF metadata** — `Producer` / `Creator` fields from `pdfinfo` (most reliable for identifying the actual renderer used)
- **Project files** — `package.json`, `requirements.txt`, `pyproject.toml`, `Makefile`, build scripts
- **HTML/source signals** — CSS `@page` rules, `.pagedjs_page` class, vendor-prefixed properties, source file extensions (`.tex`, `.typ`)

Supports detection of: Paged.js, WeasyPrint, PrinceXML, Puppeteer, Playwright (headless Chrome), wkhtmltopdf, LaTeX/XeLaTeX/LuaLaTeX, Typst, InDesign, Affinity Publisher, Word/LibreOffice, React-PDF, ReportLab/fpdf2, Pandoc pipelines, and Docusaurus/Hugo/Sphinx.

Each detection result includes a confidence rating (`confident` / `likely` / `possible` / `unknown`) and lists the specific signals that support it. Known misleading cases are called out — monorepos, CDN-loaded Paged.js, the Chromium Producer field shared by multiple renderers.

### Document classification (Phase 2)

Classifies the document into one of eight types: designed report, data report, long-form prose, slide deck, form/fillable, invoice/receipt, marketing collateral, or technical docs. This determines which tool capabilities matter most for the recommendation.

### Stay-and-fix assessment (Phase 3, always produced)

For the current stack: genuine strengths for this document type, fixable pain points (CSS bugs, configuration gaps), structural limitations (things that cannot be resolved within the tool), and a plain-language verdict on whether staying is the right call.

Unmaintained or CVE-carrying stacks (wkhtmltopdf) are excluded from this option — only a migration brief is produced.

### Alternative comparison (Phase 4, on request or when structural limits found)

Top 2–4 alternatives with, for each: quality ceiling for the document type, where it wins and loses versus the current stack, and a five-component migration cost breakdown:

1. Source rewrite
2. Layout-engine retuning (break heuristics, footnotes, running content, SVG/font fidelity)
3. Build pipeline changes
4. Ecosystem regression risk
5. Author retraining

### Recommendation (Phase 5)

A single clear recommendation with the decisive reason, plus up to three concrete flip conditions — situations where the recommendation changes to a named alternative.

### Tool capability matrix

A reference matrix covering nine tools across 18 capability dimensions (CSS layout fidelity, table handling, running headers/footers, widow/orphan control, footnotes, SVG, JavaScript execution, data binding, render speed, PDF/X-4 conformance, math typesetting, text typesetting quality, version-control friendliness, license cost, and more).

The matrix includes a quarterly review procedure (invoked with `/visual-stack-review review-matrix`) that checks each tool's release activity and produces a structured diff of changed ratings.

---

## What's inside

```
skills/visual-stack-review/
├── SKILL.md                          # Full skill definition and tool comparison matrix
├── metadata.json                     # Name, version, author, keywords
└── references/tools/                 # Deep-dive reference files (one per tool)
    ├── pagedjs.md
    ├── weasyprint.md
    ├── prince.md
    ├── typst.md
    ├── headless-chrome.md
    ├── puppeteer-playwright.md
    ├── latex.md
    ├── pandoc.md
    ├── react-pdf-programmatic.md
    ├── indesign-affinity.md
    └── wkhtmltopdf.md

.claude-plugin/
└── plugin.json                       # Claude Code plugin manifest
```

Each reference file in `references/tools/` covers a single renderer or tool family: version at time of writing, when to choose it, known limitations, and migration considerations.

---

## Installation

This skill is distributed as a Claude Code plugin. Install it from the Claude Code plugin registry or by pointing Claude Code at this repository.

Once installed, invoke the skill by describing your document stack or asking a tool-choice question. Claude Code will recognise the intent and load the skill automatically.

---

## Status

Version `1.0.1` — adds the `metadata.version` field; 1.0.0 was the initial release (May 2026).

The tool comparison matrix was last reviewed **2026-05-16**. Next review due **2026-08-16**. Run `/visual-stack-review review-matrix` to trigger a structured review against current tool release activity.
