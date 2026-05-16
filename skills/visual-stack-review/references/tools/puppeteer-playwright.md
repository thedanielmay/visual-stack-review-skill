# Puppeteer / Playwright (Headless Chrome) — Deep Dive

**License:** Apache 2.0 / MIT (open source, free)  
**Language:** Node.js (both)  
**Install:** `npm install puppeteer` or `npm install playwright`

---

## What Puppeteer/Playwright actually is in this context

Puppeteer and Playwright are browser automation libraries. In the context of PDF generation, they control a headless Chrome or Chromium browser, load your HTML, and call the browser's built-in `Page.printToPDF()` function. The result is exactly what Chrome would print if you hit Cmd+P.

This is fundamentally different from Paged.js: Paged.js IS a polyfill that runs inside Chrome to implement CSS Paged Media. Puppeteer/Playwright WITHOUT Paged.js use Chrome's native print CSS (`@media print`) — which does not implement CSS Paged Media features like running headers/footers, `string()`, `element()`, or `@page` margin boxes.

Key distinction:
- **Paged.js + pagedjs-cli** = headless Chrome + CSS Paged Media polyfill → full pagination control
- **Puppeteer/Playwright alone** = headless Chrome + native print CSS only → limited pagination, no running headers

---

## Quality ceiling for designed documents

**Honest ceiling:** Using Puppeteer/Playwright alone (without Paged.js), the ceiling is moderate. You get everything `@media print` CSS can express — which is significant but does not include dynamic headers/footers with page numbers, named pages, or CSS counter-based TOC generation.

With Paged.js loaded inside the Puppeteer/Playwright page, the ceiling matches Paged.js's ceiling (see pagedjs.md).

---

## What Puppeteer/Playwright is genuinely good at

**Full modern CSS support.** You're rendering in Chrome. Every CSS feature Chrome supports is available — grid subgrid, container queries, `@layer`, modern color spaces, variable fonts, all of it. No CSS compatibility concerns.

**JavaScript-driven content.** Charts rendered by Chart.js, D3, Recharts, ECharts — all render correctly because JavaScript runs. React, Vue, Svelte components render. Dynamic tables, client-rendered content — all works.

**Exact browser fidelity.** If your document already looks correct in Chrome, Puppeteer/Playwright will produce a PDF that matches. There is no translation layer that might interpret your CSS differently.

**Screenshot capability.** Alongside PDFs, Puppeteer/Playwright can produce PNG screenshots of any element or full page. Useful for hybrid workflows where you need both image exports and PDFs.

**Full programmatic control.** You can interact with the page before printing — scroll to trigger lazy-loaded content, wait for network requests, inject data, manipulate the DOM. This is not possible with WeasyPrint or Prince.

**Excellent for web-first → PDF workflows.** If the primary output is a web page and PDF is secondary, Puppeteer/Playwright is the natural choice — the web page IS the source, and PDF is rendered from it.

---

## Where Puppeteer/Playwright loses to alternatives

**Running headers/footers are second-class.** Chrome's native `page.pdf()` accepts `headerTemplate` and `footerTemplate` parameters, but these are limited plain-HTML fragments that do not have access to your page's CSS or JavaScript. You cannot easily style them with your brand CSS, use CSS custom properties in them, or reference content from the page. This is the single biggest practical limitation for document production.

**Workaround cost:** Many teams solve this by using absolute-positioned fixed headers/footers in the page body and relying on `@media print` CSS to control page breaks. This works but requires significant layout discipline — elements positioned outside the normal flow on every page.

**CSS Paged Media is not supported natively.** Without Paged.js, `@page { @top-left { ... } }`, `string()`, `running()`, and CSS named counters for page numbers do not work. You get `@page { margin: ... }` but nothing inside the margin boxes.

**Pagination is uncontrolled.** Chrome decides where to break pages. `break-before: page` and `break-after: page` work, but `widows`, `orphans`, and fine-grained pagination control are unreliable without Paged.js.

**Build overhead.** Starting a headless Chrome browser takes time (1-3 seconds cold start). For server-side generation at scale, browser pool management adds architectural complexity. WeasyPrint and Prince are single-process tools with no browser lifecycle to manage.

**Chrome version drift.** Your PDF output may differ slightly between Chrome versions. This is rarely a practical problem but matters for regression testing.

**File sizes.** Chrome-generated PDFs are often larger than WeasyPrint or Prince equivalents because Chrome embeds full font sets and doesn't always subset aggressively.

---

## Puppeteer vs Playwright — which to use?

**Use Playwright when:**
- You need cross-browser support (Playwright can drive Firefox and WebKit too)
- You're already using Playwright for end-to-end testing
- You want auto-wait behaviour (Playwright's locator model handles dynamic content more gracefully)
- You want the better maintained library (Playwright is more actively developed as of 2025)

**Use Puppeteer when:**
- You have existing Puppeteer code
- You need the maximum PDF control options (Puppeteer's PDF options are slightly more extensive)
- Your team knows Puppeteer and the codebase is already built around it

For new projects: prefer **Playwright**. It is the more actively maintained library and the better default choice.

---

## The Paged.js + Playwright combination

This is the most common production setup for complex HTML documents:

```js
const browser = await chromium.launch();
const page = await browser.newPage();
await page.goto('http://localhost:3000/document');
await page.waitForFunction(() => window.PagedPolyfill?.done);
await page.pdf({ path: 'output.pdf', printBackground: true });
await browser.close();
```

The `waitForFunction` call waits for Paged.js to finish paginating before printing. Without this, the PDF captures the un-paginated version.

**When this works well:** When you need CSS Paged Media features (running headers, named pages, CSS counters) AND modern CSS features that Paged.js benefits from by running in Chrome.

**When this fails:** The Paged.js + Chrome combination has known bugs (see visual-review/references/pagedjs.md for the full list). The bugs are manageable but require CSS workarounds.

---

## Migration cost: from Paged.js+CLI to Playwright+Paged.js

**Effort: Low** (same underlying stack, different entry point)

You're already using Paged.js. The change is replacing the `pagedjs-cli` process with Playwright controlling a browser that loads Paged.js. This gives you more control over the render lifecycle but is otherwise the same output.

**What changes:** Build pipeline only. Content and CSS stay the same.

---

## Migration cost: from Puppeteer/Playwright (no Paged.js) to WeasyPrint

**Effort: Medium-High**

**What changes:**
- Remove JavaScript-rendered content or replace with static SVG/server-side equivalents
- Rewrite running headers/footers from CSS positioned elements to CSS Paged Media `@page` syntax
- Remove modern CSS features not supported by WeasyPrint
- Change build pipeline from Node to Python

**What you keep:**
- HTML structure
- Most typography CSS

---

## When to choose Puppeteer/Playwright (without Paged.js)

- Primary output is web (screen HTML) and PDF is a secondary export
- Document uses JavaScript-rendered charts/content that cannot be made static
- Running headers/footers are not required OR can be handled with `headerTemplate`/`footerTemplate`
- Your team already uses Playwright for testing — minimal new tooling

## When to choose Puppeteer/Playwright WITH Paged.js

- You need CSS Paged Media features (running content, named pages, CSS counters)
- AND you need JavaScript-rendered content
- AND you can tolerate the known Paged.js bugs (with CSS workarounds)
