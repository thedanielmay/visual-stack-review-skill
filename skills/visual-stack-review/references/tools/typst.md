# Typst — Deep Dive

**Version at time of writing:** Typst 0.12.x  
**License:** Apache 2.0 (open source, free)  
**Language:** Typst (own DSL); CLI and web editor  
**Install:** `brew install typst` / `cargo install typst-cli` / `winget install typst`

---

## What Typst actually is

Typst is a markup-based typesetting system designed as a modern alternative to LaTeX. It has its own language (not HTML, not Markdown — Typst markup) and its own rendering engine written in Rust. It does not use a browser and has no CSS. Understanding this is prerequisite to understanding its trade-offs: if you're thinking in HTML+CSS, Typst is a paradigm shift, not a tool upgrade.

Typst compiles `.typ` source files directly to PDF. There is no intermediate HTML, no CSS, no browser. The Typst language handles both content and layout — the equivalent of writing HTML and CSS merged into one file, but with a completely different syntax and paradigm.

---

## Quality ceiling for designed documents

**Honest ceiling:** Typst produces some of the best PDF output available — on par with LaTeX for text quality, but with a much faster compile-edit cycle and significantly better error messages. For documents that are primarily text with structured elements (tables, figures, callouts), Typst output quality is excellent.

The ceiling is hit when:
- You need pixel-perfect brand layouts with complex absolute positioning
- The source needs to be editable by non-developers (Typst syntax is learnable but not WYSIWYG)
- You have existing complex HTML templates that would require complete rewriting
- You need web (screen HTML) output from the same source — Typst outputs PDF only (HTML export is experimental and limited)

---

## What Typst is genuinely good at

**Compilation speed.** Typst compiles a 100-page document in ~1-2 seconds. LaTeX takes 10-30 seconds. Paged.js takes 30-120 seconds for the same page count. This matters for iterative authoring.

**Incremental compilation.** Typst tracks which parts of the document changed and only recompiles affected sections. `typst watch` gives live preview with sub-second updates.

**Math typesetting.** First-class native math support (no external packages required). Substantially easier to author than LaTeX math, on par with output quality.

**Tables.** Typst's table system is powerful and handles complex layouts (merged cells, header repetition, grid lines, styled cells) in a way that is explicitly specified and reliable. No guessing about what the renderer will do.

**Error messages.** Typst gives readable, precise error messages with line numbers. LaTeX's error messages are famously incomprehensible. This dramatically reduces debugging time.

**Scripting and data binding.** Typst has a built-in scripting language (functions, loops, conditionals, imports). You can generate table rows from arrays, conditionally include sections, parameterize documents. This makes it excellent for programmatic documents.

**Typography defaults.** Typst's default typography is excellent out of the box — good line breaking, hyphenation, kerning. You don't fight the defaults the way you do with CSS-based tools.

**Self-contained output.** Typst embeds all fonts and assets in the PDF. Distribution is simple.

---

## Where Typst loses to alternatives

**No CSS — steeper learning curve for web developers.** If your team knows HTML+CSS, they do not know Typst. The learning investment is real: Typst syntax, its layout model, its scripting language. Expect 1-2 weeks before a developer is productive.

**No HTML output (to speak of).** Typst outputs PDF. There is an experimental HTML export, but it is not suitable for production web use. If you need screen-first HTML AND print-quality PDF from one source, Typst is not the right tool.

**No WYSIWYG.** Typst is code-first. There is a web editor (typst.app) with live preview, and VS Code extension with preview, but there is no drag-and-drop layout. Designers who work in InDesign or Figma will find this alien.

**Ecosystem is young.** LaTeX has 40 years of packages. Typst's package ecosystem is growing but limited. If you need a niche layout feature (musical notation, chess diagrams, specialized academic formats), there may not be a Typst package yet.

**Rewriting existing HTML templates is a full migration.** Unlike switching between Paged.js and WeasyPrint (both of which use HTML+CSS), migrating to Typst means rewriting your templates in an entirely different language. All existing HTML, CSS, and templates are discarded.

**Complex absolute positioning.** Typst's layout model is flow-based with some absolute positioning support, but it is not as flexible as CSS absolute positioning for complex overlapping layouts. Brand-heavy designs with many precisely positioned elements are harder to achieve.

