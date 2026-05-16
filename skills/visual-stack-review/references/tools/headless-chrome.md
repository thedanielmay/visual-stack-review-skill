# Headless Chrome as a PDF Renderer — Deep Dive

This file covers Chrome/Chromium as a PDF rendering engine specifically — what it can and can't do natively, how it differs from the tools that sit on top of it (Paged.js, Puppeteer, Playwright), and when to use Chrome directly vs. via an abstraction layer.

---

## Understanding the stack layers

Chrome-based PDF generation has layers that are often confused:

```
Your HTML + CSS
       ↓
[Paged.js polyfill] ← optional, adds CSS Paged Media support
       ↓
[Puppeteer / Playwright / pagedjs-cli] ← automation library
       ↓
[Headless Chrome / Chromium] ← the actual rendering engine
       ↓
PDF via Page.printToPDF (Chrome DevTools Protocol)
```

Each layer is optional. You can call Chrome directly via CDP without Puppeteer. You can use Puppeteer without Paged.js. Understanding which layer does what is the key to debugging and evaluating this stack.

---

## What Chrome's native `Page.printToPDF` actually supports

When you call `page.pdf()` in Puppeteer/Playwright (which calls Chrome's `Page.printToPDF` under the hood), you get:

**Supported natively:**
- `@media print` CSS rules
- `@page { margin: ... }` — page margin control
- `break-before`, `break-after`, `break-inside` — page break hints
- `page-break-before`, `page-break-after` — legacy equivalents
- Background printing (`printBackground: true` option)
- Custom page sizes and orientations
- Scale factor
- Header and footer templates (`headerTemplate`, `footerTemplate`) — but see limitations below

**NOT supported natively (without Paged.js):**
- `@page` margin box content (`@top-left`, `@bottom-center`, etc.)
- `string()` and `element()` — CSS running content for dynamic headers
- `running()` — flowing content into margin boxes
- Named pages (`page: named-page`) with separate `@page named-page` rules
- CSS `counter()` in page margin boxes
- `orphans` and `widows` CSS properties — Chrome ignores these
- `float: footnote` — CSS footnote support
- `marks: crop cross` — crop marks

This is the core gap that Paged.js exists to fill.

---

## Chrome's `headerTemplate` / `footerTemplate` — detailed capabilities and limitations

The `headerTemplate` and `footerTemplate` parameters in Puppeteer/Playwright's `page.pdf()` accept HTML strings for page headers and footers. These are more limited than they appear.

**What works:**
- Basic HTML structure (`<div>`, `<span>`, `<p>`)
- Inline styles (`style="..."`)
- Special class variables:
  - `<span class="pageNumber">` — injects current page number
  - `<span class="totalPages">` — injects total page count
  - `<span class="title">` — injects document `<title>`
  - `<span class="url">` — injects document URL
  - `<span class="date">` — injects current date

**What does NOT work:**
- External CSS (your page's stylesheet is NOT loaded in the header/footer context)
- JavaScript execution within the template
- CSS custom properties from your page
- Font references (fonts from your page's `@font-face` are not available)
- Images via relative paths (must use absolute paths or data URIs)
- Access to page content (no "running headers" — the header is static, not content-aware)

**The practical consequence:** If you want a styled header with your brand font and colors, you must either:
1. Inline all styles in the template string
2. Base64-encode fonts and include them as data URIs in the template
3. Use Paged.js for proper CSS Paged Media support instead

**Example of a working styled header:**
```js
headerTemplate: `
  <div style="font-family: Arial, sans-serif; font-size: 10px; 
               width: 100%; padding: 5px 20px; 
               border-bottom: 1px solid #ccc; color: #666;">
    <span style="float: left;">Document Title</span>
    <span style="float: right;">Page <span class="pageNumber"></span> of <span class="totalPages"></span></span>
  </div>
`
```

---

## Chrome's rendering characteristics for PDF

**What Chrome does well for PDF:**
- Full CSS Grid Level 1 and 2 (subgrid)
- CSS Container Queries
- CSS Custom Properties (variables) — including in `calc()` and `@keyframes`
- All modern color spaces (`oklch`, `lab`, `p3`)
- Variable fonts with all axes
- SVG rendering — including filters, masks, clip-paths, animations (animations freeze in PDF)
- WebP and AVIF images
- CSS Shapes (`shape-outside`)
- Blend modes (`mix-blend-mode`, `background-blend-mode`)

**Where Chrome PDF rendering differs from screen rendering:**
- `position: fixed` elements appear on every page (can be both useful and surprising)
- `vh`/`vw` units reference the viewport size, not the page size
- `window.innerWidth`/`window.innerHeight` reflect the headless viewport, not the printed page dimensions
- CSS animations and transitions are frozen at their initial state in PDF
- `@keyframes` do not run during PDF export
- Video and audio elements are replaced with a placeholder or their poster image
- `window.print()` triggering from JavaScript does not work in headless mode — use Puppeteer/Playwright's `page.pdf()` directly
- `pointer-events`, `:hover`, `:focus` pseudo-classes have no effect in PDF

**Chrome's print background handling:**
- Background colors and images only print if `printBackground: true` is set
- `@media print { -webkit-print-color-adjust: exact; }` or `print-color-adjust: exact` in CSS is required for background fills to render correctly — even with `printBackground: true`

---

## Running Chrome directly via CDP (no Puppeteer/Playwright)

For high-performance server-side PDF generation, you can drive Chrome directly via the Chrome DevTools Protocol without the Puppeteer/Playwright abstraction layer. This reduces Node.js process overhead.

**Using `chrome-remote-interface`:**
```js
import CDP from 'chrome-remote-interface';
// Connect to already-running Chrome: chrome --headless --remote-debugging-port=9222
const client = await CDP();
const { Page } = client;
await Page.enable();
await Page.navigate({ url: 'http://localhost:3000/document' });
await Page.loadEventFired();
const { data } = await Page.printToPDF({
  printBackground: true,
  paperWidth: 8.27, // A4 in inches
  paperHeight: 11.69,
  marginTop: 0,
  marginBottom: 0,
  marginLeft: 0,
  marginRight: 0,
});
fs.writeFileSync('output.pdf', Buffer.from(data, 'base64'));
await client.close();
```

**The advantage:** Keep Chrome running as a persistent process. Subsequent requests connect to the existing browser instance — no cold-start overhead (1-3 seconds saved per document).

**When this matters:** High-volume PDF generation (>100 documents/hour) where cold-start time is a bottleneck.

---

## Chrome vs. Chromium — the practical difference

**Chrome** = Google's distribution, includes proprietary codecs (AAC, H.264), Google update mechanism, Google services integration.

**Chromium** = Open-source base, no proprietary codecs, no auto-updates.

**For PDF generation:**
- Puppeteer downloads its own Chromium version by default — consistent, reproducible
- Playwright downloads browser binaries that match its tested version
- PDF output between Chrome and Chromium is functionally identical for document production
- Use the browser version bundled with your automation library, not the system browser

---

## Chrome version and PDF output stability

Chrome's PDF rendering does change between major versions. Known areas of version-sensitive behavior:
- Font rasterization quality changes between Chrome 100-120 series
- Background-color rendering in `@page` has historically been inconsistent
- Some flexbox edge cases changed between Chrome 110-120

**Mitigation:** Pin your Puppeteer/Playwright version in `package-lock.json`, which pins the bundled Chromium version. Do not use the system Chrome for automated PDF generation unless you also control Chrome updates.

---

## Performance characteristics at scale

| Scenario | Cold start | Per-document | Memory |
|---|---|---|---|
| New browser per PDF | 1.5-3s | + 0.5-2s render | 200-300MB peak |
| Persistent browser, new page per PDF | None | 0.5-2s | 150-250MB sustained |
| Persistent browser + page pool | None | 0.3-1s | Scales with pool size |

**For production at scale:** run a pool of persistent headless Chrome instances (using libraries like `generic-pool` with Playwright contexts, or a dedicated service like `browserless.io`).

---

## Chrome-specific rendering bugs affecting PDF production

**Background color on `<html>` or `<body>` does not fill the page margin area.** The background only fills the content area. For full-page background colors in PDF: use `@page { background-color: ... }` in `@media print` — but Chrome's support for this is inconsistent. Most reliable: set a background on a full-page `<div>` with `position: fixed; top: 0; left: 0; width: 100%; height: 100%; z-index: -1`. This `position: fixed` element will repeat on every page. (This is the same root cause as Paged.js PJ-1, because Paged.js runs inside Chrome.)

**`position: fixed` header/footer without Paged.js:** A common pattern for adding headers/footers to Chrome-printed PDFs is to use `position: fixed` elements in the HTML body. These render on every page. Limitations: they consume page space (content may be hidden behind them), they can't adapt to page-specific content, and they don't have access to page numbers (without JavaScript tricks). Use the `headerTemplate`/`footerTemplate` approach for simple page numbers, or Paged.js for complex running content.

**Subpixel rendering differences:** Chrome's PDF output uses a different text rendering path than its screen rendering. Fonts that appear perfectly kerned on screen may have very slightly different spacing in PDF. This is usually not visible to the naked eye but can affect tight layouts.

---

## When to use headless Chrome directly (no Paged.js)

- Screen-first web pages that also need PDF export (the HTML was designed for screen)
- Documents without running headers/footers (or with simple ones via `headerTemplate`)
- Documents using modern CSS features that Paged.js or WeasyPrint don't support
- High-performance server-side generation with a persistent browser pool
- Documents where JavaScript-rendered content (charts, dynamic data) is required

## When to add Paged.js on top of Chrome

- Running headers with dynamic content (section titles, document title)
- CSS counter-based page numbers in styled headers/footers
- Named pages (different `@page` rules for different page types)
- Precise widow/orphan control for long-form text
- `float: footnote` CSS footnotes
- Any CSS Paged Media feature

## When to choose a different renderer entirely

- Tables are complex and multi-page → WeasyPrint or Prince
- Print-production PDF/X output → Prince
- No JavaScript content, Python ecosystem → WeasyPrint
- Long-form academic/technical text → LaTeX or Typst
- Performance at scale with simple documents → Typst or WeasyPrint
