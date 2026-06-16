# Portfolio Site (plantplot) Implementation Plan

> **For agentic workers:** REQUIRED: Use superpowers:subagent-driven-development (if subagents available) or superpowers:executing-plans to implement this plan. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Build an editorial, Hugo-powered portfolio site that ships one complete plantplot case study and is structured to add more projects with no template changes.

**Architecture:** Custom in-repo Hugo theme (no third-party theme). Editorial-neutral chrome (off-white, Object Sans, lots of whitespace). Projects are Hugo page bundles under `content/work/<slug>/`; the home grid and a single section-driven case-study template render them. Deploy to GitHub Pages via GitHub Actions.

**Tech Stack:** Hugo (extended), Go templates, hand-written CSS, self-hosted Object Sans (`.woff2`), GitHub Pages + Actions. No Node, no JS framework.

**Spec:** `docs/superpowers/specs/2026-06-16-portfolio-plantplot-design.md`

---

## File structure (target)

```
hugo.toml                              # site config: baseURL, title, params
content/
  _index.md                            # home intro copy
  about.md                             # bio + contact
  work/
    plantplot/
      index.md                         # case-study front matter + section copy
      images/                          # curated, web-ready deck exports
layouts/
  baseof.html                          # html shell: head, header, footer
  index.html                           # home: intro + work grid
  page.html                            # generic single page (about)
  work/single.html                     # section-driven case-study template
  partials/
    head.html                          # meta, css, fonts
    header.html                        # site nav
    footer.html                        # site footer
    project-card.html                  # work-grid card
    section.html                       # one case-study section (heading+prose+figures)
assets/
  css/main.css                         # editorial-neutral tokens + layout
  fonts/                               # Object Sans .woff2 (Regular, Heavy)
static/
  .nojekyll                            # let Hugo output serve as-is on Pages
.github/workflows/hugo.yml             # build + deploy
```

> Note: Hugo lookup order means top-level `layouts/baseof.html` and
> `layouts/work/single.html` work without a `themes/` dir. Keeping the theme
> in the project root (not a sub-theme) is intentional and simplest here.

---

## Chunk 1: Buildable skeleton

### Task 1: Install Hugo and scaffold a building site

**Files:**
- Create: `hugo.toml`
- Create: `layouts/baseof.html`
- Create: `layouts/index.html`
- Create: `content/_index.md`
- Create: `static/.nojekyll`

- [ ] **Step 1: Install Hugo (extended)**

Run: `brew install hugo` then `hugo version`
Expected: prints a version string containing `extended` (e.g. `hugo v0.x ... extended`).

- [ ] **Step 2: Write `hugo.toml`**

```toml
baseURL = "/"
languageCode = "en-us"
title = "Jamie Meggas"

[params]
  role = "Brand & identity designer"
  email = "jlmeggas@gmail.com"

[markup.goldmark.renderer]
  unsafe = true
```

(`unsafe = true` lets curated inline HTML in content render; `baseURL` is
finalized in the deploy task.)

- [ ] **Step 3: Write minimal `content/_index.md`**

```markdown
---
title: "Jamie Meggas"
---

Brand & identity designer.
```

- [ ] **Step 4: Write `layouts/baseof.html`**

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="utf-8">
  <meta name="viewport" content="width=device-width, initial-scale=1">
  <title>{{ if .IsHome }}{{ site.Title }}{{ else }}{{ .Title }} · {{ site.Title }}{{ end }}</title>
</head>
<body>
  {{ block "main" . }}{{ end }}
