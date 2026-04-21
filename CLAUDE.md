# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this repo is

Two things in one:

1. **`lywedo/lywedo` — the special GitHub profile repo.** `README.md` renders on <https://github.com/lywedo> and is the primary artifact. It is *not* ordinary project documentation — it's a stylized profile page assembled from external image services (capsule-render, skillicons.dev, shields.io, github-readme-stats, github-readme-streak-stats, github-readme-activity-graph, github-profile-trophy, readme-typing-svg, komarev view counter). Changes to README alter the user's public GitHub profile.
2. **Static personal portal site** (`index.html` + `style.css` + `script.js`) deployed to Azure Static Web Apps at `https://green-sky-08c907b00.4.azurestaticapps.net/`.

Both share a red & black cyberpunk visual identity (accents `#e11d48`, `#dc2626`, `#ff2d55`).

## Development

There is **no build step, bundler, framework, test suite, or linter**. `package.json` has no scripts — its only dep (`https-proxy-agent`) is a leftover tooling artifact unrelated to the site; do not treat this as a Node project.

- **Run locally:** serve the directory statically, e.g. `python3 -m http.server 8000` then open `http://localhost:8000/`. Opening `index.html` via `file://` also works.
- **Deploy:** pushes to `main` flow through Azure Static Web Apps (no workflow file in-repo — the deploy is configured on the Azure side).
- **Verify changes:** open the page in a browser and exercise the feature. There is no automated test that will catch regressions.

## Architecture

Three files, three concerns, no cross-file abstractions:

- **`index.html`** — single page with sections `#hero`, `#about`, `#skills`, `#projects`, `#contact`. Contains a `<canvas id="particles">` for the background and structural hooks (`.glass-card`, `[data-tilt]`, `[data-target]`, `.reveal`) that `script.js` binds to.
- **`style.css`** — theme tokens in `:root` (colors, gradients, radii, fonts `Inter` + `JetBrains Mono`, `--nav-h`). Anything visual (colors, spacing, animations, scanline overlay, glow) should be edited via these variables, not by hardcoding values in components.
- **`script.js`** — one IIFE (`(function(){ 'use strict'; ... })()`) containing all behavior in sequential blocks:
  - Particle canvas with mouse repulsion + proximity line connections (`PARTICLE_COUNT`, `CONNECTION_DIST`, `MOUSE_DIST` are the knobs).
  - Typing-effect loop over `phrases[]` targeting `#typed`.
  - Navbar scroll-state toggle (`.scrolled` on `#navbar` past 50px).
  - Mobile hamburger toggle on `#navToggle` / `#mobileMenu`.
  - Scroll reveal via `IntersectionObserver` — elements listed in `revealEls` get `.reveal` then `.visible`.
  - Stat counter animation driven by `data-target` attributes.
  - Smooth-scroll for `a[href^="#"]`.
  - 3D tilt on `[data-tilt]` cards.

To wire up a new animated element, add the selector to `revealEls` (for scroll-in), add `data-tilt` (for hover tilt), or add `data-target="N"` to a `.stat-number` (for counter). No module system — extend the IIFE.

## Conventions worth preserving

- Keep the portal zero-dependency. Resist adding a framework, bundler, or npm-based tooling unless explicitly asked.
- Both README and portal share the red/black cyberpunk palette — if you change one, consider whether the other should track.
- README widgets use URL query params for theming (`theme=radical`, `bg_color=050505`, `color=fb7185`, `line=e11d48`, etc.). Edit URLs, not component code — these are third-party image services.
