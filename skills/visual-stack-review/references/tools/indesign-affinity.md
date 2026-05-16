# InDesign / Affinity Publisher — Deep Dive

This file covers desktop layout applications used for professional print and screen PDF production. These are fundamentally different from code-based tools — they are GUI applications, not programming tools.

---

## Adobe InDesign

**License:** Commercial — Adobe Creative Cloud subscription (~$55 USD/month standalone, or ~$60/month all-apps)  
**Language:** N/A (GUI application)  
**Platform:** macOS, Windows  
**PDF metadata signal:** `Creator: Adobe InDesign CC 20xx`

### What InDesign actually is

InDesign is the professional standard for page layout. It is used by graphic designers, publishers, and print production houses for magazines, books, brochures, annual reports, and high-end marketing collateral. It is not a code-based tool — there is no HTML, no CSS, no source file that a developer would recognise.

InDesign's output quality for print is the benchmark against which all other tools are measured. Its font rendering, colour management, and print production capabilities (bleeds, crop marks, colour profiles, PDF/X output) are used directly by commercial printers worldwide.

### Where InDesign wins decisively

**Print production.** The only tool in this comparison that a commercial printer will accept without question. Built-in preflight checks, correct bleed and crop mark generation, ICC colour profile embedding, spot colour support, PDF/X-1a and PDF/X-4 output — all first-class.

**Typography.** InDesign's optical margin alignment, balanced ragged right, Adobe Paragraph Composer (equivalent to Knuth-Plass), and comprehensive OpenType feature support produce the best text typesetting of any tool outside LaTeX. For long-form prose in a visual layout, InDesign and LaTeX are peers — each better in their respective domain.

**Complex visual layouts.** Multi-column, mixed orientations, bleed, wrap-around graphics, text frames with arbitrary shapes — InDesign handles these natively because they are its core feature set. HTML+CSS tools approximate InDesign; InDesign is the original.

**Master pages and templates.** Running headers, page numbers, recurring design elements across pages — all managed through master pages in a WYSIWYG environment. No CSS, no code, immediate visual feedback.

**Book assembly.** Long documents composed of multiple files (each chapter as a separate file, assembled into a Book) with shared styles, shared TOC, and unified page numbering — InDesign's Book feature handles this correctly.

**Interactive PDFs.** Buttons, hyperlinks, animations, video embeds, form fields — InDesign can export interactive PDFs for screen use. These are different from fillable AcroForms but serve design-led interactive documents.

### Where InDesign loses

**No code.** There is no source file that can be version-controlled meaningfully. An InDesign `.indd` file is a binary. You cannot diff it, you cannot do code review, you cannot generate it from a database (without significant ESTK/ExtendScript work or InDesign Server).

**Automation requires InDesign Server.** Generating PDFs from InDesign in a CI/CD pipeline requires InDesign Server — an additional expensive license (~$2,000/year) plus infrastructure. For programmatic generation from data, this is prohibitive.

**Designer-only authoring.** Non-designers cannot reasonably author in InDesign. If the content is written by a team of writers and designed by a designer, the workflow requires a handoff step (copy paste or import from Word).

**Updates can break documents.** An `.indd` file from InDesign 2019 may need updating to open correctly in InDesign 2024. Long-lived templates require maintenance.

**Cost.** $55/month per seat, every seat, forever. For a team of 5 designers, that's $3,300/year for InDesign alone.

### Detection and reverse-engineering

When given a PDF produced by InDesign:
- `pdfinfo` → `Creator: Adobe InDesign CC 20xx`
- Font list shows embedded subset fonts with distinctive naming patterns
- Colour profiles embedded correctly
- PDF version is typically 1.5 or higher
- Named pages, bleeds, and crop marks if it was a print document

**InDesign-produced PDFs are read-only from a code perspective.** You can extract text with pdftotext, but you cannot edit the layout without InDesign. If the user needs to modify the document, they need access to the original `.indd` file.

### When to recommend InDesign

