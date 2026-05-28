# Architecture

This document explains how Monogram Studio is put together. Read this before making structural changes.

---

## 1. Top-level constraint: single file

The entire application — markup, styles, fonts (`@font-face` links), React code, and the in-browser Babel compiler — lives in `index.html`. There is no build step, no bundler, no `package.json`. This is a deliberate choice that drives several design decisions:

- All React code is wrapped in `<script type="text/babel">` and transpiled by Babel Standalone in the browser at page load.
- All CSS lives in a single `<style>` block inside `<head>`, or inline inside the `<style>{`}</style>}` template inside the React tree (the bulk of layout CSS).
- Constants (`FONTS`, `PRESETS`, `SHAPES`, `TEXTURES`, `DEFAULT_SETTINGS`) are top-level `const` declarations inside the Babel script.
- Asset files in the repo root (`fonts/`, `favicon.png`, `kofi logo.png`, `preview/`) are referenced by relative path.

**If you are tempted to split the file, don't.** The portability is the feature. The codebase is small enough (~1650 lines) that section comments are sufficient navigation.

---

## 2. File map

```
Monogram Logo Generator/
├── index.html          # The entire application
├── readme.md           # Public-facing intro
├── Architecture.md     # This file
├── Schemas.md          # Data shapes (Settings, FONTS, PRESETS, URL hash, JSON export)
├── Plan.md             # Implementation checklist, testing criteria, roadmap
├── Agents.md           # Guide for downstream agentic coders
├── LICENSE             # MIT
├── favicon.png         # Tab icon
├── preview-image.png   # OG / social preview
├── kofi logo.png       # Ko-fi button asset
├── Poppins-Bold.ttf    # (Legacy — fonts now served from fonts/)
├── fonts/              # Self-hosted woff2 files (one per family)
└── preview/            # Marketing screenshots (preview1..5.png)
```

`fonts/` is loaded via `@font-face` rules at the top of `index.html` (lines ~18–27). One file per font family, all `latin` subset, woff2.

---

## 3. index.html layout

The file has a stable structure. Line numbers shift as you edit; reach for the markers instead.

```
<head>
  <style>                                       # @font-face declarations only
    @font-face { ... }
  </style>
  <script src=".../react.production.min.js">    # React 18 UMD
  <script src=".../react-dom.production.min.js">
  <script src=".../babel.min.js">               # Babel Standalone
</head>
<body>
  <div id="root"></div>
  <script type="text/babel">
    // ── Hooks shim ──
    const { useState, useRef, useEffect, useCallback, useMemo } = React;

    // ── Top-level constants ──
    function useDebounce(...)        // custom hook
    const FONTS = [...]              // Font catalogue (8 families)
    const TEXTURES = [...]
    const PRESETS = [...]            // 6 built-in design presets
    const SHAPES = [...]
    const DEFAULT_SETTINGS = {...}   // Canonical Settings shape

    // ── Pure render functions ──
    function hexToRgb(hex)
    function createDynamicGradient(...)
    function drawTexture(ctx, texture, W, H)
    function getFontString(fontFamily, weight, size)
    function drawMonogram(canvas, settings)        // ← MAIN CANVAS RENDER

    // ── SVG export ──
    function buildShapeSVGPath(shape, cx, cy, R)
    function exportSVG(settings)

    // ── Canvas shape helpers ──
    function clipShape(ctx, shape, cx, cy, R)
    function strokeShape(ctx, shape, cx, cy, R, color, width)
    function buildShapePath(ctx, shape, cx, cy, R)

    // ── Presentational components ──
    function PresetThumb({ preset, active, onClick, fontLoaded })

    // ── URL encoding ──
    function encodeSettings(s)
    function decodeSettings(str)

    // ── Color utility ──
    const hslToHex = (h, s, l) => ...

    // ── Form components ──
    function ColorRow({ label, value, onChange })
    function SliderRow({ label, value, min, max, step, onChange, unit })

    // ── App ──
    function App() {
      // State, refs, history
      // Effects (font load, autosave, canvas redraw, keyboard)
      // Actions (set, undo, redo, applyPreset, downloadPNG, downloadSVG,
      //          shareURL, exportJSON, importJSON, handleRandomize)
      // JSX
      //   <style>{`...all layout CSS...`}</style>   ← bulk of CSS lives here
      //   <div className="app">
      //     <header />
      //     <main className="canvas-area">
      //       <div className="canvas-row">
      //         <div className="canvas-sidebar">      ← presets + actions (desktop)
      //         <div className="canvas-wrap"><canvas /></div>
      //       </div>
      //     </main>
      //     <div className="panel">
      //       <div className="tabs">
      //       <div className="panel-content">
      //         <div className="mobile-presets-wrap"> ← mobile-only presets copy
      //         {tab === "design"    && ...}
      //         {tab === "colors"    && ...}
      //         {tab === "export/import" && ...}
      //         <button>Randomize</button>
      //       </div>
      //     </div>
      //   </div>
    }

    const root = ReactDOM.createRoot(document.getElementById('root'));
    root.render(<App />);
  </script>
