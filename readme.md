# Monogram Studio

**Live Demo:** [bankenichi.github.io/Monogram-Logo-Generator](https://bankenichi.github.io/Monogram-Logo-Generator/)

---

![Monogram Studio Preview](/preview/preview4.png)

A sleek, completely free, browser-based generator for creating high-quality monograms, logos, and custom tokens.

No sign-ups, no premium paywalls, no watermarks, no server-side processing. Everything runs purely in your browser. Open the page, design, hit download. Nothing leaves your machine.

---

## Perfect For

- **Tabletop RPGs & VTTs** — faction crests, player tokens, campaign logos for D&D, sci-fi games, and more.
- **Indie Developers** — polished placeholder logos or app icons in seconds.
- **Personal Branding** — fast avatars and profile pictures for Discord, GitHub, or social media.
- **Streamers & Creators** — channel marks, sub-badges, and emote stand-ins.

---

## Features

**Output**

- One-click **PNG** download at 700×700, 1400×1400, or 2800×2800 — native resolution render, no upscaling.
- **SVG** export — fully vector, scales infinitely, ideal for print and logos. Font-independent (converts text to paths at export time).
- **Shareable URLs** — your entire design serialized into the URL hash; send the link, recipient sees the exact same design. Warns if the URL exceeds browser limits and offers JSON export as fallback.
- **JSON save/load** — back up or hand off a design as a portable `.json` file.
- **Preset bundle export/import** — export all 10 preset slots as a `.json` bundle for backup or sharing.

**Design controls**

- **Twenty-four curated fonts** (Poppins, Playfair SC, Bebas Neue, Cinzel, Dancing Script, Space Mono, Cormorant, Abril Fatface, Cinzel Decorative, UnifrakturCook, Pirata One, Audiowide, Black Ops One, Russo One, Caveat, Special Elite, Pacifico, Bungee, Monoton, Bangers, Rye, Silkscreen, Righteous, Merriweather) — dropdown picker with per-font live preview.
- **Nine shape silhouettes** (none, circle, square, rounded-square, diamond, hexagon, triangle, pentagram, hexagram).
- **Five texture overlays** (none, crosshatch, dots, grid, lines).
- **Gradient or solid fill** per color slot (background, big letter, name text) — toggle between gradient and flat color modes.
- Independent gradient angle and stop position per channel.
- Inner + outer rings with independent width, color, and gap.
- **Glow** with toggle, color, and intensity (big letter and name text).
- **Drop shadow** with toggle, offset, blur, color, and opacity (big letter and name text).
- **Emboss / bevel** with toggle, depth, angle, highlight, shadow color, and opacity (big letter).
- **Letter transformations** — independent scale X/Y, skew X/Y, and rotation on the big letter.
- **Blend modes + opacity** on the big letter (multiply, screen, overlay, lighten, etc.). The blend composites the letter against the **background only** — glow, drop shadow, and emboss are independent layers and are never tinted by the blend — and canvas/PNG and SVG output match.
- **Layer stack** — visibility toggles and reorder for Background, Rings, Big Letter, and Name. Effects render in a consistent order (shadow → glow → emboss → fill) across both canvas and SVG.
- Outline stroke around the shape perimeter.
- Per-element fine-position offsets and outline strokes.

**Color picker**

- HSV color wheel (hue ring + saturation/value square) with live hex and R/G/B inputs.
- Click-to-pick or drag-on-wheel for precise color selection.

**Workflow**

- **Six built-in presets** — Royal Gold, Midnight Aurora, Crimson Ember, Arctic Silver, Forest Obsidian, Neon Noir (in slots 0–5 of the 10-slot preset system).
- **10-slot preset system** — save, clear, reset to shipped, and export/import bundles.
- **Randomize** generator with per-category lock toggles (content, font, shape, texture, bgColor, letterColor, nameColor, outline, rings, typography, effects) plus a one-click **Reset Locks** button.
- **Undo / redo** with full keyboard shortcuts (Ctrl/Cmd+Z, Ctrl/Cmd+Y, Ctrl/Cmd+Shift+Z) and up to 100 steps of history.
- **Reset to default** — one-click restore to the default design, with undo support.
- **Auto-save** — every change persists to `localStorage` so closing the tab doesn't lose work (0ms debounce — writes on every render cycle).
- Responsive layout — large preview + sidebar on desktop, stacked layout with full-width preview on mobile.
- **Floating action panel** on canvas (PNG / SVG / Share) with tap-to-hide on mobile.
- **Collapsible accordion sections** for all control groups (persisted across reloads).
- **Dedicated Randomizer tab** with per-category lock toggles and a Reset Locks button.

---

## Tech Stack

Built as a portable Single Page Application contained entirely in one HTML file. No build step, no `node_modules`, no bundler.

- **React 18** — loaded via UMD CDN.
- **Babel Standalone** — in-browser JSX compilation.
- **HTML5 Canvas API** — for live preview and PNG rasterization.
- **Hand-rolled SVG** — for vector export.
- **Typr.js** (pinned to commit `02c1210`) — in-browser font parsing for SVG text-to-path conversion.
- **Vanilla CSS3** — CSS Grid for desktop layout, Flexbox + `display: contents` for the mobile reflow.
- **Self-hosted woff2 fonts** (24 families, latin subset) — no Google Fonts dependency, works fully offline.

---

## Local Setup

Zero backend, zero build steps.

```bash
git clone https://github.com/bankenichi/Monogram-Logo-Generator.git
cd Monogram-Logo-Generator
```

Then open `index.html` in any modern browser. That's it.

If you'd rather serve it (recommended so the browser can read the local `fonts/` directory cleanly):

```bash
python -m http.server 8000
# or
npx serve .
```

Then visit `http://localhost:8000`.

---

## Keyboard Shortcuts

| Action | Shortcut |
|---|---|
| Undo | Ctrl/Cmd + Z |
| Redo | Ctrl/Cmd + Y *or* Ctrl/Cmd + Shift + Z |

---

## Browser Support

Tested on current Chrome, Firefox, Safari, and Edge. The mobile layout uses `display: contents`, which is supported in all modern browsers (2019+). Requires Canvas and ES2018 syntax — no IE11.

---

## Project Documentation

Deeper docs live alongside this README:

| File | Purpose |
|---|---|
| `Architecture.md` | How the app is built — rendering pipeline, state model, layout system. |
| `Schemas.md` | The `Settings` shape, font catalogue, preset slot format, URL hash format, JSON export format. |
| `Plan.md` | Implementation checklist, testing criteria, and forward roadmap. |
| `Agents.md` | Guide for agentic coders contributing to the project. |
| `Sprint-Plan.md` | Current sprint plan with feature specs and acceptance criteria. |

---

## Contributing

Issues and PRs welcome. Before opening a PR:

1. Read `Architecture.md` to understand the rendering pipeline.
2. Read `Agents.md` for naming conventions and common pitfalls.
3. Verify your change against the testing criteria in `Plan.md`.

The codebase is intentionally a single file — keep that constraint when adding features.

---

## License

MIT. See `LICENSE`.

---

<div align="center">
  <a href="https://ko-fi.com/bankenichi" target="_blank">
    <img src="https://raw.githubusercontent.com/bankenichi/Monogram-Logo-Generator/main/kofi%20logo.png" alt="Support me on Ko-fi" height="120">
  </a>
</div>