- Print production is the primary output (commercial press, offset printing)
- Visual design quality is the primary constraint
- A professional designer will author and maintain the document
- The document is published periodically (magazine, annual report) rather than generated programmatically
- The organization already has Adobe Creative Cloud

---

## Affinity Publisher

**License:** One-time purchase (~$70 USD, no subscription)  
**Language:** N/A (GUI application)  
**Platform:** macOS, Windows, iPad  
**PDF metadata signal:** `Creator: Affinity Publisher`

### What Affinity Publisher is

Affinity Publisher is the professional InDesign alternative from Serif. It is a full professional page layout application with no subscription fee — one perpetual license per platform. Feature parity with InDesign is approximately 80-85% for most print production workflows.

### Where Affinity Publisher matches InDesign

- PDF/X output (PDF/X-1a, PDF/X-3, PDF/X-4)
- Bleed and crop marks
- Master pages and facing pages
- CMYK colour space and colour profile management
- OpenType font features
- Multi-column text frames
- Paragraph and character styles
- Linked content (linked text frames, linked assets)

### Where Affinity Publisher lags behind InDesign

- **Ecosystem.** InDesign has 30 years of plugins, scripts, and production workflows. Affinity Publisher has a fraction of this.
- **Book assembly.** No equivalent to InDesign's Book feature. Long documents must be managed as a single file.
- **InDesign Server equivalent.** No server product for programmatic generation.
- **Third-party compatibility.** Print houses and agencies standardised on InDesign. Delivering an Affinity Publisher file to an external printer may require exporting to PDF/IDML; the `.afpub` format is not widely accepted.
- **Data merge.** Affinity Publisher's data merge (for variable data printing) is basic compared to InDesign's.
- **Script/automation support.** Limited scripting compared to InDesign's ExtendScript/JavaScript API.

### The price advantage is real

For a designer or small studio producing marketing collateral, brochures, and annual reports, Affinity Publisher at $70 (perpetual) versus InDesign at $660/year is a compelling case. The functional gap is real but narrow for most typical document work.

### When to recommend Affinity Publisher over InDesign

- Individual designer or small team (no enterprise Adobe subscription already in place)
- Print quality is required but InDesign ecosystem (plugins, scripts) is not needed
- Budget is a genuine constraint
- Documents are not shared with external agencies or printers who require InDesign

---

## Word / LibreOffice Writer — Brief Note

**Detection:** `Creator: Microsoft Word` or `Creator: LibreOffice Writer` in PDF metadata.

These are word processors, not page layout tools. They export PDF but the output is optimised for word-processed documents — continuous text with styles, headers/footers, basic tables. They are not suitable for designed documents with complex layouts.

**When a Word/LibreOffice-produced PDF is the source:**
- Content can be extracted, but layout recreation requires a proper tool
- Style information is partially recoverable via the source `.docx`/`.odt`

**When to use Word/LibreOffice for PDF:**
- Internal documents where layout quality is not a concern
- Documents that must be editable by non-technical users
- Documents that need track-changes and comment workflows

For anything requiring design quality: migrate to an appropriate tool from this list. See `references/tools/docx-generation.md` if the requirement is specifically to produce `.docx` output.

---

## The GUI vs code trade-off, summarised

| Dimension | InDesign / Affinity | Code-based tools |
|---|---|---|
| Design quality ceiling | Highest | High (approaches InDesign for text; lower for complex layout) |
| Automation / CI | Requires InDesign Server (expensive) | Native |
| Version control | Binary files, not diffable | Plain text, fully diffable |
| Authoring accessibility | Designer skill required | Developer skill required |
| Data-driven generation | Possible but complex (data merge) | Natural (template + data) |
| Cost | $55/month (InDesign) or $70 one-time (Affinity) | Free (most tools) |
| Iteration speed | Fast (WYSIWYG) | Medium (code → render cycle) |

**The rule:** if a human designer will own the document and visual quality is primary, InDesign/Affinity. If the document is generated from data or maintained by developers, a code-based tool.
