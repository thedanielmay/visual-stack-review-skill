# Stack detection reference

Load this file during Phase 1. It holds the detection signals, the reliability caveats and the stack detection output format.

Detect as much as possible before asking the user anything. Document your findings and confidence level for each signal.

Extract Producer and Creator fields from a `pdfinfo`-readable PDF. These are the most reliable stack signals.

---

## Detection signals by stack

**Paged.js**
- package.json: `"pagedjs"` or `"pagedjs-cli"` in dependencies
- HTML source: the polyfill script tag `<script src="paged.polyfill.js">` or `import 'pagedjs'`
- PDF metadata: `Producer: pagedjs-cli` or `Producer: Paged.js`
- Runtime DOM: the `.pagedjs_page` class appears after the polyfill runs. Inspect the rendered page, not the source files.
- Paged.js runs inside Chrome, so Chrome-style PDF metadata can appear too. Name Paged.js from the dependency, the script tag or the PDF Producer.

**WeasyPrint**
- Python requirements: `weasyprint` in requirements.txt or pyproject.toml
- PDF metadata: `Producer: WeasyPrint` (includes version)
- HTML source: WeasyPrint-specific CSS comments, or lack of JS-dependent features
- Build scripts: `weasyprint input.html output.pdf` in Makefile or shell scripts

**Prince / PrinceXML**
- PDF metadata: `Producer: Prince` (very distinctive)
- CSS: `-prince-*` vendor-prefixed properties
- HTML: `<?xml-stylesheet ... ?>` processing instructions common in Prince workflows
- Build: `prince input.html -o output.pdf`

**Headless Chrome (bare, or via Puppeteer / Playwright)**
- package.json: `"puppeteer"` or `"playwright"` in dependencies. Check whether `"pagedjs"` is present as well.
- Build scripts: `page.pdf()` calls and `--headless` flags. A bare Chrome build calls `page.pdf()` without `waitForFunction(() => window.PagedPolyfill?.done)`.
- PDF metadata: `Producer: Skia/PDF mNNN` with `Creator: Chromium` or `HeadlessChrome`.
- Chrome supports named pages with the `page` property and, as of Chrome 131, `@page` margin boxes with page counters. Treat `@page` CSS as a paged-media signal, not as proof of Paged.js.
- Bare Chrome has no Paged.js polyfill script tag and no Paged.js PDF Producer. Use those signals to tell bare Chrome from Paged.js.

**wkhtmltopdf** ⚠️ UNMAINTAINED
- PDF metadata: `Producer: Qt` or `wkhtmltopdf`
- Build scripts: `wkhtmltopdf` commands in Makefile, Rakefile, or shell scripts
- Gems/packages: `wicked_pdf` (Rails), `pdfkit` (Python/Ruby)
- Signal: older projects (pre-2020), often Rails/Django stacks
- IMPORTANT: if detected, recommend moving away. See `references/tools/wkhtmltopdf.md` for the single stay rule and the migration cost.

**Paged-media engine (shared CSS features, engine-agnostic)**
- CSS: `@page`, the `page` property, `running()`, `string()`, `@top-center` and other margin boxes
- Paged.js, WeasyPrint and Prince all use these rules
- These rules show that a paged-media engine handles the layout. They do not name the engine.
- Name the engine from package dependencies, build commands, the polyfill script tag or PDF metadata

**LaTeX / XeLaTeX / LuaLaTeX**
- Source files: `.tex`, `.sty`, `.cls` extensions
- PDF metadata: `Producer: pdfTeX`, `XeTeX`, `LuaTeX`, or `MiKTeX`
- Build: `latexmk`, `pdflatex`, `xelatex`, `lualatex` in Makefile
- Sub-variant detection: `pdfTeX` → pdfLaTeX; `XeTeX` → XeLaTeX (modern fonts); `LuaTeX` → LuaLaTeX (scripting)
- Pandoc pipeline signal: `.md` source + `pandoc` in build scripts + LaTeX metadata in PDF

**Typst**
- Source files: `.typ` extension
- PDF metadata: `Producer: typst`
- Build: `typst compile` commands
- Config: `typst.toml` or `.typstrc` in project root

