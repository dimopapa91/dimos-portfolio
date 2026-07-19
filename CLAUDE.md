# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Local Development

```bash
python3 -m http.server 8080
# then open http://localhost:8080
```

No build step, no framework, no npm. Everything is static.

## Architecture

**Everything lives in a single file: `index.html`.**

The file is structured in this order:
1. `<head>` — Google Fonts, CSS vars, all styles (~660 lines)
2. `<body>` — loader, canvas, mobile terrain SVG, grain, cursor, header, project list, overlays, footer
3. `<script>` — `revealSite()` runs first (no dependencies); then Three.js + GSAP CDN scripts; then all JS logic

### CSS Architecture

CSS vars at the top of `<style>`:
- `--orange: #d05c18`, `--orange-glow: #ff8c38`, `--bg: #060100`
- All entrance animations are gated by `body.site-loaded` — nothing animates in until `revealSite()` adds that class

### Background Terrain

**Desktop (WebGL):** Three.js plane mesh with a GLSL simplex-noise shader (`vertSrc`/`fragSrc`). Geometry: 260×260 units, 150×150 segments (100×100 on mobile). The `u_time` uniform drives animation. Camera slowly orbits.

**Mobile / WebGL fallback:** `#mobile-terrain` — an inline SVG with two `<path>` layers using cubic bezier curves for smooth mountain ridges. Shown via CSS on `max-width: 768px`; also shown if WebGL throws (the `catch` block sets `canvas.display='none'` and forces the SVG visible).

### Overlay System

All panels (project, music, about, education, contact) share the `.overlay` class: `position:fixed`, starts at `transform: translateX(100%)`, transitions to `translateX(0)` when `.active` is added. Closed by the `✕` button or pressing Escape.

Opening a project calls `openProject(key)` which looks up `projectData[key]`, builds HTML, and injects it into `#project-content`.

### projectData

JS object (keys: `waveline`, `blakenor`, `education`). Each entry has:
- `name`, `year`, `category`, `description`
- `tech` — array of tech names (must exist in `TECH_ICONS` or render as dot)
- `links` — array of `{ label, href, icon }` where `icon` keys into `LINK_ICONS`
- `color` — used as overlay accent / background tint
- `images` + `imageLabels` — for projects with screenshot galleries (renders as `.po-screenshots` grid)
- `artwork` — for music projects (renders as `.po-artwork-wrap`)

### Tech Icons

```js
const DI = 'https://cdn.jsdelivr.net/gh/devicons/devicon@v2.16.0/icons/';
const SI = 'https://cdn.simpleicons.org/';
```

`TECH_ICONS` maps tech names → `{ src, inv? }`. Set `inv: true` for dark SVG icons that need `filter: brightness(0) invert(1)` on the dark background. Set the value to `null` to render a dot indicator instead of an icon.

`techPill(name)` returns a `<span class="tech-pill">` with either an `<img>` or a `<span class="tp-dot">`.

`ABOUT_SKILLS` is the ordered list rendered into `#ao-skills-list` in the About overlay.

`LINK_ICONS` maps icon keys → CDN source for project/music link buttons.

### Screenshot Gallery Layout

`.po-screenshots` is a 2-column CSS grid. The first `.po-shot` spans both columns (`grid-column: 1 / -1`), sized 16:9. Remaining shots are 4:3 with slight brightness/saturation reduction. Labels use an absolutely-positioned gradient overlay.

### revealSite()

Idempotent (guarded by `_done` flag). Hides `#loader`, adds `body.site-loaded`. Called by `setTimeout(revealSite, 1200)` as a hard fallback — also called early on `DOMContentLoaded` if desired.

## Deployment

GitHub Pages with a custom domain. `CNAME` file contains `dimospapageorgiou.com`. Push to `main` branch to deploy.

**Git pushes from this machine require terminal authentication — run from your own terminal:**
```bash
cd ~/dimos-portfolio && git add index.html && git commit -m "your message" && git push origin main
```

## Assets

```
assets/
  Dimos-Papageorgiou-CV.pdf        # downloadable from nav
  images/
    waveline-1.jpg                  # dashboard screenshot (hero)
    waveline-4.jpg                  # taste profile screenshot
    waveline-5.jpg                  # music news screenshot
    colours-ep.jpg                  # Blakenor album artwork
    education/                      # cert/degree images
```