</body>
</html>
```

- [ ] **Step 5: Write minimal `layouts/index.html`**

```html
{{ define "main" }}
  <h1>{{ site.Title }}</h1>
  <p>{{ site.Params.role }}</p>
{{ end }}
```

- [ ] **Step 6: Create `static/.nojekyll`** (empty file)

Run: `touch static/.nojekyll`

- [ ] **Step 7: Build and verify**

Run: `hugo --gc --minify`
Expected: build succeeds, `public/index.html` exists and contains
`Brand &amp; identity designer` (verify: `grep -q "identity designer" public/index.html && echo OK`).

- [ ] **Step 8: Commit**

```bash
git add hugo.toml layouts/baseof.html layouts/index.html content/_index.md static/.nojekyll
git commit -m "feat: scaffold buildable Hugo site"
```

---

### Task 2: Object Sans fonts + CSS foundation

**Files:**
- Create: `assets/fonts/ObjectSans-Regular.woff2`, `assets/fonts/ObjectSans-Heavy.woff2`
- Create: `assets/css/main.css`
- Create: `layouts/partials/head.html`
- Modify: `layouts/baseof.html` (use head partial + css)

- [ ] **Step 1: Obtain Object Sans (free PangramPangram release)**

Download the free Object Sans family, convert to `.woff2` if needed, and place
`ObjectSans-Regular.woff2` and `ObjectSans-Heavy.woff2` in `assets/fonts/`.
Verify license permits web embedding.
Expected: `ls assets/fonts/*.woff2` lists both files.

- [ ] **Step 2: Write `assets/css/main.css` (tokens + base)**

```css
:root{
  --bg:#FAF9F5; --ink:#111; --muted:#555; --hair:#E8E6DD;
  --maxw:1100px;
}
@font-face{font-family:"Object Sans";src:url("/fonts/ObjectSans-Regular.woff2") format("woff2");font-weight:400;font-display:swap}
@font-face{font-family:"Object Sans";src:url("/fonts/ObjectSans-Heavy.woff2") format("woff2");font-weight:800;font-display:swap}
*{box-sizing:border-box}
html{font-size:18px}
body{margin:0;background:var(--bg);color:var(--ink);
  font-family:"Object Sans",system-ui,-apple-system,Segoe UI,Roboto,sans-serif;
  line-height:1.6;-webkit-font-smoothing:antialiased}
h1,h2,h3{font-weight:800;line-height:1.05;letter-spacing:-0.02em;margin:0 0 .4em}
a{color:inherit}
img{max-width:100%;display:block}
.wrap{max-width:var(--maxw);margin:0 auto;padding:0 24px}
```

- [ ] **Step 3: Write `layouts/partials/head.html`**

```html
{{ $css := resources.Get "css/main.css" | minify | fingerprint }}
{{ $reg := resources.Get "fonts/ObjectSans-Regular.woff2" }}
<link rel="preload" as="font" type="font/woff2" href="{{ $reg.RelPermalink }}" crossorigin>
<link rel="stylesheet" href="{{ $css.RelPermalink }}">
```

> The `@font-face` src uses `/fonts/...`; ensure fonts are emitted to that path.
> Simplest: also copy fonts to `static/fonts/` OR publish via
> `{{ $reg.Publish }}`. Implementer: place the two `.woff2` in `static/fonts/`
> so `/fonts/ObjectSans-*.woff2` resolves, and keep the preload via assets.
> (If using `static/fonts/`, the preload href can be hardcoded `/fonts/...`.)

- [ ] **Step 4: Wire head partial into `baseof.html`**

Replace the `<head>` inner tags after `<title>` with `{{ partial "head.html" . }}`
(keep charset, viewport, title; add the partial before `</head>`).

- [ ] **Step 5: Build and verify CSS + fonts load**

Run: `hugo --gc --minify`
Expected: build succeeds; `public/index.html` references a `main.<hash>.css`;
`public/fonts/ObjectSans-Regular.woff2` exists
(verify: `ls public/fonts/ && grep -q "main\..*\.css" public/index.html && echo OK`).

- [ ] **Step 6: Visual check**

Run: `hugo server` and open the local URL; confirm Object Sans renders and
background is off-white.

- [ ] **Step 7: Commit**

```bash
git add assets layouts/partials/head.html layouts/baseof.html
git commit -m "feat: self-hosted Object Sans + CSS foundation"
```

---

## Chunk 2: Home, header/footer, work grid

### Task 3: Header and footer partials

**Files:**
- Create: `layouts/partials/header.html`, `layouts/partials/footer.html`
- Modify: `layouts/baseof.html` (include header/footer around main)

- [ ] **Step 1: Write `layouts/partials/header.html`**

```html
<header class="site-head wrap">
  <a class="logo" href="/">JM</a>
  <nav><a href="/#work">work</a> <a href="/about/">about</a></nav>
</header>
```

- [ ] **Step 2: Write `layouts/partials/footer.html`**

```html
<footer class="site-foot wrap">
  <p>&copy; {{ now.Year }} {{ site.Title }} · <a href="mailto:{{ site.Params.email }}">{{ site.Params.email }}</a></p>
</footer>
```

- [ ] **Step 3: Include in `baseof.html`**

Wrap the body content:
```html
{{ partial "header.html" . }}
<main>{{ block "main" . }}{{ end }}</main>
{{ partial "footer.html" . }}
```

- [ ] **Step 4: Add header/footer CSS to `main.css`**

```css
.site-head{display:flex;justify-content:space-between;align-items:center;padding-top:24px;padding-bottom:24px}
.site-head .logo{font-weight:800;text-decoration:none}
.site-head nav a{margin-left:18px;color:var(--muted);text-decoration:none}
.site-head nav a:hover{color:var(--ink)}
.site-foot{border-top:1px solid var(--hair);margin-top:80px;padding:32px 24px;color:var(--muted);font-size:.85rem}
```

- [ ] **Step 5: Build + verify**

Run: `hugo --gc --minify`
Expected: `public/index.html` contains `site-head` and `mailto:`
(verify: `grep -q "site-head" public/index.html && echo OK`).

- [ ] **Step 6: Commit**

```bash
git add layouts/partials/header.html layouts/partials/footer.html layouts/baseof.html assets/css/main.css
git commit -m "feat: site header and footer"
```

---

### Task 4: Home hero + work grid

**Files:**
- Create: `layouts/partials/project-card.html`
- Modify: `layouts/index.html`, `assets/css/main.css`, `content/_index.md`

- [ ] **Step 1: Write `layouts/partials/project-card.html`**

```html
<a class="card" href="{{ .RelPermalink }}" style="--accent:{{ .Params.accent | default "#103D2E" }}">
  <span class="card-thumb">{{ with .Params.thumb }}<img src="{{ . }}" alt="{{ $.Title }} thumbnail">{{ end }}</span>
  <span class="card-meta">
    <strong>{{ .Title }}</strong>
    <em>{{ .Params.subtitle }}</em>
  </span>
</a>
```

- [ ] **Step 2: Rewrite `layouts/index.html`**

```html
{{ define "main" }}
<section class="wrap hero">
  <h1>Brand &amp; identity designer.</h1>
  <p class="lede">Logos, systems, and stories. Selected work below.</p>
</section>
<section class="wrap work" id="work">
  <div class="grid">
    {{ range where site.RegularPages "Section" "work" }}
      {{ partial "project-card.html" . }}
    {{ end }}
  </div>
</section>
{{ end }}
```

- [ ] **Step 3: Add hero + grid CSS to `main.css`**

```css
.hero{padding:64px 24px 40px}
.hero h1{font-size:clamp(2.4rem,7vw,4.5rem)}
.hero .lede{color:var(--muted);max-width:34ch;font-size:1.1rem}
.grid{display:grid;grid-template-columns:repeat(auto-fill,minmax(280px,1fr));gap:24px}
.card{display:block;text-decoration:none;border:1px solid var(--hair);border-radius:12px;overflow:hidden;background:#fff}
.card-thumb{display:block;aspect-ratio:16/10;background:var(--accent)}
.card-thumb img{width:100%;height:100%;object-fit:cover}
.card-meta{display:block;padding:14px 16px}
.card-meta strong{display:block}
.card-meta em{color:var(--muted);font-style:normal;font-size:.9rem}
```

- [ ] **Step 4: Build + verify (grid renders even with the plantplot stub once Task 5 lands)**

Run: `hugo --gc --minify`
Expected: build succeeds; `public/index.html` contains `class="grid"`
(verify: `grep -q 'class="grid"' public/index.html && echo OK`).

- [ ] **Step 5: Commit**

```bash
git add layouts/index.html layouts/partials/project-card.html assets/css/main.css
git commit -m "feat: home hero and work grid"
```

---

## Chunk 3: plantplot content + case-study template

### Task 5: Curate plantplot images

**Files:**
- Create: `content/work/plantplot/images/*` (curated exports)

- [ ] **Step 1: Select and copy ~10–12 deck assets**

Source: `2026/plantploit/SVGS/*.svg` (preferred) and `2026/plantploit/PNGS/*.png`.
Copy into `content/work/plantplot/images/` with semantic names. Suggested set
(confirm exact page during this step by eyeballing the files):

| Target name | Source (likely) | Use |
|---|---|---|
| `hero.png` | deep-green wordmark page | hero / card thumb |
| `mark.svg` | "garden plot pin" page | the mark |
| `logo-system.png` | lockups + app icon page | logo system |
| `color-story.png` | "color story" page | color & pattern |
| `pattern.svg` | diamond tessellation page | color & pattern |
| `type.png` | "garden fonts" page | type & voice |
| `branded-words.png` | "branded words" page | type & voice |
| `season-spring.png` | spring page | seasonal |
| `season-summer.png` | summer page | seasonal |
| `season-fall.png` | fall page | seasonal |
| `season-winter.png` | winter page | seasonal |
| `business-card.png` | business card page | applications |

- [ ] **Step 2: Verify**

Run: `ls content/work/plantplot/images/`
Expected: lists the curated files.

- [ ] **Step 3: Commit**

```bash
git add content/work/plantplot/images
git commit -m "assets: curate plantplot case-study images"
```

---

### Task 6: plantplot content file (front matter + section copy)

**Files:**
- Create: `content/work/plantplot/index.md`

- [ ] **Step 1: Write front matter + sections**

Use a `sections` list in front matter so the template stays generic. Copy is
pulled from the brand deck.

```markdown
---
title: "plantplot"
subtitle: "Brand identity"
date: 2026-01-01
accent: "#00b86c"
eyebrow: "Brand identity · 2026"
thumb: "images/hero.png"
hero: "images/hero.png"
tagline: "plantplot helps gardeners remember what grows."
meta:
  - { label: "Role", value: "Identity & system" }
  - { label: "Deliverables", value: "Logo, type, color, pattern, applications" }
  - { label: "Year", value: "2026" }
sections:
  - heading: "Overview"
    body: "Part garden planner, part living archive — plantplot lets you map your landscape, organize your plants, and document the seasons as your garden changes over time. Track what thrives, what returns, and the stories rooted in every corner of your space."
  - heading: "The mark"
    body: "A garden plot pin: a geometric flower fused with a location marker. The four circles are the four seasons, the center is the north star, and the base reads as both leaf and map pin."
    images: ["images/mark.svg"]
  - heading: "Logo system"
    body: "Primary lockups in horizontal and stacked forms, on-color variants, and an app-icon treatment."
    images: ["images/logo-system.png"]
  - heading: "Color & pattern"
    body: "A grounded core of greens — #004231, #2c6455, #00b86c — extended by a seasonal palette and a four-point diamond tessellation drawn from the mark."
    images: ["images/color-story.png", "images/pattern.svg"]
  - heading: "Type & voice"
    body: "Garden fonts pair a confident grotesque with an editorial serif — 'Nothing worth growing should be forgotten.' The voice is grounded, rooted, archival, seasonal, cultivated, intentional, blooming."
    images: ["images/type.png", "images/branded-words.png"]
  - heading: "Seasonal system"
    body: "Each season gets its own color and message — built for every spring ahead, the season of growth, autumn leaves its mark, the season of reflection."
    images: ["images/season-spring.png", "images/season-summer.png", "images/season-fall.png", "images/season-winter.png"]
  - heading: "Applications"
    body: "Stationery and identity in use, from the master-gardener business card to a mark that is full of color."
    images: ["images/business-card.png"]
  - heading: "Remember what grows"
    body: "Gardens are living records, shaped season by season and rooted in memory. plantplot began with a personal question about how we hold on to fragile things — stories, places, the small details of a life. It imagines design as a way to preserve not just information, but connection: the relationship between people, memory, and the landscapes they care for."
---
```

- [ ] **Step 2: Build + verify the page exists**

Run: `hugo --gc --minify`
Expected: `public/work/plantplot/index.html` exists
(verify: `test -f public/work/plantplot/index.html && echo OK`).

- [ ] **Step 3: Commit**

```bash
git add content/work/plantplot/index.md
git commit -m "content: plantplot case-study copy"
```

---

### Task 7: Section partial + case-study template

**Files:**
- Create: `layouts/partials/section.html`
- Create: `layouts/work/single.html`
- Modify: `assets/css/main.css`

- [ ] **Step 1: Write `layouts/partials/section.html`**

`.` is one section map (from the `sections` list); `$.Page` is the page for
resolving bundle-relative image paths.

```html
<section class="cs-section wrap">
  <div class="cs-text">
    <h2>{{ .heading }}</h2>
    <p>{{ .body }}</p>
  </div>
  {{ with .images }}
  <div class="cs-figs">
    {{ range . }}
      {{ $img := $.page.Resources.GetMatch . }}
      {{ if $img }}<img src="{{ $img.RelPermalink }}" alt="{{ $.heading }}">{{ else }}<img src="{{ . | relURL }}" alt="{{ $.heading }}">{{ end }}
    {{ end }}
  </div>
  {{ end }}
</section>
```

> Implementer note: pass a dict so the partial can see both the section and the
> page — see template step below (`dict "heading" ... "page" $`).

- [ ] **Step 2: Write `layouts/work/single.html`**

```html
{{ define "main" }}
<article class="case-study" style="--accent:{{ .Params.accent | default "#103D2E" }}">

  <header class="cs-hero" style="background:var(--accent)">
    <div class="wrap">
      <p class="eyebrow">{{ .Params.eyebrow }}</p>
      <h1>{{ .Title }}</h1>
      {{ with .Params.tagline }}<p class="cs-tagline">{{ . }}</p>{{ end }}
    </div>
    {{ with .Params.hero }}{{ $h := $.Resources.GetMatch . }}{{ with $h }}<img class="cs-hero-img" src="{{ .RelPermalink }}" alt="{{ $.Title }}">{{ end }}{{ end }}
  </header>

  {{ with .Params.meta }}
  <div class="wrap cs-meta">
    {{ range . }}<div><span>{{ .label }}</span><strong>{{ .value }}</strong></div>{{ end }}
  </div>
  {{ end }}

  {{ range .Params.sections }}
    {{ partial "section.html" (dict "heading" .heading "body" .body "images" .images "page" $) }}
  {{ end }}

  <nav class="cs-next wrap">
    {{ with .NextInSection }}<a href="{{ .RelPermalink }}">Next project: {{ .Title }} &rarr;</a>{{ else }}<a href="/#work">&larr; Back to work</a>{{ end }}
  </nav>

</article>
{{ end }}
```

> Adjust `section.html` to read `.page` (the dict key) instead of `$.Page`:
> use `{{ $img := (index . "page").Resources.GetMatch ... }}`. Implementer
> reconciles the partial with the dict keys passed here; keep it simple.

- [ ] **Step 3: Add case-study CSS to `main.css`**

```css
.cs-hero{color:#fff;padding:72px 0 40px}
.cs-hero .eyebrow{text-transform:uppercase;letter-spacing:.12em;font-size:.75rem;opacity:.85;margin:0 0 8px}
.cs-hero h1{font-size:clamp(2.6rem,8vw,5rem)}
.cs-hero .cs-tagline{font-size:1.2rem;max-width:42ch;opacity:.95}
.cs-hero-img{width:100%;max-height:520px;object-fit:cover;margin-top:32px}
.cs-meta{display:flex;gap:40px;flex-wrap:wrap;border-bottom:1px solid var(--hair);padding:28px 24px}
.cs-meta span{display:block;color:var(--muted);font-size:.8rem}
.cs-section{display:grid;grid-template-columns:1fr;gap:20px;padding:56px 24px;border-bottom:1px solid var(--hair)}
.cs-text{max-width:60ch}
.cs-text p{color:#222}
.cs-figs{display:grid;grid-template-columns:repeat(auto-fit,minmax(240px,1fr));gap:16px}
.cs-figs img{border:1px solid var(--hair);border-radius:10px;width:100%}
.cs-next{padding:48px 24px 0;font-size:1.1rem}
.cs-next a{font-weight:800;text-decoration:none}
```

- [ ] **Step 4: Build + verify the case study renders sections + images**

Run: `hugo --gc --minify`
Expected: build succeeds; the page contains each heading and at least one image
(verify: `grep -q "The mark" public/work/plantplot/index.html && grep -q "cs-figs" public/work/plantplot/index.html && echo OK`).

- [ ] **Step 5: Visual check**

Run: `hugo server`; open `/work/plantplot/`; confirm hero is deep-green, sections
read top-to-bottom, images load (including the `.svg` ones), and the home grid
card links here.

- [ ] **Step 6: Commit**

```bash
git add layouts/work/single.html layouts/partials/section.html assets/css/main.css
git commit -m "feat: section-driven plantplot case study"
```

---

## Chunk 4: About page + deploy

### Task 8: About page

**Files:**
- Create: `content/about.md`
- Create: `layouts/page.html`

- [ ] **Step 1: Write `content/about.md`**

```markdown
---
title: "About"
---

Jamie Meggas is a brand & identity designer focused on logos, systems, and the
stories behind them.

Get in touch: [jlmeggas@gmail.com](mailto:jlmeggas@gmail.com)
```

- [ ] **Step 2: Write `layouts/page.html`**

```html
{{ define "main" }}
<article class="wrap page">
  <h1>{{ .Title }}</h1>
  {{ .Content }}
</article>
{{ end }}
```

- [ ] **Step 3: Add `.page` CSS to `main.css`**

```css
.page{padding:56px 24px;max-width:60ch}
.page p{color:#222}
```

- [ ] **Step 4: Build + verify**

Run: `hugo --gc --minify`
Expected: `public/about/index.html` exists and contains `mailto:`
(verify: `grep -q "mailto:jlmeggas" public/about/index.html && echo OK`).

- [ ] **Step 5: Commit**

```bash
git add content/about.md layouts/page.html assets/css/main.css
git commit -m "feat: about page"
```

---

### Task 9: GitHub Pages deploy workflow

**Files:**
- Create: `.github/workflows/hugo.yml`
- Modify: `hugo.toml` (set production `baseURL`)

- [ ] **Step 1: Decide and set `baseURL`**

If using the default Pages URL for a repo named `Portfolio`:
`baseURL = "https://<github-username>.github.io/Portfolio/"`.
If a user/org Pages repo or custom domain, set accordingly (and add
`static/CNAME` for a custom domain). Update `hugo.toml`.

- [ ] **Step 2: Write `.github/workflows/hugo.yml`**

```yaml
name: Deploy Hugo site to Pages
on:
  push:
    branches: [main]
  workflow_dispatch:
permissions:
  contents: read
  pages: write
  id-token: write
concurrency:
  group: pages
  cancel-in-progress: false
jobs:
  build:
    runs-on: ubuntu-latest
    env:
      HUGO_VERSION: 0.140.0
    steps:
      - uses: actions/checkout@v4
        with: { submodules: recursive, fetch-depth: 0 }
      - name: Install Hugo
        run: |
          wget -O hugo.deb https://github.com/gohugoio/hugo/releases/download/v${HUGO_VERSION}/hugo_extended_${HUGO_VERSION}_linux-amd64.deb
          sudo dpkg -i hugo.deb
      - uses: actions/configure-pages@v5
        id: pages
      - name: Build
        run: hugo --gc --minify --baseURL "${{ steps.pages.outputs.base_url }}/"
      - uses: actions/upload-pages-artifact@v3
        with: { path: ./public }
  deploy:
    needs: build
    runs-on: ubuntu-latest
    environment:
      name: github-pages
      url: ${{ steps.deployment.outputs.page_url }}
    steps:
      - id: deployment
        uses: actions/deploy-pages@v4
```

> Pin `HUGO_VERSION` to the version verified locally in Task 1.

- [ ] **Step 3: Verify workflow is valid locally**

Run: `hugo --gc --minify --baseURL "https://example.github.io/Portfolio/"`
Expected: build succeeds with the production baseURL.

- [ ] **Step 4: Commit**

```bash
git add .github/workflows/hugo.yml hugo.toml
git commit -m "ci: deploy to GitHub Pages"
```

- [ ] **Step 5: Enable Pages (manual, after first push)**

In the GitHub repo: Settings → Pages → Source = "GitHub Actions". Push to
`main`; confirm the workflow runs green and the site is live at the Pages URL.

---

## Done criteria

- `hugo --gc --minify` builds with no errors.
- Home shows the plantplot card; clicking it opens the case study.
- Case study renders all sections with images (SVG + PNG) and a deep-green hero.
- About page renders with a working mailto link.
- GitHub Actions deploys the site to Pages.

## Notes for future projects

To add project #2: `cp -r content/work/plantplot content/work/<slug>`, swap the
`images/`, rewrite `index.md` front matter (incl. `accent`, `thumb`, `sections`),
and it appears on the home grid and gets a "next project" link automatically. No
layout or CSS changes required.
