# Programmatic PDF: React-PDF, pdf-lib, jsPDF, ReportLab, fpdf2 — Deep Dive

This file covers tools that generate PDFs **programmatically** — by drawing PDF primitives (text, shapes, lines, images) directly to a PDF canvas, rather than rendering HTML+CSS. These are fundamentally different from HTML-to-PDF tools.

---

## The fundamental distinction

HTML-to-PDF tools (Paged.js, WeasyPrint, Prince, wkhtmltopdf) take HTML content you authored and render it to PDF. The authoring model is writing HTML+CSS.

Programmatic PDF tools give you an API to place text at coordinates, draw lines, embed images, set fonts — assembling the PDF directly. The authoring model is writing code that describes the document imperatively.

**Choose programmatic when:** the document is data-driven, templated, and generated from structured data at scale (invoices, receipts, certificates, labels, tickets, reports from database queries).

**Choose HTML-to-PDF when:** the document is authored content with visual design, brand elements, and complex layout that would be tedious to describe imperatively.

---

## React-PDF (`@react-pdf/renderer`)

**License:** MIT (free)  
**Language:** JavaScript / TypeScript (React)  
**Install:** `npm install @react-pdf/renderer`

### What it is
React-PDF lets you define PDF documents as React components using its own primitive components (`<Document>`, `<Page>`, `<View>`, `<Text>`, `<Image>`). It has its own layout engine based on Yoga (Facebook's flexbox implementation) — it is NOT rendering HTML. The React component tree is compiled directly to PDF primitives.

### Where it shines
- **Invoice/receipt generation** from React applications — feels natural if you're already in a React codebase
- **Type safety** — TypeScript support for all style props
- **Server-side rendering** — runs in Node.js via `renderToStream()` or `renderToBuffer()`
- **Reusable components** — you can build a component library of PDF building blocks (header, footer, table row) and compose them
- **Consistent output** — not affected by browser rendering differences

### Where it falls short
- **Not HTML.** Cannot render arbitrary HTML strings or markdown into a PDF. Every piece of the document must be a React-PDF component. If you have rich text content (user-generated HTML), you must parse it and convert it to React-PDF components — non-trivial work.
- **Limited text layout.** React-PDF's text rendering is basic compared to CSS engines. Complex typography (hyphenation, justification, optical margin alignment) is not available.
- **No running headers/footers** (natively). You build them as `<View>` elements positioned on each page using `fixed` prop.
- **Flexbox only** — no CSS grid, no absolute positioning in the CSS sense. All layout is Yoga flexbox.
- **Style prop API** — styles are written as JS objects, not CSS. Close to CSS but different syntax, limited properties.
- **Cannot load web fonts from CDN** easily — fonts need to be registered via `Font.register()` with local paths or specific URLs.

### Common pain points
- **Multi-page dynamic content** — React-PDF auto-paginates, but controlling where breaks fall requires `wrap={false}` and `break` props
- **Rich text from CMS** — must write an HTML-to-React-PDF parser, or use `react-pdf-html` (community package with limitations)
- **SVG support** — limited; complex SVGs with filters, clip paths, or masks may not render

### When to choose
- React codebase, invoice/receipt generation, certificates, labels
- Content is fully data-driven (no authored prose)
- Team wants type-safe PDF components that feel like React

---

## pdf-lib

**License:** MIT (free)  
**Language:** JavaScript / TypeScript  
**Install:** `npm install pdf-lib`

### What it is
pdf-lib is a low-level PDF manipulation library. It operates on the PDF format directly — you can create PDFs from scratch, modify existing PDFs, merge documents, fill form fields, and embed fonts. No rendering engine, no layout — you place primitives at exact coordinates.

### Where it shines
- **PDF manipulation** — the best tool for modifying existing PDFs (add pages, fill form fields, add watermarks, merge PDFs, extract pages)
- **Low-level control** — if you know exactly what you want at what coordinates, pdf-lib delivers it
- **No external dependencies** — pure JS, works in browser and Node.js
- **Form filling** — fill AcroForm fields in existing PDFs
- **Metadata manipulation** — set/read PDF metadata fields

### Where it falls short
- **No layout engine.** You specify exact coordinates for every element. Building a "table" means calculating every cell's x/y/width/height manually.
- **No text flow.** Text does not wrap automatically. You calculate line breaks yourself or use helper functions.
- **Not a document authoring tool.** Building a multi-page document with pdf-lib is like building a web page by specifying every pixel's color. Possible, but not the right abstraction.

### When to choose
- PDF post-processing (add watermark to existing PDF, merge PDFs, fill form fields)
- Very simple fixed-layout documents where content is one or two lines at known positions
- You need to manipulate PDF internals directly

### When NOT to choose
- Any document with flowing text or dynamic content length
- Anything where you want HTML/CSS authoring

---

## jsPDF

**License:** MIT (free)  
**Language:** JavaScript  
**Install:** `npm install jspdf`

### What it is
jsPDF is an older JavaScript library for generating PDFs from the browser or Node.js. Like pdf-lib, it operates imperatively — you call `doc.text()`, `doc.line()`, `doc.addImage()` at coordinates.

### Status note
jsPDF is more mature than pdf-lib but also more limited in its PDF capabilities. For new projects, **prefer pdf-lib** — it has better TypeScript support, better font handling, and a cleaner API. jsPDF is often found in legacy codebases.

### jsPDF + html2canvas pattern
The common pattern `html2canvas(element) → jsPDF.addImage()` captures a screenshot of HTML and embeds it as an image in the PDF. This is **not a real PDF** — the "text" is a rasterized image, not selectable or searchable. Accessibility is zero. File size is large. Text appears blurry at sizes different from the capture resolution. Avoid this pattern for any document where quality matters.

---

## ReportLab (Python)

**License:** Open-source (BSD) and commercial (Pro version)  
**Language:** Python  
**Install:** `pip install reportlab`

### What it is
ReportLab is the dominant programmatic PDF library in the Python ecosystem. It has two APIs:
- **`canvas`** — low-level, draw primitives at coordinates (like pdf-lib)
- **`platypus`** — high-level, document layout with `Flowables` (paragraphs, tables, images, spacers) that automatically flow across pages

The Platypus API is where ReportLab's power is — it handles text flow, automatic page breaks, multi-page tables, and headers/footers in a way that makes it genuinely usable for document generation.

### Where it shines
- **Enterprise Python PDF generation** — the de-facto standard; 20+ years of production use
- **Multi-page table handling** — Platypus `Table` with `repeatRows` produces reliable multi-page tables
- **Performance at scale** — generating 10,000 invoices overnight in a batch job; ReportLab is fast
- **Custom page geometry** — precise control over every element's position and appearance
- **PDF forms** — can generate fillable PDF forms programmatically
- **PDF/A output** — Pro version supports PDF/A for archival compliance

### Where it falls short
- **No HTML/CSS** — content must be defined via Platypus Flowables or canvas API. Rich text requires `Paragraph` with a limited inline markup language (not HTML)
- **Typography** — not as good as LaTeX or Prince for long-form prose; adequate for business documents
- **Learning curve** — Platypus has its own abstraction model that takes time to learn
- **Styling API** — styles are defined via `ParagraphStyle` objects, not CSS; the API feels dated by modern standards

### The hidden advantage
ReportLab excels at exactly the kind of documents that HTML-to-PDF tools handle worst: hundreds of nearly-identical pages generated from database records with complex table structures. A financial report with 500 rows per page, generated nightly — ReportLab handles this reliably and fast.

### When to choose ReportLab
- Python ecosystem
- Data-heavy documents generated from structured data at scale
- Complex tabular output (financial reports, analytics dashboards, inventory reports)
- Performance matters (batch generation of hundreds/thousands of PDFs)
- HTML authoring is not needed

---

## fpdf2 (Python)

**License:** LGPL (free)  
**Language:** Python  
**Install:** `pip install fpdf2`

### What it is
fpdf2 is a modern Python PDF library, lighter than ReportLab, with a cleaner API. It handles flowing text, multi-cell layouts, images, and basic table structures. It does not have ReportLab's Platypus complexity — simpler API but fewer capabilities.

### When to choose over ReportLab
- Simpler documents (invoices, labels, certificates)
- You want a lighter dependency
- Modern API is preferred over ReportLab's dated interface
- You don't need advanced Platypus features (complex multi-page tables, conditional rendering)

---

## The rasterization anti-pattern (html2canvas, Puppeteer screenshot)

**Warning: this is common and almost always wrong for document production.**

The pattern:
1. Render HTML in a browser or with Puppeteer
2. Take a screenshot (PNG/JPEG)
3. Embed the screenshot in a PDF via jsPDF or pdf-lib

**Problems:**
- **Text is not text.** The PDF contains images, not text. Users cannot select, copy, or search text. Screen readers cannot read it. It fails any accessibility check.
- **Resolution-dependent quality.** The rasterized image must be captured at very high DPI (300+) to look sharp in print. Screenshots captured at screen resolution (72-96 DPI) look terrible in print.
- **File size.** A rasterized PDF is 5-20× larger than a text-based PDF.
- **No font embedding.** The PDF doesn't know what fonts were used — it's just pixels.

**When it's acceptable (narrow):**
- The "document" is actually a signature capture or drawing
- You need an exact pixel-perfect screenshot of a UI for documentation purposes
- You're deliberately creating a non-selectable security copy

**In all other cases: use a proper HTML-to-PDF tool or programmatic PDF library.**

---

## Decision tree: programmatic vs HTML-to-PDF

```
Is the document primarily data-driven (generated from structured data)?
  ↓ Yes → Is the team in Python or JS/React?
    ↓ Python → ReportLab (complex) or fpdf2 (simple)
    ↓ JS/React → React-PDF (React team) or pdf-lib for manipulation
  ↓ No → Is the document authored content with visual design?
    ↓ Yes → Use HTML-to-PDF (Paged.js, WeasyPrint, Prince)
    ↓ Mixed (designed template + data) → HTML+CSS template + Paged.js/WeasyPrint
                                          OR React-PDF with component library
```
