# Paged.js — Deep Dive (Stack Choice Perspective)

**Note:** For debugging Paged.js bugs in an existing document, see `visual-review/references/pagedjs.md`. This file is about whether Paged.js is the right tool to use in the first place.

**Version at time of writing:** Paged.js 0.4.x / pagedjs-cli 0.4.x  
**License:** MIT (open source, free)  
**Language:** JavaScript / Node.js  
**Install:** `npm install -g pagedjs-cli` or `npm install pagedjs`

---

## What Paged.js actually is

Paged.js is a JavaScript polyfill that implements the W3C CSS Paged Media specification inside a browser (Chrome/Chromium). It wraps your HTML content in `.pagedjs_page` containers, handles pagination, generates margin boxes for running headers and footers, and then the browser's native print function renders the result to PDF.

The key architectural fact: Paged.js runs **inside Chrome**. This means you get full CSS support (because Chrome handles all your CSS), but you also get Chrome's quirks — and Paged.js's quirks layered on top. The debugging model requires understanding both.

`pagedjs-cli` is the command-line wrapper that handles starting a headless Chrome, loading Paged.js, waiting for pagination to complete, and outputting the PDF.

---

## Quality ceiling for designed documents

**Honest ceiling:** Paged.js produces excellent output for designed, brand-heavy documents when its quirks are understood and worked around. The CSS flexibility (because Chrome handles all CSS) makes it the best choice for documents that require complex visual design. The ceiling is hit by its known systematic bugs — all documented and workaround-able, but requiring CSS discipline.

---

## What Paged.js is genuinely good at

**Full modern CSS support.** Because it runs inside Chrome, Paged.js gives you access to every CSS feature Chrome supports. Grid layouts, container queries, CSS custom properties, blend modes, modern color spaces, variable fonts — all available. No other pagination tool offers this.

**Designed/brand-heavy documents.** The combination of full CSS support + CSS Paged Media for running headers/footers makes Paged.js the best choice for documents where visual design is primary. InDesign-level visual control is achievable.

**Browser-based preview.** Load your HTML in Chrome (with the Paged.js polyfill) and see exactly how the PDF will look. No other tool offers this live-preview workflow. This dramatically speeds up design iteration.

**CSS Paged Media features.** `@page` rules, `running()`, `string()`, named pages, page counters, `@top-left`/`@bottom-right` margin boxes — all supported. This is what makes dynamic running headers and CSS-generated page numbers possible.

**JavaScript-rendered content.** Because Chrome executes JavaScript before Paged.js paginates, Chart.js, D3, and other JS-rendered charts work. Static SVGs are not required.

**Node.js ecosystem.** If your pipeline is in Node/npm, Paged.js integrates naturally. `pagedjs-cli` is a one-line command.

---

## Where Paged.js loses to alternatives

**Known systematic bugs.** Paged.js has a set of documented, reproducible bugs that affect many documents:
- Background colours don't fill the full page including margin boxes (PJ-1)
- Header clearance inconsistencies on continuation pages (PJ-2)
- `border-top` on block elements repeats on every continuation page (PJ-3)
- Flowing sections can overflow into footer margin bands (PJ-4)
- Section openers sometimes don't get the right page break (PJ-5)
- Long-form sections lose headers/footers on continuation pages (PJ-6)

These bugs have CSS workarounds, but they require knowing the bugs exist and actively defending against them. A new document starts clean and accumulates workarounds as bugs are hit.

**Table handling is unreliable.** Multi-page tables with repeating headers, complex cell merging, and `break-inside: avoid` on rows are the most common source of visual bugs. WeasyPrint and Prince handle tables more reliably.

**Render performance.** Starting a headless Chrome browser takes 1-3 seconds. For large documents (50+ pages), pagination can take 30-120 seconds. For documents generated at scale (hundreds per day), this adds up. Typst and WeasyPrint are 5-20× faster.

**Debugging complexity.** When something looks wrong, you may be debugging: your HTML structure, your CSS, a Paged.js bug, a Chrome quirk, or the interaction between all four. The debugging surface is large.

**No print-production PDF/X output.** Paged.js cannot output PDF/X-1a or PDF/X-3 (commercial print standards). If the document goes to a commercial press, Prince is the right tool.

**`break-inside: avoid` reliability.** CSS break properties work better in Paged.js than in pure Chrome, but are still less reliable than Prince or Typst for long-form text with complex pagination requirements.

---

## The right mental model for Paged.js

Paged.js is the right tool when:
1. Design quality matters (complex brand layouts, multi-column, full-bleed)
2. You're already in the Node/JavaScript ecosystem
3. You can accept and manage its known bugs with CSS discipline
4. You need live browser preview during authoring

Paged.js is the wrong tool when:
1. Complex tables are the primary content — WeasyPrint or Prince handle these better
2. You need print-ready PDF (Prince is required for PDF/X)
3. Render speed is a constraint at scale — use Typst or WeasyPrint
4. Your team doesn't know CSS well enough to debug the known bugs

---

## Migration cost: from Paged.js to WeasyPrint

**Effort: Medium**

See `references/tools/weasyprint.md` for the full breakdown. The short version:
- Remove Paged.js script, replace with WeasyPrint CLI call
- Audit for JS-rendered content (charts, dynamic elements) — these need static SVG alternatives
- Most CSS transfers; test tables carefully (they'll behave better)
- Build pipeline change: Node → Python

**Migration benefit:** Table rendering improvements, elimination of PJ-1 through PJ-7 bugs, simpler build pipeline, faster render times, more predictable output.

**Migration cost:** Loss of modern CSS features (container queries, etc.), loss of JS-rendered content, Python ecosystem required.

---

## Migration cost: from Paged.js to Prince

**Effort: Low**

Both tools use HTML+CSS with CSS Paged Media. This is the easiest migration in the ecosystem.

See `references/tools/prince.md` for the full breakdown. The short version:
- Remove Paged.js script, replace with `prince` CLI call
- CSS Paged Media rules transfer directly (Prince implements the standard correctly)
- Test for CSS extensions you may need (`-prince-*` prefix for some features)
- No JavaScript (same constraint as WeasyPrint)

**Migration benefit:** Eliminates all Paged.js bugs, best-in-class table handling, print-ready PDF output, better widow/orphan control, faster renders.

**Migration cost:** Commercial license cost, no JavaScript support.

---

## Migration cost: from Paged.js to Typst

**Effort: Very High** — see `references/tools/typst.md`. This is a full rewrite, not a migration.

---

## Stay + fix assessment for Paged.js

If you're currently using Paged.js and considering whether to migrate:

**The bugs are manageable.** Every known Paged.js bug has a CSS workaround. A Paged.js document that has been through a careful visual review and bug-fixing cycle produces excellent output. The question is whether you want to accept ongoing maintenance of those workarounds.

**Stay if:**
- Your document is design-heavy and relies on modern CSS features (grid, container queries)
- Your team knows the bugs and has working workarounds
- You need JS-rendered content (charts that can't be made static)
- You value the live browser preview workflow
- You don't need print-ready PDF/X output

**Migrate to WeasyPrint if:**
- Tables are central and causing pain
- You're in the Python ecosystem
- You want simpler, more predictable rendering

**Migrate to Prince if:**
- You need the highest quality output
- Tables and long-form text are primary content
- Licensing cost is acceptable
- You don't need JavaScript-rendered content

**Migrate to Typst if:**
- Performance at scale matters
- The document is text-primary (not complex brand layout)
- You're willing to invest in learning a new tool