---

## When Typst shines specifically

- **Academic papers, reports, technical documentation** — the primary target audience and where the ecosystem is deepest
- **Data-driven documents** — invoices, analytics reports, generated PDFs from datasets, anything where the content comes from a database or API
- **Long-form structured documents** — 50+ page reports where consistent typography, automatic cross-references, and reliable TOC generation matter
- **Math-heavy documents** — no other tool matches Typst's math quality at this ease of authoring
- **Teams willing to learn the tool** — the investment pays off quickly in compile speed and reliability

---

## Migration cost: from Paged.js/HTML+CSS to Typst

**Effort: Very High** (this is not a migration, it is a rewrite)

**What changes:**
- Every template rewritten from HTML+CSS into Typst markup
- Build pipeline completely replaced (no more Node/npm, no browser)
- Anyone who authors content learns Typst syntax
- All CSS knowledge becomes inapplicable
- If you have JS-driven charts, need static SVG or Typst-native alternatives

**What you keep:**
- Content (body text can be imported from Markdown or plain text with some conversion work)
- Design intent (you will re-implement the design in Typst, not throw it away)
- Data pipeline (Typst can read JSON/CSV, so programmatic content generation is preserved)

**One-time vs ongoing:**  
Very high one-time cost. Ongoing authoring cost is similar or lower once the team knows Typst — the scripting language and fast compilation make iteration faster than HTML+CSS tools.

**When this migration is worth it:**
- The document is produced repeatedly (monthly, per-client) — the one-time rewrite cost is amortized
- Render speed matters (Typst is 10-50× faster than Paged.js for large documents)
- The document is primarily text with structured elements (not complex brand layouts)
- The team is willing to invest in learning

**When this migration is not worth it:**
- One-off document that won't be reproduced
- Complex brand layout with precise absolute positioning
- Team is design-first, not code-first
- You need screen HTML output too

---

## Migration cost: from LaTeX to Typst

**Effort: High** (not Medium — see below)

The template rewrite is only part of the cost. The deeper cost is the **ecosystem gap**, which grows with project complexity:

**Citation and bibliography:** Typst uses Hayagriva (`.yml` format) instead of BibTeX/BibLaTeX. Hayagriva has no parity with BibLaTeX's `authoryear`, `authortitle`, `verbose-ibid`, and other citation styles used in humanities, law, and social sciences. If your project uses BibLaTeX with a non-default citation style, you will either write a custom Typst CSL or compromise on citation style. This is not a small fix.

**CJK and RTL scripts:** Typst's CJK (Chinese, Japanese, Korean) support is improving but not at LaTeX's maturity. Complex CJK documents with furigana, vertical text, or specialized dictionaries should not migrate until Typst's CJK packages reach LaTeX-equivalent depth. RTL (Arabic, Hebrew) is similarly in-progress.

**Package ecosystem depth:** LaTeX has 40 years and 6,000+ CTAN packages. Typst has a growing registry but a fraction of that coverage. Niche layout features (musical notation, chess boards, specialized legal citation formats, linguistics glosses) may not have Typst equivalents.

**Community knowledge:** 20 years of LaTeX Stack Overflow answers do not transfer. Your team's muscle memory for debugging LaTeX is worthless in Typst. Expect a 2-4 month learning curve before productivity recovers.

**Collaboration:** LaTeX has Overleaf (real-time collaborative editing in a browser). Typst has typst.app (similar, but smaller user base). If your team or co-authors use Overleaf as a workflow dependency, migration disrupts that.

**Rate as High** for any project with: substantive citations in a humanities/legal style, CJK or RTL content, niche package requirements, or collaborative authoring via Overleaf. Rate as **Medium** only for a project that is primarily programming-style (algorithms, computer science) with simple bibliography needs and a single author.

---

## When to choose Typst

- Long-form technical or academic documents
- Documents generated from structured data at scale
- You need the fastest compile times in the ecosystem
- Math typesetting is a requirement
- You're starting fresh (not migrating an existing HTML+CSS template)
- The team is technical and willing to learn a new tool