</body>
```

---

## 4. Rendering pipeline

The canvas is the source of truth for the visible preview. The render path is:

```
settings change ─┐
                 ├──► setSettings({...prev, [key]: val})
                 ├──► history push
                 ├──► useEffect [settings] ─► localStorage.setItem(...)
                 └──► useEffect [debouncedSettings, fontLoaded] ─► drawMonogram(canvasRef.current, settings)
```

`useDebounce(settings, 0)` is currently a zero-delay debounce — effectively a no-op pass-through. The hook is in place so you can raise the delay later if needed without restructuring the effect dependency.

### `drawMonogram(canvas, settings)`

The single function that owns the render. Draws into a fixed `700 x 700` coordinate space (the canvas element is then CSS-scaled to its container).

Order of operations:

1. **Clear** the canvas.
2. **Background fill** — `createDynamicGradient(...)` produces either a linear gradient (using `bgGradientAngle`) or a radial gradient (centered, scaled by `bgGradientStop`). Filled inside the shape clip.
3. **Texture overlay** — `drawTexture(ctx, texture, W, H)` paints a low-opacity pattern inside the clip (crosshatch / dots / grid / lines / none).
4. **Outer ring** — drawn around the shape edge if `showRing1`.
5. **Inner ring** — drawn inset by `ring1Gap` if `showRing2`.
6. **Big letter** — `getFontString(fontFamily, 900, letterSize)`, optionally with `outlineWidth` stroke and a multi-pass shadow for glow.
7. **Big letter gradient fill** — `createDynamicGradient` on `kTop`/`kBot` clipped to the text path.
8. **Name text** — same treatment with `letterSpacing` between characters, scaled by `nameSize`, vertically offset by `nameOffsetY`.

Glow is implemented as repeated `ctx.shadowBlur` passes scaled by `glowIntensity` (a 0.2–3.0 multiplier). Higher intensity = more passes = brighter halo.

### `exportSVG(settings)` — async

Builds an SVG document **from scratch as a template string** — it does *not* serialize the canvas. SVG and PNG can drift if you change one without the other. Any change to `drawMonogram` that affects visual output must be mirrored in `exportSVG`. See `Agents.md` for the full checklist.

Shape paths for SVG come from `buildShapeSVGPath(shape, cx, cy, R)`; canvas paths come from `buildShapePath(ctx, ...)`. Keep these two in sync.

**Text is rendered as vector `<path>` elements**, not `<text>` elements, so the exported SVG is fully font-independent — it renders identically in any browser or design tool with no font dependency. The conversion happens at export time via `Typr.js` (loaded from CDN in `<head>`):

```
exportSVG(settings)
  ├─► loadParsedFont(family)          # fetch ./fonts/<file>.woff2, Typr.parse() the buffer, cache it
  ├─► textToCombinedPathD(...)         # walk codepoints, get glyph paths, merge into one d string
  │     • each glyph translated by its advance width (in font units)
  │     • result is wrapped in <g transform="translate(...) scale(scale, -scale)">
  │     • single combined path means a single gradient fill spans all glyphs
  └─► assemble final <svg> string with <defs> (gradients, glow filter, clip), shape, rings, paths
```

The function is `async` because the font fetch is async. The caller (`downloadSVG` in `App`) awaits it. If Typr.js fails to load or the font fetch fails, `exportSVG` falls back to `<text>` elements (which look correct only if the SVG is opened in the same directory as the `fonts/` folder — the legacy behavior).

Font cache (`_parsedFontCache`) is module-level and lives for the page lifetime. The first SVG export for a given font pays the fetch + parse cost (~50-200ms); subsequent exports for that font are effectively instant.

The big letter renders at font weight 900 in canvas; `FONT_FILES["Poppins"]` is intentionally pointed at the 900-weight woff2 to match. Other fonts only have one weight file each.

### Coordinate space

- Canvas internal resolution: `700 x 700` (constants `W = H = 700` in `drawMonogram` and `exportSVG`).
- Center: `cx = 350, cy = 350`.
- Outer radius: `R = W/2 - 16 = 334`.
- All offsets in settings (`letterOffsetX/Y`, `nameOffsetY`) are in this 700-unit space, so a value of `100` always means the same proportion of the canvas.

---

## 5. State model

All design state lives in a single `settings` object. See `Schemas.md` for the complete key reference.

```
settings ──► setSettings()         single source of truth
   │
   ├── set(key, val)               update one key, push history, autosave
   ├── applyPreset(preset)         merge preset into settings, push history
   ├── importJSON(file)            replace from disk, push history
   └── handleRandomize()           regenerate large slice, push history
