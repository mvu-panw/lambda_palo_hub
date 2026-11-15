# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this directory is

There are two things here:

1. **`site/`** is a **customer-facing** static resource hub. Lambda employees use it to learn about and download Palo Alto Networks datasheets, Prisma AIRS guides, and GenAI reference architectures. It is pushed to the **public** repo `github.com/mvu-panw/lambda_palo_hub` (branch `main`), and `.github/workflows/pages.yml` publishes only `site/` to GitHub Pages. One-time manual step: Settings → Pages → Source = "GitHub Actions".
2. **Everything else** is a local-only PDF reference library of about 1.1 GB. `.gitignore` is an allow-list (`/*`, then `!/site/`, `!/.github/`, and so on), so nothing new at the top level gets committed unless you add it there on purpose.

The UI is a sibling of the internal account site in `~/lambda/site` (the `lambda_research` repo). That site holds account positioning, competitive notes, and recommendations, and **none of that may appear here**. Keep copy neutral and sourced from public PANW material: no incumbent-vendor talk, no "we recommend model X for you", and no internal-only wording.

**Never publish `AI_Factory_Security_Blueprint.pdf`.** Every page is marked "Proprietary and confidential information." The two `lambda_resources/` topology diagrams *are* published (on `topology.html`, at the user's request); they're the same files already public on the `lambda_research` site.

## Commands

```bash
python3 -m http.server 8100 --directory site    # preview (port 8100 by convention)
```

There is no build, lint, or test. To check links, extract every `href`/`src` from `site/*.html` and curl each against the local server. Headless Chrome can't render narrower than about 500px, so to check mobile layout, screenshot a scratch page that iframes the site at `width="390"`.

## Site conventions

- Nine pages, one per platform plus hub pages:

  | Page | Holds |
  |---|---|
  | `index` | Quick downloads (PA-5500/5400/3400/1400 + AIRS datasheet), Explore grid linking every page |
  | `firewalls` | Spec comparison table, per-series cards |
  | `ai-security` | AIRS datasheet, 4 AIRS product guides, 8 GenAI reference architectures |
  | `soc` | Cortex XSIAM and Cortex Agentic Endpoint Security (datasheet + at-a-glance for each) |
  | `identity` | Idira PAM, Endpoint Privilege Manager, IGA |
  | `cloud-management` | Strata Cloud Manager, Next-Generation Trust Security, certificate-outage brief (both trust PDFs say they extend SCM) |
  | `unit42` | Unit 42 Threat Intel Services |
  | `topology` | Embedded Lambda diagrams |
  | `resources` | Remaining firewall briefs (single-pass, clustering, Precision AI bundle, PA-400R), a "Datasheets by platform" index with hand-written download counts, online docs |

  When a new PDF lands in `Datasheets/`, put it on the platform page it belongs to (and only there), and update the download count on the matching `resources.html` card. Each platform page follows the same pattern: featured `.dl-card` downloads, then summary cards written *only* from the datasheet text, then "Online" links. Check every number against the PDF.
- **No templating:** the nav and footer are copied into every page. The nav lists every page except the current one, with Home first. Regenerate it on all nine pages at once (a short Python loop over a `(file, label)` list that replaces the `<div class="nav-links">` block) rather than editing by hand. Nav wraps to its own row under 1240px.
- `site/assets/css/styles.css` is lines 1–445 of `~/lambda/site/assets/css/styles.css` (the ai_sec_by_design template), unchanged, plus a block at the end marked "Lambda resource hub additions" (`.dl-grid`/`.dl-card` download cards, `.featured` gradient border, `.series`, `.pill-row`). The stylesheet is linked as `styles.css?v=5`; bump the version on every page together after editing the CSS.
- **Topology embeds:** each `.pdf-embed` is a link to the PDF that contains a PNG render (`site/assets/docs/topology/*.png`). PDF.js (self-hosted in `site/assets/vendor/pdfjs-3.11.174/`) swaps in a canvas render of page 1. It must be same-origin: loaded from cdnjs, the cross-origin worker stalls and `getDocument` never resolves. After editing a diagram PDF, re-render the PNG with `pdftoppm -png -r 150 -singlefile in.pdf out` so the fallback matches. Headless Chrome screenshots are unreliable for confirming the canvas swap; use `--timeout=15000` and treat the PNG as an acceptable result.
- Downloads are plain `<a href="assets/docs/..." download>` links. PDFs live in `site/assets/docs/{datasheets,airs,genai}/` under URL-safe renamed filenames; the originals stay in the library folders. Each card's page count and size (`PDF · N pages · X MB`) is written by hand, so update it when you swap a PDF (`pdfinfo` for pages, `du -h` for size).
- Spec numbers and port lists on `firewalls.html` and the index cards come straight from the datasheet tables. Re-extract them with `pdftotext -layout`. Some table cells only show up with `grep -B3 -A3` because of line wrapping.
- GitHub rejects files over 100 MB and warns above 50 MB. The largest hosted PDF is the AIRS admin guide at 33 MB.

## Reference library layout

| Folder | Contents |
|---|---|
| `Datasheets/` | PA-400R/1400/3400/5400/5500, NGFW clustering, Prisma AIRS runtime, Precision AI bundle, Unit 42, cert/trust briefs |
| `AIRS/` | Prisma AIRS: runtime security admin and onboarding, AI model security, AI red teaming |
| `NGFW & PAN-OS/` | PAN-OS admin, networking, CLI, API, Panorama API, GlobalProtect, User-ID, PA-5500 hardware reference, 12.1.8 release notes |
| `NGFW & PAN-OS/12.1/` | Newer builds of five of those guides. Prefer these; only `ngfw-api.pdf` is identical to the top-level copy. |
| `SCM/`, `VM/` | Strata Cloud Manager and VM-Series guides |
| `Reference Architectures/` | Design, deployment, and solution guides by domain (`<Title> - <Design\|Deployment\|Solution> Guide - <MonYY>.pdf`) |
| `lambda_resources/` | Lambda topology diagram sources (copies published under `site/assets/docs/topology/`) |

To read the PDFs, use `pdftotext` and `pdfinfo` (`pypdf` isn't installed). Extract the text and grep it, or read page ranges with `-f N -l M`. Many guides run to 1,000+ pages. Paths contain spaces, `&`, and parentheses, so always quote them.
