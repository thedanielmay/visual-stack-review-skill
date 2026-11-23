# Changelog

## 1.2.0 - 2026-11-23

- Settled one wkhtmltopdf rule. The skill recommends moving away. Staying requires simple templates that will not change, no untrusted HTML or URLs, and a report that states the archived tool gets no security fixes. The description, the working principles and the wkhtmltopdf deep dive now say the same thing, and the README, the matrix note and the detection note agree.
- Added two intake questions. Ask for the printer's specification (bleed, trim size, colour profile and PDF/X level) when output goes to print. Ask for the accessibility requirement (tagged PDF or PDF/UA) when people need accessible documents. A file is press-ready or accessible only when the stated standard is checked and met. Unchecked items are reported as "not checked".
- Moved the detection catalogue into `references/detection.md` and left a one-line load trigger in `SKILL.md`. Merged the two Chrome detection sections into one, keeping the Chrome 131 margin boxes, named pages and Skia/PDF producer facts. Kept the recommendation format, the migration-cost axes, the wkhtmltopdf rule and the press-ready checks in `SKILL.md`.

## 1.1.1 - 2026-11-23

- Corrected the Chrome 131 facts. Chrome supports named pages and `@page` margin boxes with page counters. It still lacks running elements with `position: running()` and `string-set` / `string()`, and it lacks footnotes.
- Corrected Chrome PDF detection. PDFs made by Chrome and Puppeteer usually carry `Producer: Skia/PDF mNNN` and `Creator: Chromium` or `HeadlessChrome`. Detection no longer keys on `Producer: Chromium`.
- Corrected the Paged.js signals. Use the package dependency, the polyfill script tag or the PDF Producer. The `.pagedjs_page` class exists only in the rendered DOM. Shared CSS features such as `@page`, `running()` and `string()` identify a paged-media engine, not a specific engine.
- Made the Prince price consistent across the skill. Prince has a one-time server licence of about USD 495, plus a personal or non-commercial option. Check princexml.com for current prices.

## 1.1.0 - 2026-11-23

- Moved the tool comparison matrix and its reading guide into `references/comparison-matrix.md`.
- Moved the matrix review procedure into `references/matrix-review.md`.
- Added pointer lines in `SKILL.md` for both references, cutting the body under 500 lines.

## 1.0.4 - 2026-11-23

- Adds the marketplace icon approved by Daniel (thedanielmay/agent-tools#2).

## 1.0.3 - 2026-11-23

- Marketplace file gains a description, so the full check passes again (thedanielmay/agent-tools#10).

## 1.0.2 - 2026-11-22

- Added a marketplace manifest, an MIT licence and this changelog.
- Added `license` to the SKILL.md frontmatter and moved the version to 1.0.2.
- Removed the author email from `plugin.json` and added the author URL, homepage, repository and licence.
- Removed the version number from the README prose and added an Install section.

Earlier versions: see git tags.