```

### History (undo / redo)

Implemented as two refs (not React state, to avoid re-renders on every push):

- `historyRef.current` — array of `Settings` snapshots, capped at 100 entries.
- `historyIndexRef.current` — pointer into that array.

Every mutation calls a small helper pattern: slice forward history, push new snapshot, advance pointer. `undo` / `redo` move the pointer and call `setSettings(historyRef.current[i])` directly.

`canUndo` and `canRedo` are computed from the refs on every render — they don't trigger re-renders themselves but stay correct because state changes already cause a render.

### Persistence

Two persistence channels:

| Channel | Format | Trigger | Read |
|---|---|---|---|
| `localStorage["monogram-studio-settings"]` | JSON | `useEffect [settings]` | Initial load fallback. |
| URL hash `#config=<base64>` | base64-encoded JSON | Share button | Initial load — **takes priority over localStorage**. |
| `.json` file | pretty-printed JSON | Export JSON button | Import JSON button. |

The initial settings resolver order: URL hash → localStorage → `DEFAULT_SETTINGS`.

`encodeSettings(s)` uses `btoa(JSON.stringify(s))` — keep settings JSON-clean (no `undefined`, no functions). `decodeSettings(str)` is `JSON.parse(atob(str))` with a try/catch returning `null` on failure.

---

## 6. Components

The component tree is shallow:

```
<App>
  <header>...</header>
  <main className="canvas-area">
    <div className="canvas-row">
      <div className="canvas-sidebar">
        Presets list:
          {PRESETS.map(p => <PresetThumb ... />)}
        <canvas-actions>
          PNG / SVG / Share buttons
      <div className="canvas-wrap">
        <canvas ref={canvasRef} />
  <div className="panel">
    <div className="tabs">Design / Colors / Export-Import</div>
    <div className="panel-content">
      <mobile-presets-wrap>
        {PRESETS.map(p => <PresetThumb ... />)}
      Tab content:
        design  → Content, Font, Shape, Typography, Texture, Rings, Glow
        colors  → Background, Big Letter, Name Text, Rings
        export  → Download / Share / Save-Load JSON / Current Settings (JSON view)
      <button>Randomize</button>
```

**`PresetThumb`** — renders a tiny version of the preset on its own 60x60 canvas using a stripped-down render. Memoized via `useEffect` on `[preset, fontLoaded]`. Updates only when fonts are ready.

**`ColorRow`** — label + native color input + read-only hex string.

**`SliderRow`** — label + range input + numeric readout with unit. Generates an `id` from the label slug for `htmlFor`.

---

## 7. Layout system

The CSS lives in two places:

1. **Global font-face block** at the top of `<head>` — fonts only.
2. **Component-scoped style block** inside `<App>` — everything else. This is just a `<style>` element rendered inside the React tree.

### Desktop (`min-width: 769px`)

The top-level `.app` is a CSS Grid:

```css
.app {
  display: grid;
  grid-template-columns: 1fr 380px;     /* canvas area | panel */
  grid-template-rows: 85px 1fr;          /* header | content */
  height: 100vh;
}
```

Header spans column 1 row 1. Canvas area sits in column 1 row 2. Panel spans column 2 rows 1–2 (full height on the right).

Inside `.canvas-area`, the `.canvas-row` is a horizontal flex:

```
[ .canvas-sidebar 180px ] - gap 40 - [ .canvas-wrap (preview, max 600) ]
                                                  flex grows
```

`.canvas-sidebar` is itself a vertical flex with `gap: 22px` separating the presets list from the action buttons (PNG / SVG / Share).

### Mobile (`@media max-width: 768px`)

`.app` collapses to a flex column: `header → canvas-area → panel`. `.canvas-area` becomes a fixed-height region (preview + actions); `.panel` flexes to fill the rest and scrolls.

The clever piece: **`.canvas-sidebar` uses `display: contents`** on mobile.

