# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project overview

Single-page personal portfolio/showcase site for Alpay Kucuk ("Albay"), published at `alpaykucuk.com` via GitHub Pages (custom domain set by the `CNAME` file). There is no build system, package manager, bundler, linter, or test framework — the entire site is one static `index.html` plus an `assets/` directory.

## Commands

- No install/build/lint/test commands exist. To preview locally, open `index.html` directly in a browser, or serve the directory with any static file server (e.g. `python3 -m http.server`) so `fetch`-based features (photo grid listing) behave closer to production.
- Deployment is automatic: GitHub Pages serves whatever is pushed to `main`. There is no CI workflow in this repo — pushing to `main` is the deploy.

## Architecture

`index.html` (~5,300 lines) contains all markup, a single large `<style>` block in `<head>`, and all JS as independent inline `<script>` tags near the end of `<body>`. There's no module system or bundler: each behavior is its own self-invoking function, `(function featureName() { ... })();`, scoped to its own DOM ids/classes. Cross-feature communication happens through a small shared `window.__hero` namespace (currently `.poly` and `.drone`), not through imports.

Each content section is a `<section id="...">` containing a `.section-card` that expands into a full-screen `<div class="modal">` via `data-modal-target` — opening/closing all modals is handled centrally by the single `modalController` IIFE (also wires `iframe[data-src]` so embeds only load once a modal opens, and `.subcard-toggle` expand/collapse).

Sections, in page order:
- **`#hero`** — full-viewport intro. `#bg-canvas` is driven by `bgCanvasEngine`, a Verlet-integration rope/network simulation reacting to pointer, touch, and device tilt, with per-browser/mobile perf tuning (iOS Safari, Firefox, desktop Safari detected separately) and an `ENTROPY` meter. Scroll is locked to the hero on load (`heroScrollLock`, uses manual `scrollRestoration`) until a nav link or the WORKS toggle is clicked; `navUpGuard` clamps scroll back up while `nav-locked-down`. `droneEngine` plays a Web Audio ambient chord pad whose filter cutoff/Q are modulated by the canvas sim's entropy/parting/moved stats (read through `window.__hero.poly`); it's exposed back out as `window.__hero.drone`. `heroSaveButton` exports the canvas to a downloadable PNG.
- **`#three-d`** — 3D showreel tiles. `initTileLightbox` builds one reusable overlay carousel; each tile's slides are probed at `assets/gifs/<slug>-NN.webp` (missing files render a "coming soon" placeholder instead of erroring). Touch devices get a tap-to-peek-then-open pattern instead of hover.
- **`#synths`** — two sub-builds inside collapsible `<details class="synth-subcard">`:
  - **ANE** ("Annoying Noise Emulator"): a playable modular synth (`initSynth`) built on the Web Audio API — patchable VCO/LFO/LPF/OUT modules plus fuzz/reverb pedals. Every knob can be bound to a virtual "sensor" source (`light`, `tiltX`, `tiltY`, `pointerX`, `pointerY`, `pointerDist`) via its `<select class="mod-select">`.
  - **ANG** ("Annoying Noise Generator"): real hardware builds shown via Google's `<model-viewer>` web component (loaded from a CDN `<script type="module">` in `<head>`, lazy-rendered until interaction). It points at `models/*.glb`, which do not exist in the repo yet — the poster slot shows a "drop /models/\*.glb" placeholder until added.
- **`#music`** — `initMusic` IIFE: a sample/loop preview list plus an ambient-track playlist player with a canvas visualizer (`#musicViz`), sourced from `assets/Audio/*.wav` (samples/loops) and `assets/Tracks/*.mp3` (full tracks). A "keep playing when closed" checkbox keeps audio going via the floating `#miniPlayer` after the modal closes.
- **`#photography`** — the grid (`#photo-grid`) renders its inline `<img>` tiles first, then tries to refresh from the live GitHub Contents API (`kucukalpay/sayfa`, branch `main`, path `assets/images/pht`) so new uploads to that folder show up with no code change; network failures silently keep the inline fallback. Tiles are classified landscape/portrait from natural image dimensions and reordered so portrait tiles always appear in pairs (keeps the dense 4-col grid from leaving gaps).
- **`#contact`** — static email/social links; social hrefs (`Instagram`, `Vimeo`, `ArtStation`, `SoundCloud`) are currently placeholders (`href="#"`).

Other standalone IIFEs near the end of `<body>`: the Showreel YouTube lightbox (clears `iframe.src` on close to hard-stop playback), reveal-on-scroll (`IntersectionObserver` toggling `.visible` on `.reveal` elements) plus the footer year, and pausing `<model-viewer>` auto-rotate when its card scrolls off-screen.

## Conventions

- New client-side behavior should follow the existing shape: one `(function featureName() { ... })();` IIFE placed with the other scripts at the bottom of `<body>`, querying its own DOM by id/class and bailing early (`if (!el) return;`) rather than assuming elements exist. Add a short comment above the IIFE only when the *why* isn't obvious from the code (existing blocks all do this).
- Theming goes through CSS custom properties on `:root` (`--bg`, `--ink`, `--accent`, `--line`, etc.); `.synth-section` overrides `--line` locally to restore the 1px borders that are `transparent` everywhere else on the page.
- Asset placement: general images in `assets/images/` (`.webp` preferred), photography specifically in `assets/images/pht/` (auto-discovered by the photo grid, no markup change needed), audio samples/loops in `assets/Audio/`, full tracks in `assets/Tracks/`. Adding a track or sample still requires a manual row in the corresponding list inside `#modal-music`; only the photography grid is auto-populated.
