# Matrix review procedure

This reference holds the full matrix review procedure. Read it when the user invokes `/visual-stack-review review-matrix` or asks to check the matrix for staleness.

**Invocation:** User says `/visual-stack-review review-matrix` or "review the matrix for staleness."

**When to run:** Every 3 months from the Last reviewed date, or immediately after a major version release of any tool in the matrix.

**Do not skip this procedure and do an ad-hoc check.** The procedure exists so reviews are consistent, traceable, and cover the same questions every time. An improvised check misses rows and produces no audit trail.

### Step 1 — Check release activity for each tool

For each tool in the matrix, fetch the latest release information:

```
Paged.js:        https://github.com/pagedjs/pagedjs/releases
WeasyPrint:      https://github.com/Kozea/WeasyPrint/releases
Prince:          https://www.princexml.com/latest/ (changelog)
Typst:           https://github.com/typst/typst/releases
Playwright:      https://github.com/microsoft/playwright/releases
Puppeteer:       https://github.com/puppeteer/puppeteer/releases
LaTeX (TeX Live):https://tug.org/texlive/acquire-netinstall.html (annual release)
React-PDF:       https://github.com/diegomura/react-pdf/releases
ReportLab:       https://hg.reportlab.com/hg-public/reportlab/ or PyPI changelog
wkhtmltopdf:     https://github.com/wkhtmltopdf/wkhtmltopdf (expect no activity — confirm)
```

For each tool: note the current version, the version at last review, and any releases since then. If a tool shows no releases in 12+ months, flag as potentially unmaintained.

### Step 2 — Run per-row capability questions

For each row in the matrix, ask the specific question that would cause the rating to change. Do not just re-read the matrix — actively query against current state.

| Row | Question to ask |
|---|---|
| Table handling | Has WeasyPrint or Paged.js shipped fixes for multi-page table header repetition or `break-inside: avoid` since last review? |
| Running headers/footers | Has WeasyPrint shipped further improvements to named-string/running-element support? Is v60 still the breakpoint? |
| SVG / figure support | Has WeasyPrint's SVG engine shipped fixes for `<use>` with nested transforms? Has Typst's SVG import improved? |
| PDF/X-4 spec conformance | Has any tool added native PDF/X-4 output that wasn't there before? |
| JavaScript execution | No change expected — this is architectural. Skip unless a tool has announced JS support. |
| Widow/orphan control | Has Paged.js shipped fixes for `orphans`/`widows` CSS property handling? |
| Footnotes | Has WeasyPrint improved `float: footnote` support? |
| Typst math | Any regressions or significant new capabilities? |
| wkhtmltopdf maintenance | Confirm still archived. If a fork has emerged with active maintenance, note it. |
| Maintenance status | Check for any tool that has gone quiet, changed ownership, or announced deprecation. |

### Step 3 — Check the `†` (unverified) rows

Rows marked `†` were not independently verified at last review. For each, attempt to find a primary source (release note, documentation, independent benchmark) that either confirms the rating or corrects it. Remove `†` if confirmed; update the rating and note the source if changed.

### Step 4 — Produce a structured diff

Output the review as a structured diff. Format:

```
## Matrix review — [date]

**Tools checked:** [list]
**Period covered:** [last review date] → [today]
**Next review due:** [today + 3 months]

### Changed ratings
| Tool | Row | Previous | New | Reason | Source |
|---|---|---|---|---|---|
| WeasyPrint | SVG / figure support | ★★★☆☆ | ★★★★☆ | v62 fixed nested-transform <use> elements | [link to release note] |

### Unchanged ratings (confirmed)
[List rows where active checking confirmed no change — not just "assumed unchanged"]

### Unverified rows (still †)
[List any † rows where a primary source could not be found — explain why]

### New tools to consider adding
[Any tool that has emerged since last review that belongs in the matrix]

### Maintenance alerts
[Any tool showing reduced activity, ownership changes, or deprecation signals]
```

### Step 5 — Apply changes to the matrix

Edit `references/comparison-matrix.md`:
1. Update the changed rating cells
2. Remove `†` from newly-confirmed rows, or update ratings for corrected rows
3. Update the `Last reviewed` and `Next review due` dates at the top of that file
4. Update the WeasyPrint column header if the minimum version threshold has changed (e.g. `WeasyPrint ≥v60` → `WeasyPrint ≥v62`)
5. Update the relevant deep-dive reference file (`references/tools/[tool].md`) if the version note at the top needs updating

### What this procedure does NOT do

- It does not re-run benchmarks or independent testing — ratings are based on release notes, documented fixes, and known issues
- It does not evaluate entirely new tools not already in the matrix (add a note for the user to consider if warranted)
- It does not change migration cost estimates in the deep-dive files — those require separate review against current field experience