**InDesign / Affinity Publisher / GUI layout tools**
- PDF metadata: `Creator: Adobe InDesign CC 20xx` → InDesign
- PDF metadata: `Creator: Affinity Publisher` → Affinity Publisher
- No editable source code — layout tool output; binary `.indd` / `.afpub` files
- Signal: very high-quality font subsetting, named spot colors in PDF metadata, ICC profiles embedded
- Signal: `Trapped: False` and bleed box metadata → print-production intent

**Word / LibreOffice**
- PDF metadata: `Creator: Microsoft Word` or `Creator: LibreOffice Writer`
- Source: `.docx`, `.odt` files
- Signal: `Tagged: Yes` (Word exports tagged PDFs by default), revision metadata visible

**ReportLab / fpdf2 / programmatic Python PDF**
- Python requirements: `reportlab`, `fpdf2`, `borb` in requirements.txt or pyproject.toml
- PDF metadata: `Producer: ReportLab PDF Library` or `Producer: fpdf2`
- Source: Python files calling `canvas.drawString()`, `Paragraph()`, `SimpleDocTemplate()`
- Signal: very structured tabular layout, no HTML source

**Docusaurus / Hugo / Sphinx / other static site tools** (for HTML output, not PDF)
- Source: `docusaurus.config.js`, `config.yaml` with `theme:`, `conf.py` with Sphinx imports
- Build: `npm run build`, `hugo`, `make html`
- Note: these produce HTML, not PDF — a separate PDF step is needed

**React-PDF / pdf-lib / jsPDF** (programmatic PDF generation)
- package.json: `"@react-pdf/renderer"`, `"pdf-lib"`, `"jspdf"`
- Source: JSX/TSX with `<Document>`, `<Page>`, `<View>` components (React-PDF)
- Signal: very structured data-driven output, no HTML+CSS source
- Anti-pattern signal: `html2canvas` + `jsPDF` in the same codebase — see rasterization anti-pattern in `references/tools/react-pdf-programmatic.md`

**Pandoc pipeline**
- Source: `.md` files with YAML frontmatter containing `pdf-engine:`, `template:`, or `bibliography:`
- Build: `pandoc` commands in Makefile, often with `--pdf-engine=xelatex` or `--pdf-engine=weasyprint`
- Config: `pandoc.yaml` defaults file, `_quarto.yml` (Quarto project)
- PDF metadata: inherits from backend engine (LaTeX, WeasyPrint, etc.) — check Producer

### Detection reliability — caveats (read before labelling confidence)

Detection signals can and do mislead. Common failure modes:

- **Monorepos:** `pagedjs` may sit in a sibling workspace with no relationship to the PDF being reviewed. Always check which workspace produced the artifact.
- **CDN-loaded tools:** Paged.js is frequently loaded via `<script src=".../paged.polyfill.js">` with no npm entry at all. Absence of `pagedjs` in package.json does NOT mean Paged.js isn't in use.
- **Dead dependencies:** Legacy projects carry unused deps for years. A package in package.json is evidence, not proof.
- **Shared Chrome metadata:** PDFs made by Chrome and Puppeteer usually carry `Producer: Skia/PDF mNNN` and `Creator: Chromium` or `HeadlessChrome`. Do not key Chrome detection on `Producer: Chromium`. That metadata does not name the pagination engine. Name Paged.js from the package dependency, the polyfill script tag or `Producer: pagedjs-cli` or `Producer: Paged.js`. Check the rendered DOM for `.pagedjs_page`, since the class does not exist in the source files.
- **Pandoc pipelines:** PDF metadata reflects the backend engine (LaTeX, WeasyPrint), not Pandoc itself. Check for `pandoc` in build scripts separately.

**Rule:** Always corroborate package.json against PDF metadata AND at least one HTML/source signal before labelling confidence above `possible`. If signals disagree, list them all and ask the user.

### Detection output format

After running detection, produce this summary:

```
## Stack detection

Primary stack:    [tool name]
Signal evidence:
  - [signal 1] → [what it tells us]
  - [signal 2] → [what it tells us]
  - [any contradicting signals → noted]

Confidence:      [confident (3+ corroborating signals, no contradictions) |
                  likely (2 signals, minor contradictions) |
                  possible (1 signal or conflicting signals) |
                  unknown (no signals — ask)]

Secondary tools: [e.g. "CSS preprocessor: SCSS", "Build system: Make", "Template engine: Jinja2"]

Document type:   [see Phase 2]
Output targets:  [detected: screen PDF | print | web | unknown]

Questions needed before full assessment: [list only what can't be inferred]
```
