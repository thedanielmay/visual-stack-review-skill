# Tool comparison matrix

This reference holds the capability matrix, its rating legend, the press-ready note, and the matrix reading guide. Read it when the user needs a common-axis tool comparison, an appendix, or the current review dates.

Use this when the user needs to compare tools across a common set of axes. Include as an appendix or on request — don't lead with it, as it can obscure the recommendation.

**⚠️ Matrix review schedule: every 3 months.** Invoke with: `/visual-stack-review review-matrix` — the skill runs the defined procedure below and outputs a structured diff. Do not skip; stale ratings cause real migration decisions to go wrong.

**Last reviewed:** 2026-05-16 · **Next review due:** 2026-08-16

**Key:** ★★★★★ = best in class · N/A = not applicable to this tool's model · ⚠️ = known issues · † = unverified, treat as provisional

| Capability | Paged.js | WeasyPrint ≥v60 | Prince | Typst | Headless Chrome (bare) | LaTeX | React-PDF | ReportLab | InDesign |
|---|---|---|---|---|---|---|---|---|---|
| CSS layout fidelity | ★★★★★ | ★★★☆☆ | ★★★★☆ | N/A | ★★★★★ | N/A | ★★☆☆☆ | N/A | N/A |
| Table handling | ★★★☆☆ ⚠️ | ★★★★☆ | ★★★★★ | ★★★★☆ | ★★★☆☆ ⚠️ | ★★★★★ | ★★★☆☆ | ★★★★☆ | ★★★★★ |
| Running headers/footers | ★★★★☆ | ★★★★☆ | ★★★★★ | ★★★★☆ | ★★☆☆☆ | ★★★★★ | ★★☆☆☆ | ★★★☆☆ | ★★★★★ |
| Widow/orphan control | ★★★☆☆ | ★★★☆☆ | ★★★★★ | ★★★★☆ | ★☆☆☆☆ | ★★★★★ | ★☆☆☆☆ | ★★☆☆☆ | ★★★★★ |
| Footnotes | ★★☆☆☆ | ★★☆☆☆ | ★★★★★ | ★★★★☆ | ★☆☆☆☆ | ★★★★★ | ★☆☆☆☆ | ★★☆☆☆ | ★★★★★ |
| SVG / figure support | ★★★★★ | ★★★★☆ | ★★★★☆ | ★★★★☆ | ★★★★★ | ★★★☆☆ | ★★★☆☆ | ★★☆☆☆ | ★★★★★ |
| JavaScript execution | ★★★★★ | ✗ | ✗ | ✗ | ★★★★★ | ✗ | ✗ | ✗ | ✗ |
| Data binding / templating | ★★★☆☆ | ★★★★☆ | ★★★★☆ | ★★★★☆ | ★★★★★ | ★★★☆☆ | ★★★★★ | ★★★★★ | ★★☆☆☆ |
| Render speed (100 pages) | ★★★☆☆ | ★★★★☆ | ★★★★★ | ★★★★★ | ★★☆☆☆ | ★★★☆☆ | ★★★★☆ | ★★★★☆ | ★★★☆☆ |
| Screen + print from one source | ★★★★☆ | ★★★★☆ | ★★★★★ | ★★★☆☆ | ★★★★★ | ★★☆☆☆ | ★★★☆☆ | ★☆☆☆☆ | ★★★☆☆ |
| PDF/X-4 spec conformance | ✗ | ✗ | ★★★★☆ | ✗ | ✗ | ★★★☆☆ † | ✗ | ✗ | ★★★★★ |
| Press-ready output †† | ✗ | ✗ | ★★★☆☆ | ✗ | ✗ | ★★★☆☆ | ✗ | ✗ | ★★★★★ |
| Math typesetting | ★★☆☆☆ | ★★☆☆☆ | ★★★☆☆ | ★★★★★ | ★★☆☆☆ | ★★★★★ | ★☆☆☆☆ | ★★☆☆☆ | ★★★☆☆ |
| Text typesetting quality | ★★★★☆ | ★★★☆☆ | ★★★★★ | ★★★★★ | ★★★★☆ | ★★★★★ | ★★☆☆☆ | ★★★☆☆ | ★★★★★ |
| Human-readable source | ★★★★★ | ★★★★★ | ★★★★★ | ★★★★☆ | ★★★★★ | ★★★☆☆ | ★★☆☆☆ | ★★☆☆☆ | ✗ (binary) |
| Version control / CI-friendly | ★★★★★ | ★★★★★ | ★★★★★ | ★★★★★ | ★★★★★ | ★★★★★ | ★★★★★ | ★★★★★ | ★☆☆☆☆ |
| Learning curve | Medium | Low | Low | High | Medium | Very High | Medium | Medium | High |
| License cost | Free | Free | $$$ (~$495/yr) | Free | Free | Free | Free | Free / Pro | $$$ ($55/mo) |
| Ecosystem | Node | Python | Any | Any | Node | Any | Node/React | Python | macOS/Win GUI |
| Maintenance status | Active | Active | Active | Active | Active (Google) | Active | Active | Active | Active |

**†† Press-ready output** is not a tool capability — it is a workflow outcome. PDF/X spec conformance (above row) is a necessary but not sufficient condition. A press-ready file also requires: BleedBox + TrimBox set at 3mm offset from trim, an ICC OutputIntent profile matching the press condition (e.g. FOGRA51 for coated EU, GRACoL 2006 for US), total area coverage ≤ the printer's limit (typically 300% coated / 260% uncoated), overprint behaviour verified, and spot colours named exactly to the printer's ink library. **No HTML-to-PDF tool produces all of this automatically.** InDesign's ★★★★★ rating reflects its Ink Manager, Separations Preview, and prepress workflow — not just spec output. For commercial press work from any code-based tool, a prepress review step is required. "Prince is required for PDF/X at press" is false — Ghostscript can post-process any well-formed PDF to PDF/X-4.

**wkhtmltopdf** is excluded — unmaintained since 2020, known CVEs. See `references/tools/wkhtmltopdf.md`.

### Matrix reading guide

**"Screen + print from one source"** — can the same source produce both a web HTML page AND a print-quality PDF without two codebases? Headless Chrome scores highest because it renders the actual web page.

**"CI-friendly"** — can this run unattended in Docker/GitHub Actions? InDesign scores lowest — requires GUI and a license seat.

**"Text typesetting quality"** — line-breaking, hyphenation, spacing, paragraph texture. LaTeX and Prince are the reference standard.

**"Running headers/footers — WeasyPrint"** — updated to ★★★★☆ for WeasyPrint ≥ v60 (Nov 2023), which substantially improved named-string and running-element support. If on < v60, treat as ★★★☆☆. Upgrade before evaluating.

**Interpretation note:** ⚠️ means "technically supported but known bugs require workarounds." Check the relevant deep-dive file before committing.
