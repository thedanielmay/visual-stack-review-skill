# wkhtmltopdf — Deep Dive

**Version at time of writing:** 0.12.6 (released 2020 — effectively unmaintained)  
**License:** LGPL (open source, free)  
**Language:** Any (CLI tool)  
**Install:** Binary download from wkhtmltopdf.org; also `brew install wkhtmltopdf`  
**Status:** ⚠️ EFFECTIVELY UNMAINTAINED — last release 2020, project archived

---

## The critical context: wkhtmltopdf is dead infrastructure

This must be stated before anything else: **wkhtmltopdf uses Qt WebKit, a browser engine frozen at approximately 2016 Chrome capabilities.** It does not receive security updates. The GitHub repository is archived. The project is not maintained.

If you are currently running wkhtmltopdf in production, you are running on a browser engine that:
- Does not support CSS Grid (Level 1 only, Level 2 not supported)
- Does not support CSS Custom Properties (variables) in all contexts
- Does not support Flexbox reliably in complex nested layouts
- Does not support modern font formats (WOFF2 has issues)
- Has known JavaScript execution bugs
- Has CSS rendering bugs that will never be fixed

**The only question for wkhtmltopdf users is not "should I migrate?" but "which tool should I migrate to?"**

---

## Why wkhtmltopdf exists everywhere despite being dead

It was the right tool for its time (2008-2018). It was:
- Easy to install (single binary, no Node/Python required)
- Fast (no browser startup overhead — used a patched Qt process)
- Good enough for simple HTML reports and invoices
- Supported out of the box in Rails, Django, Laravel via gems/packages
- The default recommendation in countless Stack Overflow answers from 2010-2018

This is why you still find it in enterprise codebases built before 2020. It worked then; the question is whether it works now for what you need to build.

---

## What wkhtmltopdf is good at (was good at)

**Simple HTML reports.** Tables, text, basic styling — wkhtmltopdf handles these correctly if you stay within its CSS capabilities (pre-2016 CSS).

**Speed.** No browser startup overhead. A simple document renders in ~0.5-1 seconds.

**Zero-dependency binary.** Download one binary, it works. No Node, no Python, no browser management.

**Header and footer support.** wkhtmltopdf has `--header-html` and `--footer-html` flags that accept HTML files for page headers and footers. The `[page]` and `[topage]` variables inject page numbers. This was wkhtmltopdf's killer feature — browser-based tools didn't support this until Paged.js arrived.

---

## Where wkhtmltopdf fails today

**Modern CSS.** Any CSS written in the last 5-8 years will have issues. CSS Grid (Level 2), container queries, `@layer`, CSS custom properties in `calc()`, modern color functions — all unsupported or broken.

**JavaScript.** wkhtmltopdf executes JavaScript but the environment is WebKit 2016. Many modern JS APIs are missing. D3 v7, React 18, Chart.js v4 will not work correctly or at all.

**Unicode and font support.** WOFF2 fonts have inconsistent support. Right-to-left text rendering is unreliable.

**HTTPS and modern TLS.** wkhtmltopdf uses the system's Qt SSL, which may not support modern TLS versions. Loading resources from HTTPS URLs can fail silently.

**No security updates.** Any HTML you feed to wkhtmltopdf is processed by an unpatched browser engine with known CVEs. If user-supplied content enters the pipeline, this is a security risk.

**Rendering bugs that will never be fixed.** There are open GitHub issues with confirmed reproducible rendering bugs that were never fixed before the project was archived.

---

## Migration path recommendations

**Migrate to Playwright + Chrome (headless)** if:
- You need JavaScript execution (charts, dynamic content)
- You need modern CSS
- You're already in a Node ecosystem
- Your documents don't need complex running headers/footers with CSS Paged Media
- Effort: Low-Medium (mostly build pipeline change, content transfers)

**Migrate to WeasyPrint** if:
- You're in a Python ecosystem (Django, Flask, FastAPI)
- Your documents are tables/text without heavy JS content
- You want a tool with active maintenance and a stable API
- Effort: Low-Medium (Python, CSS audit required)

**Migrate to Paged.js** if:
- Your documents are design-heavy and need modern CSS features
- You need reliable CSS Paged Media (running headers, page counters)
- You're in the Node ecosystem
- Effort: Medium (HTML stays, CSS needs audit, build pipeline changes)

**Do NOT migrate to Prince** just for the header/footer feature (that was wkhtmltopdf's main advantage). Prince has far better capabilities but the cost may not be justified if WeasyPrint covers your needs.

---

## Migration cost: from wkhtmltopdf to Playwright/Chrome

**Effort: Low**

**What changes:**
- Replace `wkhtmltopdf input.html output.pdf` with Playwright script
- Running headers/footers: replace `--header-html` with `headerTemplate` / `footerTemplate` in Playwright's `page.pdf()` (note: Playwright's header/footer templates are more limited than wkhtmltopdf's — they don't execute JavaScript and don't have access to page CSS)
- Review CSS for wkhtmltopdf workarounds that are no longer needed in Chrome

**What you keep:**
- All HTML content
- Most CSS (Chrome supports far more than wkhtmltopdf's Qt WebKit)
- General workflow (HTML in, PDF out)

**Known gotcha:** wkhtmltopdf's `--header-html` and `--footer-html` accept full HTML pages with access to your CSS. Playwright's `headerTemplate`/`footerTemplate` are sandboxed fragments without your CSS. If you rely heavily on styled headers/footers, you'll either need to style them inline or switch to Paged.js for proper CSS Paged Media support.

---

## Migration cost: from wkhtmltopdf to WeasyPrint

**Effort: Low-Medium**

**What changes:**
- Replace CLI call or Ruby/Python gem with `weasyprint input.html output.pdf`
- Audit CSS for wkhtmltopdf-specific workarounds (many were fixing bugs that WeasyPrint doesn't have)
- Replace `--header-html` / `--footer-html` with CSS `@page` margin box rules (WeasyPrint supports these natively)
- Vendor fonts locally (WeasyPrint requires local font access)

**What you keep:**
- All HTML content
- Most CSS (WeasyPrint CSS support is broader than wkhtmltopdf's)
- Running headers/footers (via CSS Paged Media — actually better than wkhtmltopdf's approach)

---

## When you might stay on wkhtmltopdf (narrow circumstances)

Only if ALL of these are true:
- Your HTML is static, simple, and uses only pre-2016 CSS
- You have no intention of updating the HTML templates
- The documents are internal-only (no security exposure)
- Migration effort genuinely cannot be justified (e.g. legacy system being decommissioned in 6 months)

In every other circumstance, migration is the right call. wkhtmltopdf is not a stable foundation for new work.