```css
@media (max-width: 768px) {
  .canvas-sidebar { display: contents; }
  .canvas-sidebar > .presets-label,
  .canvas-sidebar > .presets { display: none; }
}
```

`display: contents` makes the sidebar's own box disappear from layout while its children participate in the parent's flex/grid. The net effect: `.canvas-actions` (a child of `.canvas-sidebar`) "promotes" to become a direct child of `.canvas-row`, where the mobile `flex-direction: column` puts it just below the preview. The desktop preset list inside the sidebar is hidden with the selector above.

Mobile presets render from a **second copy** of the preset map, placed at the top of `.panel-content` so they scroll alongside the parameter controls. On desktop, that mobile copy is hidden via `.mobile-presets-wrap { display: none }`. On the export/import tab, the mobile presets are conditionally not rendered (`{tab !== "export/import" && (...)}`).

### Order on mobile

Inside `.canvas-row` on mobile:

| Element | `order` | Visual position |
|---|---|---|
| `.canvas-wrap` | 1 | top (preview, 200x200 centered) |
| `.canvas-actions` | 2 | below preview (horizontal row, PNG/SVG/Share, flex: 1 each) |

Presets-related children of `.canvas-sidebar` are `display: none` at this breakpoint.

### Small phones (`@media max-width: 380px`)

Preview shrinks to `180x180`, button padding tightens.

---

## 8. Font loading

Fonts are self-hosted in `fonts/` (woff2, latin subset). They are declared via `@font-face` in `<head>` and consumed by name (`'Poppins'`, `'Playfair Display'`, etc.).

Canvas does not auto-trigger font fetch when you call `ctx.font = '900 100px Poppins'` — the browser may fall back to system fonts until the woff2 is parsed. To prevent flashing the wrong font in the preview and preset thumbnails:

```js
useEffect(() => {
  const fontsToLoad = FONTS.map(f => document.fonts.load(`${f.weight} 10px '${f.family}'`));
  document.fonts.ready.then(() => {
    Promise.all(fontsToLoad).then(() => setFontLoaded(true)).catch(() => setFontLoaded(true));
  });
}, []);
```

`fontLoaded` gates both the main canvas redraw effect and each `PresetThumb` redraw. The catch branch still sets `fontLoaded = true` so the app degrades to system fonts rather than hanging forever.

---

## 9. Randomize generator

`handleRandomize` is the largest single block in `App`. It does not just pick random hex colors — it builds a **cohesive palette** using HSL color theory.

Algorithm overview:

1. Pick a base hue `0..360`.
2. Pick a theme roll: 45% analogous, 35% complementary, 20% monochromatic.
3. Define `getThemeHue()` based on theme — returns hues consistent with the roll.
4. Generate background (mid-to-dark luminosity range), accent colors (high lightness), and name text colors (rich saturation, moderate lightness).
5. Roll fonts/shapes/textures uniformly from their lists.
6. Roll structural toggles (ring1 85% on, ring2 70% on, glow 60% on).
7. Build a complete `randomizedMatrix` matching `DEFAULT_SETTINGS` keys.
8. Merge into current `settings` and commit a history step.

This is the function to extend if you want new "themes" or constraints on what gets randomized. Color harmony scaffolding sits in the first ~30 lines.

---

## 10. Accessibility notes

- All sliders use `<label htmlFor>` linkage.
- All buttons have `aria-label` when their visible text is icon-only.
- Tab buttons set `role="tab"` and `aria-selected`.
- Toggle buttons set `aria-pressed`.
- The share toast uses `role="status"` with `aria-live="polite"`.
- The import error uses `role="alert"`.
- `display: contents` on `.canvas-sidebar` is supported with correct semantics in current Chrome, Firefox, and Safari. The sidebar div carries no role or label, so semantic loss is not a concern.

---

## 11. Known gotchas

- **SVG export drift** — `exportSVG` is a separate template; it does not mirror `drawMonogram` automatically. Always cross-check both outputs when changing render behavior.
- **Canvas internal resolution is fixed at 700x700.** CSS scales it. Don't change the internal resolution without re-tuning every default value in `DEFAULT_SETTINGS`.
- **History pushes are scattered** — `set`, `applyPreset`, `importJSON`, and `handleRandomize` each implement the "slice forward + push + advance" pattern inline. If you add a new state mutator, you must follow the same pattern or undo/redo will get out of sync.
- **`useDebounce` delay is 0.** It is a placeholder. Increase it if a future feature needs to throttle expensive redraws.
- **localStorage and URL hash can desync.** URL hash wins on initial load. Sharing a link to someone whose localStorage holds different settings will overwrite their working state on visit.
