# Portfolio Site — Design Spec (plantplot first)

**Date:** 2026-06-16
**Author:** Jamie Meggas (with Claude)
**Status:** Approved — ready for implementation planning

## Goal

A clean, editorial portfolio website for Jamie Meggas (brand & identity
designer). It launches with **one complete case study (plantplot)** and is
structured so additional projects can be added by copying a content folder,
dropping in images, and writing copy — no template rework.

## Success criteria

- The plantplot case study reads as a polished, narrative-driven project page.
- Site builds locally with `hugo` and deploys to GitHub Pages.
- Adding project #2 requires no changes to layouts/CSS — only a new content
  bundle + assets.
- Works on mobile and desktop; loads fast; no JS framework or build step
  beyond Hugo.

## Tech & hosting

- **Generator:** Hugo (single Go binary; install via `brew install hugo`).
- **Hosting:** GitHub Pages, built from this repo (`Projects/GitHub/Portfolio`).
- **Theme:** custom lightweight theme authored in-repo (no third-party theme),
  for full creative control.
- **No Node / no JS framework.** Progressive enhancement only (small vanilla JS
  for things like a lightbox is optional, not required for v1).

## Site identity (Direction A — editorial neutral)

The site chrome stays neutral so each project carries its own color.

- Background: off-white `#FAF9F5`
- Text: near-black `#111111`; muted `#555555`; hairlines `#E8E6DD`
- Type: a bold geometric/grotesque for headings, clean sans for body
  (web-safe / Google Fonts; final faces chosen in build — candidates:
  headings in a heavy grotesque, body in DM Sans to echo plantplot's system)
- Generous whitespace, large type, hairline dividers, no gradients/shadows.
- Each project page may set an **accent color**; plantplot's is the greens.

## Information architecture

```
/                     home — name + role, work grid (plantplot card only for now)
/work/plantplot/      the case study
/about/               short bio + contact (stub for now)
```

- Projects are Hugo page bundles under `content/work/<slug>/` so each owns its
  copy and images.
- Home work grid is generated from `content/work/*` — new projects appear
  automatically.

## Hugo project structure (target)

```
config.toml                     # baseURL, title, params (author, social)
content/
  _index.md                     # home intro copy
  about.md                      # bio + contact
  work/
    plantplot/
      index.md                  # front matter + case-study body/copy
      images/                   # curated, web-optimized deck exports
layouts/
  _default/baseof.html          # head, header, footer shell
  index.html                    # home: intro + work grid
  work/single.html              # case-study template (section-driven)
  partials/                     # header, footer, project-card, image-figure
assets/css/                     # site styles (editorial-neutral tokens)
static/                         # favicon, CNAME (if custom domain), etc.
.github/workflows/hugo.yml      # build + deploy to GitHub Pages
```

The case-study template is **section-driven**: `work/single.html` renders a
sequence of content sections (each = heading + prose + one or more figures), so
every project uses the same template with different content.

## plantplot case study — content map

Copy is pulled from the brand deck and lightly edited for the web. Image
assignments below reference deck pages; the **exact page→section selection is
finalized during build** from `2026/plantploit/{PNGS,SVGS}/` (SVG preferred
where crisp, PNG otherwise). Target ~10 curated images.

1. **Hero** — full-bleed deep-green (`#004231`) with the flower mark + wordmark.
   Eyebrow: "Brand identity · 2026". Title: "plantplot".
   *(deck p.01 / p.23 style)*

2. **Overview** — tagline + pitch, plus a meta strip.
   - Tagline: "plantplot helps gardeners remember what grows."
   - Pitch: "Part garden planner, part living archive — plantplot lets you map
     your landscape, organize your plants, and document the seasons as your
     garden changes over time."
   - Meta: Role = Identity & system · Deliverables = Logo, type, color, pattern,
     applications · Year = 2026. *(deck p.10)*

3. **The mark** — "garden plot pin": flower + location pin.
   - the four seasons (four circles) · the north star (center) · location marker
     (pin/leaf base). *(deck p.15, p.05)*

4. **Logo system** — primary lockups, stacked/horizontal, app icon, on-color
   variants. *(deck p.02, p.05, p.13)*

5. **Color & pattern** — core greens `#004231 / #2c6455 / #00b86c`; seasonal
   palette spring `#ffb813`, summer `#25e8e3`, fall `#d435d8`, winter `#191040`;
   the diamond/four-point tessellation. *(deck p.04, p.03/p.21)*

6. **Type & voice** — the "garden fonts" specimen and Warbler headline
   ("Nothing worth growing should be forgotten."); branded words: grounded,
   rooted, archival, legacy, seasonal, cultivated, intentional, blooming.
   *(deck p.07, p.08, p.11)*

7. **Seasonal system** — spring / summer / fall / winter expressions.
   *(deck seasonal pages)*

8. **Applications** — business card (Michelle Horton, Master Gardener),
   "full of color" mark colorways, app icon in context. *(deck p.09, p.17/19, p.13)*

9. **The story ("Remember what grows")** — the designer's personal statement as
   a strong close: memory, loss, the garden as a living archive, and design's
   power to "preserve what matters — not just information, but connection."
   *(deck p.22, p.23)*

10. **Next project** — link to the next case study (stub/hidden until project #2
    exists).

**Optional:** a "full brand deck" gallery (all 23 pages) at the bottom, behind a
simple expander or as a final section, if completeness is wanted.

## Image handling

- Source: `2026/plantploit/PNGS/*.png` and `SVGS/*.svg` (23 pages each).
- Curate ~10 into `content/work/plantplot/images/`, renamed semantically
  (e.g. `mark.svg`, `color-story.png`).
- Prefer SVG for crisp vector pages; use PNG where rasterized detail matters.
- Hugo image processing/`figure` partial provides responsive sizing + alt text.

## Build & deploy

- Local: `hugo server` for preview; `hugo` for production build to `public/`.
- Deploy: GitHub Actions workflow (`.github/workflows/hugo.yml`) builds on push
  to `main` and publishes to GitHub Pages.
- `baseURL` set for the Pages URL (or custom domain via `static/CNAME`).

## Out of scope (v1, YAGNI)

- Blog / CMS / RSS
- Contact form backend (about page lists email only)
- 169podcast and mossmark case studies (structure supports them later)
- Heavy animation / JS framework
- Dark mode for the site chrome

## Open items to confirm during build

- Final heading/body typefaces.
- Exact deck pages chosen per section.
- Whether to include the full-deck gallery.
- GitHub Pages URL vs. custom domain (affects `baseURL`).
