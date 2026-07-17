---
name: publish-case-study
description: >
  Add and publish a new project/case study to Jamie Meggas's Hugo portfolio
  (jmeggas.com). Use this whenever the user wants to add a new case study,
  project, or brand to the portfolio, hands over a new brand-deck folder,
  points at a project folder under ~/Projects/GitHub/, or says things like
  "publish X to the site," "add a new project," or "make a case study for Y" —
  even if they don't name this skill. It codifies the required flow: branch →
  build the case study → local preview → the user's explicit approval → merge
  to main → push (which auto-deploys). Always follow this instead of improvising
  the steps, so new work is consistent, isolated on a branch, and never reaches
  main without approval.
---

# Publishing a new case study

This skill turns a folder of brand-deck art into a live case study on
`jmeggas.com`, safely and consistently. The site is a Hugo static site; a new
project is "just" a content bundle, so the hard parts are curation, accuracy,
and process discipline — not code.

## Non-negotiables (why this skill exists)

1. **Work on a branch, never on `main`.** New work is isolated so `main` always
   reflects what's live. Create `case-study/<slug>` before touching anything.
2. **The user previews locally and approves before anything merges.** Nothing
   reaches `main` (and therefore the public site) until the user has seen it
   running locally and said yes. This is the whole point — treat it as a hard
   gate.
3. **Verify every deck image by eye.** Brand-deck file numbers do NOT reliably
   map to content (this bit us on the first project — an SVG export order was
   scrambled and a "Summer" slide landed in the "mark" slot). Look at each image
   before assigning it to a section.

## The handoff contract (how the user gives you a case study)

The user drops the raw brand files in a **sibling folder** next to the repo:
`~/Projects/GitHub/<ProjectName>/`, containing at least:

- `PNGs/` — the full brand deck exported as PNGs (one per page/slide).
- `THUMB/` *(recommended)* — the same pages exported smaller (~1920px wide) and
  with **solid/white backgrounds** (not transparent — transparent art looks
  broken on the dark lightbox overlay). Prefer these as sources.
- `mp4/` *(optional)* — a logo animation for an autoplay hero video.

The user may also specify any of: accent color, hero image, tagline, project
year, or where it should sit in the running order. If they don't, propose
sensible choices and confirm (see step 3). If the handoff folder isn't obvious,
ask where it is before starting.

## Repo conventions (read once, they explain the steps)

- Projects are Hugo page bundles: `content/work/<slug>/index.md` + `images/`.
- One section-driven template (`layouts/work/single.html` +
  `partials/section.html`) renders a `sections:` list from front matter, so
  **adding a project needs no template/CSS changes** — only content + images.
- The home grid and the "next project" links are generated automatically.
  Ordering is by `date` (descending) — the **newest date shows first**.
- Images are auto-resized to WebP: cards and inline figures become small
  thumbnails; each figure links to a larger version opened in a lightbox. So
  keep source exports lean (~1920px is plenty) but you don't hand-resize.
- Accent color drives the hero band + hover states. Pick a **dark, brand-true**
  color so white hero text stays legible, and one that's distinct from the other
  projects' accents on the home grid.
- Deploy is automatic: pushing `main` triggers GitHub Actions → GitHub Pages →
  `jmeggas.com`. A `static/CNAME` keeps the custom domain wired up.

Look at an existing bundle (e.g. `content/work/sunny-bungalow/index.md`) as the
canonical example before writing a new one.

## The workflow

### 1. Branch and set up local preview

```bash
git checkout main && git pull --ff-only    # start from what's live
git checkout -b case-study/<slug>
```

Start the local dev server so the user can watch changes as you go. Prefer the
preview tooling (`preview_start` with the `hugo` config in `.claude/launch.json`,
which runs `hugo server` on :1313); fall back to `hugo server -D` in a background
shell. Tell the user the local URL.

### 2. Study the deck and build an accurate index

Read **every** page in `PNGs/` (or `THUMB/`) and note what each one actually is —
logo/mark, logo variations, color, type, patterns, applications/mockups,
lifestyle photos, closing, and any pages that are mostly written copy (pull that
copy — it's the case-study narrative in the brand's own voice). Do not trust the
filename number; trust your eyes.

### 3. Propose the case-study map and get a quick OK

Before building, present a short plan and let the user adjust:

- **slug**, **title**, **accent** (hex), **year**
- **home-grid card thumbnail** and **hero** (image, or `heroVideo` for a loop)
- **tagline** (one line, sentence case)
- **section list** — each section = heading + short body (from the deck) + the
  image(s) assigned to it

This is a light gate, not a ceremony — one message, sensible defaults, proceed
on approval.

### 4. Curate images into the bundle

Copy the chosen pages into `content/work/<slug>/images/` with **semantic names**
(`hero.png`, `mark.png`, `color.png`, `applications.png`, …). Prefer the `THUMB/`
exports (lean + solid background). Set `thumb` (home card) and `hero` separately
if you want a different card image than the hero. A real photograph often makes
the best card thumbnail for a place/product; a clean lockup for an identity.

### 5. Write `index.md`

Front matter + a `sections` list. Give the newest project a `date` after the
current top project so it leads the grid. Give **every image its own descriptive
`alt`** via the `{ src, alt }` map form (screen readers announce each figure, and
one-alt-per-section reads as duplicates). For an animated hero, add
`heroVideo: "<file>.mp4"` and keep `hero` as the poster still. Mirror the tone
and structure of an existing `index.md`.

### 6. Build, self-check, then hand the preview to the user

```bash
hugo --gc --minify        # must be warning/error-free
```

Verify in the browser yourself first: the home grid shows the new card, the case
study renders every section with the **correct** images (this is where scrambled
mappings surface), the hero looks right, the lightbox opens a larger image, and
cross-links chain correctly. Fix anything off. Then tell the user it's ready at
the local URL and **ask them to review and approve**.

### 7. Approval → merge → push (only after explicit yes)

Do not merge or push until the user approves. When they do:

```bash
git add content/work/<slug> [any layout/asset changes]
git commit -m "feat: add <Project> case study"
git checkout main && git merge --no-ff case-study/<slug>
hugo --gc --minify        # re-verify on merged main
git push origin main      # triggers deploy
git branch -d case-study/<slug>
```

Then confirm the deploy went green and the page is live
(`curl -sI https://jmeggas.com/work/<slug>/` → 200), and share the live URL.

## Gotchas worth remembering

- **Image identity over filenames** — always verify by eye (see non-negotiable 3).
- **Transparent art on the lightbox** — the overlay is dark; transparent PNGs
  look broken enlarged. Ask for / use solid-background exports.
- **Keep raw source out of git** — the huge original decks live in the sibling
  folder and are gitignored (`2026/`, and per-project raw folders). Only curated,
  web-ready images belong in `content/`.
- **DPI is irrelevant for web** — what matters is pixel dimensions. "72 dpi" only
  helps if the pixel size actually dropped.
- **Custom domain** — never delete `static/CNAME`; it keeps `jmeggas.com`
  pointed at the Actions deploy.

## Quick checklist

- [ ] On a `case-study/<slug>` branch (not `main`)
- [ ] Local `hugo server` running; user has the URL
- [ ] Every deck image verified by eye; copy pulled from the deck
- [ ] Map (slug/accent/hero/thumb/tagline/sections) approved
- [ ] Images curated with semantic names + per-image alt text
- [ ] `hugo --gc --minify` clean; you verified the page in-browser
- [ ] **User approved via local preview**
- [ ] Merged to `main`, pushed, deploy green, live URL confirmed
