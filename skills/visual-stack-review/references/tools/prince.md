# PrinceXML — Deep Dive

**Version at time of writing:** Prince 15.x  
**License:** Commercial ($$$)  
**Language:** Any (CLI tool, accepts HTML+CSS)  
**Cost:** ~USD $495 for a standalone server license (one-time); ~$50/month SaaS (DocRaptor)  
**Install:** Download from princexml.com; or use DocRaptor API

---

## What Prince actually is

Prince (PrinceXML) is a commercial HTML-to-PDF engine with its own CSS layout implementation — like WeasyPrint but significantly more mature and with better CSS Paged Media support. It is the reference implementation for much of the CSS Paged Media specification. It does not use a browser.

Prince has been developed since 2003 and is used by major publishers (O'Reilly books are printed using Prince). It is the tool professionals reach for when output quality is the only thing that matters and cost is not a constraint.

---

## Quality ceiling for designed documents

**Honest ceiling:** Prince has the highest quality ceiling of any HTML-to-PDF tool currently available. Output is at or above the quality of professionally typeset InDesign documents for text-heavy work, and competitive for design-heavy work. The ceiling is not meaningfully hit for document types — it is hit only by:
- Extreme brand designs requiring bespoke InDesign layout work
- The licensing cost

---

## What Prince is genuinely good at

**Everything in the CSS Paged Media spec, done correctly.** Running headers and footers, string sets, named pages, generated content, `@page` rules — all work reliably. Prince wrote much of the spec; it implements it most accurately.

**Table handling.** The best table-to-PDF rendering of any HTML engine. Header repetition, cell merging, table-spanning figures, multi-page tables, footnotes within tables — all handled correctly and predictably.

**Widow and orphan control.** `orphans` and `widows` CSS properties implemented correctly. `break-before`, `break-after`, `break-inside` all work. Page breaks are placed correctly for long-form text.

**Footnotes.** True CSS footnote support (`float: footnote`) works correctly and reliably, including multi-column footnotes. This is rare among HTML-to-PDF tools.

**Cross-references and TOC.** `target-counter()` and `target-text()` for generating page numbers in cross-references and TOC entries work correctly. Building a TOC from headings is a first-class workflow.

**Font handling.** Prince handles font embedding, subsetting, and fallback correctly. PDF output passes preflight checks for professional print production.

**PDF/X spec output.** Prince can emit PDF/X-1a and PDF/X-3. Note: X-1a and X-3 are 2001/2002-era specs — many commercial printers, especially in AU/EU, now require **PDF/X-4** (2010, supports live transparency). Confirm with your printer which spec they require before choosing a tool based on PDF/X capability.

**Spec conformance ≠ press-readiness.** A Prince-generated PDF/X-1a file validates against the spec but is not automatically ready for press. You still need: BleedBox + TrimBox set at 3mm offset from trim edge (via `-prince-pdf-page-layout` or CSS `@page { bleed: 3mm; marks: crop; }`), an ICC OutputIntent profile matching the press condition (e.g. FOGRA51 coated, GRACoL 2006), total area coverage within the printer's limit (typically ≤300% coated, ≤260% uncoated), overprint behaviour verified, and spot colours named exactly to the printer's ink library. A prepress review step remains required for commercial press work regardless of the tool used.

**Alternative path:** Ghostscript can post-process PDFs from any tool (Paged.js, WeasyPrint, Chrome) to PDF/X-4. "Prince is required for PDF/X at a commercial press" is false. Prince's advantage is that it makes the correct output easier to achieve — not that it is the only path.

**PDF/A (archival) and tagged PDF (accessibility)** are also supported.

**Performance.** Fast. A 200-page document renders in seconds. Significantly faster than Paged.js for large documents.

**`-prince-*` extensions.** Prince has its own CSS extensions (prefixed `-prince-`) for features the spec doesn't cover yet: image resolution control, figure float placement, PDF bookmarks from headings, bleed and crop marks, spot colors.

---

## Where Prince loses to alternatives

**Cost.** $495 for a standalone license is a real barrier for small projects or individual developers. The SaaS offering (DocRaptor) charges per render, which adds up for high-volume generation. Paged.js and WeasyPrint are free.

**Proprietary.** Prince is not open source. If the company discontinues development or is acquired, you are exposed. This matters for long-term infrastructure decisions.

**Less CSS flexibility than Paged.js/Chrome.** Prince implements the CSS Paged Media spec correctly, but it is not Chromium. Very modern CSS features (container queries, `@layer`, some newer grid syntax) may not be available or may behave differently. Documents designed to look good in a browser may need CSS adjustments for Prince.

**No JavaScript.** Like WeasyPrint, Prince does not execute JavaScript. Dynamic content, JS-rendered charts, and React components will not render.

**Community and support.** Smaller community than Chrome-based tools. Fewer StackOverflow answers, fewer examples online. Official support is available but paid.

---

## Licensing model in detail

**Standalone license:** One-time purchase (~$495 at current pricing). Runs on your own servers. Unlimited renders. No per-render cost. This is the right choice for internal tooling that renders frequently.

**Personal license:** ~$49/year. Single developer, non-commercial use. Good for evaluation or side projects.

**SaaS via DocRaptor:** Per-render pricing starting at ~$15/month for 125 renders. No server setup. Good for low-volume production use or prototyping.

**Free for non-commercial:** Prince is free to use for non-commercial purposes. Output includes a small watermark that can be removed with a license.

---

## Migration cost: from Paged.js to Prince

**Effort breakdown:**
- **Source rewrite:** Low — HTML content unchanged; CSS audit required but mostly transfers
- **Layout-engine retuning:** High — this is where the real cost lives. Specific incompatibilities:
  - **Paged.js JavaScript hooks have no Prince equivalent.** `Paged.Handler`, `beforeParsed`, `afterRendered`, `afterPageLayout`, `afterRendered` — all disappear. Any logic in these hooks must be reimplemented as server-side preprocessing, XSLT, or Prince JavaScript (which runs in Prince's own JS environment, not a browser — `window`, `document`, and most Web APIs are unavailable)
  - **Break heuristics differ.** Prince's C++ layout engine places page breaks differently from Paged.js's JavaScript polyfill, particularly inside flex/grid containers and around floated elements. Every page-break-sensitive section needs visual regression testing
  - **`@page :nth()` selector support differs.** Paged.js supports `:nth()` for targeting specific pages by number; Prince's implementation may diverge on edge cases
  - **SVG rendering diverges.** SVGs that render correctly in Chromium (Paged.js's host) may render differently in Prince's own SVG engine, particularly complex gradients, clip-paths, and `<use>` elements with transforms
  - **Font rendering differs.** Paged.js inherits Chrome's font rendering; Prince has its own. Kerning pairs, ligatures, and OpenType feature application may produce visible differences
- **Build pipeline:** Low — `pagedjs-cli input.html -o output.pdf` → `prince input.html -o output.pdf`
- **Ecosystem regression risk:** Medium — loss of Paged.js bug workarounds (which become unnecessary but must be identified and removed); loss of Chrome DevTools debugging for layout issues
- **Author retraining:** Low — HTML+CSS workflow unchanged

**What you keep:**
- All HTML content
- Most CSS (standard selectors, typography, flexbox, `@page` rules)
- The authoring workflow

**One-time vs ongoing:** High one-time cost for layout-engine retuning and regression testing. Ongoing authoring is identical once stable.

**Realistic timeline:** Budget 4–8 weeks for a document with non-trivial pagination, JS hooks, or complex SVGs. Not 2 weeks. Teams that have bid this at 2 weeks have regretted it.

---

## Migration cost: from WeasyPrint to Prince

**Effort: Low**

Both tools use HTML+CSS without JavaScript. The CSS Paged Media rules are largely compatible.

**What changes:**
- Replace `weasyprint` Python call with `prince` CLI call (or subprocess call in Python)
- Audit CSS for WeasyPrint-specific workarounds (e.g. background-color on `@page` instead of body — Prince may handle this differently)
- Test fonts: Prince handles WOFF2 and web fonts differently from WeasyPrint

**What you keep:**
- All HTML content
- Most CSS
- CSS Paged Media rules

---

## When to choose Prince

- Output quality is the primary constraint (professional print production)
- The document goes to a commercial press (requires print-ready PDF)
- Complex tables, footnotes, or cross-references are required
- The licensing cost is acceptable
- You're currently struggling with Paged.js pagination bugs that WeasyPrint also has
- Long-form documents (books, formal reports) where typography quality matters
