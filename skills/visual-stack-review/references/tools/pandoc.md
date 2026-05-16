# Pandoc — Deep Dive

**Version at time of writing:** Pandoc 3.x  
**License:** GPL (open source, free)  
**Language:** Haskell (binary); used from any environment via CLI  
**Install:** `brew install pandoc` / `apt install pandoc` / download binary

---

## What Pandoc actually is

Pandoc is a universal document converter. It converts between Markdown, HTML, DOCX, ODT, EPUB, LaTeX, PDF, and 40+ other formats. It is not a document production tool in its own right — it is a translation layer that connects authoring formats to production formats.

**In the context of PDF production**, Pandoc is almost always a pipeline component, not the end tool. When you "generate a PDF with Pandoc," what actually happens is:

1. Pandoc converts your source (Markdown, RST, DOCX) to LaTeX
2. LaTeX compiles to PDF via pdflatex/xelatex/lualatex

OR (with `--pdf-engine=weasyprint` or `--pdf-engine=prince`):

1. Pandoc converts your source to HTML
2. WeasyPrint/Prince/Paged.js renders to PDF

The PDF quality ceiling is entirely determined by the backend engine, not by Pandoc itself.

---

## Pandoc's genuine strengths

**Format conversion.** The best tool available for converting between document formats. Markdown → DOCX → PDF → EPUB from a single source — Pandoc handles this better than anything else.

**Markdown → LaTeX → PDF.** The killer workflow for academic writing: author in Markdown (readable, version-controllable), Pandoc handles the LaTeX translation, LaTeX handles typesetting. This gives you LaTeX output quality without writing LaTeX. The `pandoc-crossref` filter adds cross-references. `citeproc` handles citations from BibTeX files.

**Templates.** Pandoc has a template system that allows you to control the LaTeX (or HTML) output. A custom LaTeX template lets you set up the document class, fonts, margins, and layout once — then apply it to any Markdown source.

**Filters (Lua and Python).** Pandoc's filter system lets you transform the document's abstract syntax tree programmatically. This is how you handle things like "replace all figure captions with numbered captions" or "add a watermark to every page" — without modifying the source format.

**CI-friendly.** A single binary with no runtime dependencies (beyond the backend engine). Easy to run in Docker, GitHub Actions, and other CI environments.

---

## Where Pandoc falls short

**PDF quality depends entirely on the backend.** Pandoc's own contribution to PDF output is zero — it's a converter, not a renderer. If your LaTeX template is poor, the PDF looks poor. If you use WeasyPrint as the backend, WeasyPrint's limitations apply.

**Template complexity.** Creating a polished LaTeX template for Pandoc is as hard as writing LaTeX. The abstraction saves you from writing LaTeX *content*, not from understanding LaTeX *structure*.

**HTML-to-PDF via Pandoc is roundabout.** Pandoc → HTML → WeasyPrint works, but you're adding a conversion step that can introduce formatting loss. If you're starting from HTML, go directly to WeasyPrint rather than routing through Pandoc.

**Pandoc Markdown is not standard Markdown.** Pandoc has its own Markdown extensions (divs with `:::`, attributes like `{.class}`, footnotes, citations). These extensions are useful but create source files that don't render correctly in standard Markdown viewers.

**No visual preview.** Pandoc is a one-shot converter — you write, you convert, you check the PDF. No live preview in the authoring environment (unlike Typst's live preview or HTML authoring in a browser).

---

## Pandoc in the document production ecosystem

### Pandoc as the source-format bridge

The most valuable use of Pandoc is as an ingestion layer:
- **Client sends DOCX** → Pandoc converts to Markdown → you edit and style in your pipeline
- **Legacy HTML content** → Pandoc converts to Markdown → source-controlled, editable
- **Author writes in Markdown** → Pandoc converts to LaTeX → LaTeX compiles to print-quality PDF

### Pandoc in academic workflows

The dominant stack for academic PDF production:
```
Markdown source (with citations as @key)
  + .bib bibliography file
  + Pandoc template (LaTeX)
  + pandoc-crossref filter
  ↓ pandoc --pdf-engine=xelatex --filter pandoc-crossref --citeproc
  → Publication-quality PDF
```

This is how many researchers produce papers — write in Markdown/RMarkdown, generate LaTeX-quality PDFs. The key insight: the author never writes LaTeX, but benefits from LaTeX's typesetting quality.

### Pandoc with Quarto

Quarto is a modern scientific publishing system built on Pandoc. It extends Pandoc's Markdown with:
- Code cells that execute (R, Python, Julia, Observable)
- Cross-references (`@fig-`, `@tbl-`, `@sec-`)
- Multiple output formats from one source (PDF, HTML, DOCX, slides)
- Rich theming system

If you're in a data science / academic context and considering Pandoc, evaluate Quarto first — it's a significant improvement in usability. See `references/tools/quarto.md` when that file is created.

---

## When to choose Pandoc

- You author in Markdown and need LaTeX-quality PDF output (academic/technical writing)
- You need to convert between document formats as part of a pipeline
- You receive documents in multiple formats and need a unified output
- You want the benefits of LaTeX typesetting without writing LaTeX source

## When NOT to choose Pandoc as your primary tool

- You need a visual design tool (Pandoc's output is typeset, not designed)
- You're starting from HTML and producing PDF (go directly to WeasyPrint/Prince/Paged.js)
- Your team doesn't know Markdown or LaTeX (the template system requires LaTeX knowledge to customize)
- You need live visual preview during authoring

---

## Detection signals

- `pandoc` command in Makefile or build scripts
- `.md` source files + `pandoc.yaml` or `_quarto.yml` in the project root
- `defaults.yaml` (Pandoc defaults file) in the project
- YAML frontmatter in Markdown files with `pdf-engine:`, `template:`, or `bibliography:` keys
- PDF metadata: `Creator: pandoc` followed by LaTeX producer fields
