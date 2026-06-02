# Architecture

This document explains how Monogram Studio is put together. Read this before making structural changes.

**Checkpoint:** `index.html.inc1` is a backup snapshot taken after Waves 1+2 were completed. Refer to it to compare against pre-Wave 3 state.

---

## 1. Top-level constraint: single file

The entire application — markup, styles, fonts (`@font-face` links), React code, and the in-browser Babel compiler — lives in `index.html`. There is no build step, no bundler, no `package.json`. This is a deliberate choice that drives several design decisions:

- All React code is wrapped in `<script type="text/babel">` and transpiled by Babel Standalone in the browser at page load.
- All CSS lives in a single `<style>` block inside `<head>`, or inline inside the `<style>{`}</style>}` template inside the React tree (the bulk of layout CSS).
- Constants (`FONTS`, `PRESETS`, `SHAPES`, `TEXTURES`, `DEFAULT_SETTINGS`) are top-level `const` declarations inside the Babel script.
- Asset files in the repo root (`fonts/`, `favicon.png`, `kofi logo.png`, `preview/`) are referenced by relative path.

**If you are tempted to split the file, don't.** The portability is the feature. The codebase is small enough (~1860 lines) that section comments are sufficient navigation.

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
    const { useState, useRef, useEffect, useCallback } = React;

    // ── Top-level constants ──
    function useDebounce(...)        // custom hook
    const FONTS = [...]              // Font catalogue (24 families)
    const TEXTURES = [...]
    const SHIPPED_PRESETS = [...]    // 6 built-in design presets (factory defaults for slots 0–5)
    const SHAPES = [...]
    const DEFAULT_SETTINGS = {...}   // Canonical Settings shape (includes bgMode, kMode, nameMode)
    const TOTAL_PRESET_SLOTS = 10
    const SHIPPED_SLOT_COUNT = 6
    const PRESET_SLOTS_KEY = "monogram-studio-preset-slots"

    // ── Color utility ──
    function rgbToHex(r, g, b)
    function hexToHsv(hex)           // returns { h, s, v }
    function hsvToHex(h, s, v)       // returns #rrggbb

    // ── Pure render functions ──
    function hexToRgb(hex)
    function createDynamicGradient(...)
    function drawTexture(ctx, texture, W, H)
    function getFontString(fontFamily, weight, size)
    function drawMonogram(canvas, settings, exportSize)  // ← MAIN CANVAS RENDER

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
    function ColorRow({ label, value, onChange })          // legacy native color picker
    function SliderRow({ label, value, min, max, step, onChange, unit })
    function ColorPickerRow({ label, value, onChange })    // HSV wheel popover (swatch + HEX/R/G/B)
    function ColorWheel({ hsv, onChange, size })           // 200×200 canvas wheel
    function Section({ title, sectionKey, defaultOpen, children })  // collapsible accordion

    // ── App ──
    function App() {
      // State, refs, history
      // presetSlots state (10-slot store, persisted)
      // Effects (font load, autosave, canvas redraw, keyboard)
      // Actions (set, undo, redo, applyPreset, downloadPNG, downloadSVG,
      //          shareURL, exportJSON, importJSON, handleRandomize,
      //          handleSaveToSlot, handleClearSlot, handleResetSlotToShipped,
      //          commitSaveToSlot, exportPresets, importPresets)
      // JSX
      //   <style>{`...all layout CSS...`}</style>   ← bulk of CSS lives here
      //   <div className="app">
      //     <header />
      //     <main className="canvas-area">
      //       <div className="canvas-row">
      //         <div className="canvas-sidebar">      ← presets (desktop, collapsible)
      //         <div className="canvas-wrap">          ← preview + floating overlay
      //           <canvas />
      //           <div className="canvas-action-overlay"> ← PNG/SVG/Share
      //         </div>
      //       </div>
      //     </main>
      //     <div className="panel">
      //       <div className="tabs">                   ← includes "randomizer" tab
      //       <div className="panel-content">
      //         <Section>Presets</Section>             ← mobile-only collapsible presets
      //         {tab === "design"    && ...}
      //         {tab === "colors"    && ...}
      //         {tab === "randomizer" && ...}
      //         {tab === "export/import" && ...}
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
                  ├──► useEffect [settings] ─► localStorage.setItem(...)  (writes on every render via 0ms debounce)
                 └──► useEffect [debouncedSettings, fontLoaded] ─► drawMonogram(canvasRef.current, settings)
```

`useDebounce(settings, 0)` drives both canvas redraws and localStorage writes. The 0ms debounce defers execution to the next microtask but does not introduce a perceptible delay — canvas updates feel instant while writes stay coalesced per render cycle.

### `drawMonogram(canvas, settings, exportSize)`

The single function that owns the render. Draws into a fixed `700 x 700` coordinate space. When `exportSize` is provided, the canvas is set to that resolution and `ctx.scale(size/700, size/700)` is applied so all coordinates stay in the 700-unit space while rendering natively at the target resolution.

Order of operations:

1. **Clear** the canvas.
2. **Background fill** — branches on `bgMode`: when `"solid"`, fills with `bgTop`; when `"gradient"`, calls `createDynamicGradient(...)`. Filled inside the shape clip. Skipped when `shape === "none"`.
3. **Texture overlay** — `drawTexture(ctx, texture, W, H)` paints a low-opacity pattern inside the clip (crosshatch / dots / grid / lines / none). Skipped when `shape === "none"`.
4. **Outer ring** — drawn around the shape edge if `showRing1`. Skipped when `shape === "none"`.
5. **Inner ring** — drawn inset by `ring1Gap` if `showRing2`. Skipped when `shape === "none"`.
6. **Big letter — drop shadow** — if `effects.shadow.enabled`, draws the letter with tinted `shadowBlur`/`shadowOffset` before the glow and fill so the shadow sits behind everything.
7. **Big letter — glow** — repeated `ctx.shadowBlur` passes scaled by `effects.glow.intensity` (0.2–3.0 multiplier). Reads color from `effects.glow.color`.
8. **Big letter — fill** — branches on `kMode`: when `"solid"`, fills with `kTop`; when `"gradient"`, applies `createDynamicGradient` on `kTop`/`kBot` clipped to the text path.
9. **Name text — shadow** — if `effects.nameShadow.enabled`, draws a tinted offset pass behind the name.
10. **Name text — glow** — if `effects.nameGlow.enabled`, renders a halo pass behind the name.
11. **Name text — fill** — same treatment as big-letter fill with `letterSpacing` between characters, scaled by `nameSize`, vertically offset by `nameOffsetY`. Fill branches on `nameMode`.

Big-letter effects stacking order (bottom to top): shadow → glow → fill. Name effects stacking order: shadow → glow → fill. Both canvas and SVG follow these orders.

### `exportSVG(settings)` — async

Builds an SVG document **from scratch as a template string** — it does *not* serialize the canvas. SVG and PNG can drift if you change one without the other. Any change to `drawMonogram` that affects visual output must be mirrored in `exportSVG`. See `Agents.md` for the full checklist.

Shape paths for SVG come from `buildShapeSVGPath(shape, cx, cy, R)`; canvas paths come from `buildShapePath(ctx, ...)`. Keep these two in sync.

**Mode branching:** Each color channel (`bgMode`, `kMode`, `nameMode`) branches the fill attribute — `"solid"` uses the hex color directly, `"gradient"` references a `<linearGradient>`/`<radialGradient>` in `<defs>`. Gradient `<defs>` elements are only generated when the corresponding mode is `"gradient"` (avoids unused defs in solid mode).

**Text is rendered as vector `<path>` elements**, not `<text>` elements, so the exported SVG is fully font-independent — it renders identically in any browser or design tool with no font dependency. The conversion happens at export time via `Typr.js` (loaded from CDN in `<head>`):

```
exportSVG(settings)
  ├─► loadParsedFont(family)          # fetch ./fonts/<file>.woff2, Typr.parse() the buffer, cache it
  ├─► textToCombinedPathD(...)         # walk codepoints, get glyph paths, merge into one d string
  │     • each glyph translated by its advance width (in font units)
  │     • scale+translate transform baked directly into path coordinates (no <g> wrapper)
  │     • produces root SVG coordinates so userSpaceOnUse gradients align correctly
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
   ├── resetToDefault()            restore DEFAULT_SETTINGS, push history
   ├── importJSON(file)            replace from disk, push history
   └── handleRandomize()           regenerate large slice, push history
```

### History (undo / redo)

Implemented as two refs (not React state, to avoid re-renders on every push):

- `historyRef.current` — array of `Settings` snapshots, capped at 100 entries.
- `historyIndexRef.current` — pointer into that array.

Every mutation calls a small helper pattern: slice forward history, push new snapshot, advance pointer. `undo` / `redo` move the pointer and call `setSettings(historyRef.current[i])` directly.

`canUndo` and `canRedo` are computed from the refs on every render — they don't trigger re-renders themselves but stay correct because state changes already cause a render.

### Settings migration (`migrateSettings`)

Every settings object entering the app from an external source (URL hash, localStorage, JSON file import) passes through `migrateSettings(s)` before use. This function fills in any keys added since the data was saved, ensuring old shared URLs and exported JSON files continue to load without error.

Current migrations:

| ID | Wave | What it handles |
|---|---|---|
| M1 | F12 | Defaults `bgMode`, `kMode`, `nameMode` to `"gradient"` when absent. |
| M2 | F14 | Initialises `effects.glow`, `effects.shadow`, `effects.nameGlow`, `effects.nameShadow` sub-objects with safe defaults when absent. |

See `Schemas.md §6b` for the full migration catalogue.

### Persistence

Three persistence channels:

| Channel | Format | Trigger | Read |
|---|---|---|---|
| `localStorage["monogram-studio-settings"]` | JSON | `useEffect [debouncedSettings]` — **0ms debounce** (writes every render) | Initial load fallback. |
| `localStorage["monogram-studio-preset-slots"]` | JSON | `persistSlots()` — on any preset slot change | `getOrInitSlots()` on mount. |
| URL hash `#config=<base64>` | base64-encoded JSON | Share button | Initial load — **takes priority over localStorage**. |
| `.json` file | pretty-printed JSON | Export JSON button | Import JSON button. |

The initial settings resolver order: URL hash → localStorage → `DEFAULT_SETTINGS`.

`encodeSettings(s)` uses `btoa(unescape(encodeURIComponent(JSON.stringify(s))))` — handles non-ASCII in settings. `decodeSettings(str)` reverses the encoding with try/catch returning `null` on failure.

---

## 6. Components

The component tree is shallow:

```
<App>
  <header>...</header>
  <main className="canvas-area">
    <div className="canvas-row">
      <div className="canvas-sidebar">
        <Section title="Presets">      ← collapsible, collapsed by default
          {presetSlots.map(s => <PresetThumb ... />)}
      <div className="canvas-wrap">
        <canvas ref={canvasRef} />
        <canvas-action-overlay>        ← PNG / SVG / Share (floating)
  <div className="panel">
    <div className="tabs">Design / Colors / Effects / Randomize / Export-Import</div>
    <!-- tab keys: design, colors, effects, randomizer, export/import.
         Display labels are mapped (Randomize; "Export / Import" wraps to two
         lines via .tab-multiline). .tab uses flex:1 1 0 / min-width:0 / nowrap. -->
    <div className="panel-content">
      <Section title="Presets">        ← mobile-only collapsible presets (present at top of EVERY tab)
        {presetSlots.map(s => <PresetThumb ... />)}
      Tab content:
        design  → Content, Font (dropdown), Typography, Shape, Texture, Rings
        colors  → Background (solid/gradient toggle), Big Letter (solid/gradient toggle),
                   Name Text (solid/gradient toggle), Rings
        effects → Glow, Drop Shadow, Name Glow, Name Shadow
        randomizer → Per-category lock toggles + Reset Locks + Randomize button
        export  → Manage Presets, Download, Share, Save/Load JSON, Current Settings (JSON view)
```

**`PresetThumb`** — renders a tiny version of the preset on its own 60x60 canvas using a stripped-down render. Memoized via `useEffect` on `[preset, fontLoaded]`. Updates only when fonts are ready.

**`ColorRow`** — label + native color input + read-only hex string. Legacy; replaced in UI by `ColorPickerRow`.

**`ColorPickerRow`** — swatch button showing the current hex color, opens a popover containing a `ColorWheel` (200×200 HSV canvas) and HEX/R/G/B text inputs. Closes on click-outside or ESC.

**`ColorWheel`** — HSV color wheel rendered on a canvas. Hue ring (outer ring, draggable) + saturation/value square (inner square, draggable). Emits `hsvToHex` results.

**`SliderRow`** — label + range input + numeric readout with unit. Generates an `id` from the label slug for `htmlFor`.

**`Section`** — collapsible accordion panel. Props: `title` (displayed), `sectionKey` (persisted via `useUIState`), `defaultOpen` (initial state). Default is collapsed (`defaultOpen = false`). Chevron arrows are yellow/bold.

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

`.canvas-sidebar` is a vertical flex containing a collapsible `<Section title="Presets">` (collapsed by default). The floating action overlay (PNG / SVG / Share) is positioned absolutely inside `.canvas-wrap` at top-right — semi-transparent by default, darkens on hover.

### Mobile (`@media max-width: 768px`)

`.app` collapses to a flex column: `header → canvas-area → panel`. `.canvas-area` becomes a fixed-height region; `.panel` flexes to fill the rest and scrolls.

The clever piece: **`.canvas-sidebar` uses `display: contents`** on mobile.

```css
@media (max-width: 768px) {
  .canvas-sidebar { display: contents; }
  .canvas-sidebar > .presets { display: none; }
}
```

`display: contents` makes the sidebar's own box disappear from layout. The desktop presets inside are hidden. The floating overlay remains positioned inside `.canvas-wrap` — on mobile it switches to a horizontal row and is always slightly opaque (no hover state needed). Tapping the canvas toggles overlay visibility.

Mobile presets render from a **second copy** of the preset map inside a `<Section title="Presets">` at the top of `.panel-content`, sharing the same `sectionKey="presets"` as the desktop copy so their collapse states are synced.

### Order on mobile

Inside `.canvas-row` on mobile:

| Element | `order` | Visual position |
|---|---|---|
| `.canvas-wrap` | 1 | top (preview, 200x200 centered, with floating overlay) |

The old `.canvas-actions` row is gone — actions live on the canvas overlay now.

### Small phones (`@media max-width: 380px`)

Preview shrinks to `180x180`.

### Wide / low-verticality phones (`@media max-width: 768px and min-aspect-ratio: 0.55`)

Targets short-but-wide viewports where the stacked square preview consumed nearly the whole screen. The preview is capped at `max-height: 42dvh` / `max-width: 62%` so the controls panel keeps usable room.

All mobile widths get a slimmer tab bar (`.tabs { height: 42px }`), tighter panel padding, and the floating overlay at top-right of the preview.

### Collapsible sections (accordion)

Every control group in the panel (Content, Font, Typography, etc.) is wrapped in a `<Section>` component that can be collapsed and expanded by clicking its header. Open/closed state persists per-section across page reloads.

- **Component:** `Section({ title, sectionKey, defaultOpen, children })` — renders a header button with a chevron indicator (`▸` / `▼`) and a hidden/shown body div.
- **Persistence:** Uses the `useUIState()` hook, which reads/writes `monogram-studio-ui-state` in localStorage. UI state is stored separately from design settings — reset-to-default, JSON import, and shared URLs do not affect which sections are open.
- **Section key convention:** Each section gets a stable `sectionKey` string (e.g. `"content"`, `"font"`, `"effects_glow"`, `"presets"`). Desktop and mobile copies of the Presets section share the same key so their collapse states stay in sync.
- **Default state:** Most sections default to `open`. Presets, Export Preview (JSON debug view), Drop Shadow, Emboss, and Name Glow/Shadow default to `closed`.

### Floating action overlay

A semi-transparent panel overlaid on the canvas providing the three primary export actions: PNG download, SVG download, and Share. Replaces the old button column in `.canvas-sidebar`.

- **Markup:** `<div className="canvas-action-overlay">` as the last child of `.canvas-wrap`.
- **Position:** `absolute`, top-right inside the relatively-positioned `.canvas-wrap`.
- **Desktop:** Vertical flex column. Transparent by default (only faint individual button backgrounds visible). Hovering the overlay area darkens the container (`rgba(20,15,36,0.85)`) with a subtle border; individual buttons go fully opaque and show a yellow accent on hover.
- **Mobile:** Horizontal flex row. Always slightly opaque (`rgba(20,15,36,0.7)`). Buttons have smaller padding and font.
- **Tap-to-hide (mobile only):** Tapping the canvas outside the overlay toggles `overlayHidden` state. When hidden, the overlay gets `opacity: 0; pointer-events: none` with a 150ms CSS transition. Tapping the canvas again restores it. Taps inside the overlay itself (on PNG/SVG/Share buttons) fire normally without toggling.
- **State:** Controlled by `overlayHidden` (`useState(false)`) inside `App()`. Desktop unaffected — `touchend` is the only event that toggles it.

### Font dropdown

The font picker is a custom dropdown (no native `<select>`) so each option can show a live "Ag" preview rendered in the font's own typeface.

- **Component:** `FontDropdown({ value, onChange, fonts, fontLoaded })`.
- **Closed state:** A trigger button showing the selected font's "Ag" preview inline using its `font-family` + `font-weight`, the font label, and a chevron.
- **Open state:** A scrollable popover list (`<ul>`) with all 24 fonts. Each row displays the "Ag" preview in the font's own face alongside the label. Clicking a row calls `onChange(family)` and closes the list.
- **Dismissal:** Click-outside (mousedown listener) or Escape key closes the popover.
- **fontLoaded gating:** When fonts haven't finished loading, the previews fall back to system fonts but the dropdown still works functionally.
- **Replaces** the old 3-column button grid (`<div className="font-grid">`) which was removed in F13a.

---

## 8. Font loading

Fonts are self-hosted in `fonts/` (woff2, latin subset). They are declared via `@font-face` in `<head>` and consumed by name (`'Poppins'`, `'Playfair Display'`, etc.).

Canvas does not auto-trigger font fetch when you call `ctx.font = '900 100px Poppins'` — the browser may fall back to system fonts until the woff2 is parsed. Rather than rely on lazy `@font-face` resolution, the load effect builds `FontFace` objects **explicitly from the `FONT_FILES` paths** (the same source the SVG exporter fetches), loads them, and adds them to `document.fonts`. Each family is loaded at its declared weight (used by name text + preview buttons) **and** at weight 900 (the big letter renders at 900 in canvas), so the canvas always has the exact weight it requests:

```js
useEffect(() => {
  const loads = [];
  FONTS.forEach(f => {
    const url = FONT_FILES[f.family];
    if (!url) return;
    loads.push(new FontFace(f.family, `url('${url}')`, { weight: String(f.weight) }).load()
      .then(ff => document.fonts.add(ff)).catch(() => {}));
    loads.push(new FontFace(f.family, `url('${url}')`, { weight: "900" }).load()
      .then(ff => document.fonts.add(ff)).catch(() => {}));
  });
  Promise.allSettled(loads).then(() => setFontLoaded(true));
}, []);
```

`fontLoaded` gates both the main canvas redraw effect and each `PresetThumb` redraw. Every load's `.catch` is a no-op and `allSettled` always resolves, so the app degrades to system fonts rather than hanging forever.

> **Font-file integrity:** every woff2 in `fonts/` must contain the full basic-latin alphabet. A corrupt or wrong-subset file (e.g. missing `A`–`Z`/`a`–`z`) loads without error but renders only stray glyphs, with everything else falling back — the failure looks like a code bug but is in the asset. Verify new files cover `AEXMPLgaeo` before committing.

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
- The share warning uses `role="alert"`.
- The import error uses `role="alert"`.
- `display: contents` on `.canvas-sidebar` is supported with correct semantics in current Chrome, Firefox, and Safari. The sidebar div carries no role or label, so semantic loss is not a concern.

---

## 11. Known gotchas

- **SVG export drift** — `exportSVG` is a separate template; it does not mirror `drawMonogram` automatically. Always cross-check both outputs when changing render behavior. Mode branching (`bgMode`/`kMode`/`nameMode`) affects both fill attributes and `<defs>` generation.
- **Canvas internal resolution is fixed at 700x700.** CSS scales it. Don't change the internal resolution without re-tuning every default value in `DEFAULT_SETTINGS`.
- **History pushes are scattered** — `set`, `applyPreset`, `importJSON`, `handleRandomize`, and `commitSaveToSlot` each implement the "slice forward + push + advance" pattern inline. If you add a new state mutator, you must follow the same pattern or undo/redo will get out of sync.
- **`useDebounce` delay is 0ms.** Canvas updates and localStorage writes happen on every render cycle with no intentional delay, ensuring the preview stays perfectly in sync with the controls.
- **localStorage and URL hash can desync.** URL hash wins on initial load. Sharing a link to someone whose localStorage holds different settings will overwrite their working state on visit.
- **`presetSlots` is separate from `settings`.** Preset slot state is its own React state, persisted to its own localStorage key. Importing a settings JSON does not affect preset slots; importing a preset bundle JSON does.
- **`migrateSettings()` fills in missing mode keys.** Old shared URLs or JSON exports without `bgMode`/`kMode`/`nameMode` will get safe defaults (`"gradient"`) without losing existing settings.

---

## 12. Lessons learned — Effects pipeline

### SVG filter chaining is broken by design

SVG's `filter` attribute accepts only one filter reference. Chaining multiple via space-separated list (`filter="url(#a) url(#b)"`) is invalid and renders inconsistently across browsers. Nesting `<g filter>` groups causes the outer filter to receive the inner filter's output as `SourceGraphic`, so the outer filter's `feFlood` tints the inner filter's result — e.g. a shadow filter tints the glow output with the glow color.

**Solution:** When both glow + shadow are active, render 3 independent `<g>` layers:
1. Shadow-only filter (outputs just the shadow, no `SourceGraphic`)
2. Glow-only filter (outputs just the glow halo, no `SourceGraphic`)
3. Original text (no filter)

Each filter reads from the original `SourceGraphic`/`SourceAlpha` directly. No chaining, no nesting, no cross-contamination.

### SVG glow needs `operator="out"` to match canvas intensity

Canvas glow uses `fillStyle = rgba(r,g,b,0)` (transparent) with colored `shadowColor` — only the halo is visible, not the text shape itself. SVG `feFlood` + `feComposite IN SourceGraphic` creates a solid-colored text shape that gets blurred, so the center contributes heavy opacity.

**Solution:** After each blur pass, apply `<feComposite operator="out" in2="SourceGraphic"/>` to subtract the original text shape from the blurred result. This leaves only the spread/halo, matching the canvas behavior.

### Canvas glow needs high alpha values to match SVG

SVG glow with `operator="out"` subtraction creates a cleaner, more concentrated halo. Canvas shadow API creates a softer, more diffuse glow. To match SVG intensity, canvas alpha values need to be significantly higher (0.75–1.0 per pass vs SVG's 0.12–0.5).

### Name shadow thickness must use half-stroke

`SourceAlpha` in SVG includes the stroke, making shadows thicker than the visible text. Canvas `fillText` only draws the fill, making shadows thinner. The middle ground: use `outlineWidth` (half the full `outlineWidth * 2`) for the shadow stroke in both canvas and SVG.

### Layer order matters for visual stacking

Canvas renders in draw order (later = on top). SVG renders in document order (later = on top). For consistent visual stacking:
- Big letter: glow → shadow → main text (glow behind shadow, both behind text)
- Name text: shadow → glow → main text (shadow behind glow, both behind text)

### Filter region must be large enough

SVG filters clip to their filter region by default (`x="-100%" y="-100%" width="300%" height="300%"`). Wide glows and shadows get clipped. Use `x="-200%" y="-200%" width="500%" height="500%"` for all effect filters.

---

## 13. Wave 7 render-path notes (planned — see Sprint-Plan §5b)

Wave 7 adds four big-letter-centric capabilities. All must be mirrored in **both** `drawMonogram` and the SVG builder, and all default to no-ops so old data is unaffected (`migrateSettings`).

- **F19 — letter transforms.** Wrap *all* big-letter passes (glow, shadow, emboss, fill) in one `ctx.save()`+transform+`ctx.restore()` (canvas) and one `<g transform="…">` around `bigLetterSVG` (SVG), both anchored at `(kX, kY)`. Effects must live inside the wrapper so they track the glyph.
- **F18 — emboss.** Two offset glyph copies (highlight at `-d·(cosθ,sinθ)`, shadow at `+d·(cosθ,sinθ)`) under the main fill. Prefer the offset-copy approach in SVG over `feConvolveMatrix` to avoid filter-region clipping and to match canvas exactly. This is the *fourth* big-letter pass — fold it into the §12 layer-order list: emboss sits directly under the main fill.
- **F16 — blend mode.** Canvas `globalCompositeOperation` + `globalAlpha` on the fill pass; SVG `style="mix-blend-mode:…"` + `opacity` on the big-letter `<g>`. Names map 1:1. Document that non-browser SVG rasterizers may ignore `mix-blend-mode`.
- **F17 — layer stack.** Refactor `drawMonogram` and the SVG builder to emit four named fragments (background, rings, bigLetter, name) then concatenate per `layerOrder`, skipping hidden ones. **SP-21 is the regression guard:** default order + all visible must be pixel-identical to today. Excluded from randomize.

> ⚠️ The §12 effects pipeline relies on canvas/SVG values that were hand-tuned to *look* matched rather than being derived from one source of truth (e.g. canvas glow alpha 0.75–1.0 vs SVG 0.12–0.5). Any Wave 7 change that re-touches the big-letter passes should re-verify glow/shadow parity, not assume it holds.

---

## 14. Parity audit — glow/shadow canvas vs SVG (2026-06-02)

> **STATUS UPDATE (2026-06-02, post-rewrite).** The big-letter render path was substantially rewritten after this audit. The blend-mode architecture is now:
> - **Canvas (`drawBigLetter`):** glow/shadow/emboss are drawn on the main ctx inside the letter transform; the letter fill is then blended against a **background copy** in an offscreen buffer, masked to the glyph, and composited opaque on top. The blend mode therefore touches only the background, never the letter's own effects.
> - **SVG (`downloadSVG` assembly):** effects go in a transformed `<g>` below the letter; the letter is an **isolated, glyph-clipped group** (`isolation:isolate`) containing a background copy + the letter path with `mix-blend-mode`, so it blends only against the background copy.
> - **Stacking order** is now **shadow → glow → emboss → fill** in *both* renderers (item #1 below — RESOLVED).
>
> Net result of that work: items **#1 and #5 are resolved**; **#2, #3, #4 remain** as minor, non-blocking parity gaps (see "Remaining gaps / roadmap" at the end of this section). The line numbers below are from the pre-rewrite code and are kept for historical context only.

Findings from a line-by-line comparison of the big-letter and name effect passes in `drawMonogram` against the SVG filters/assembly in the `downloadSVG` builder. Outputs currently look matched, but several matches are **coincidental** (hold only at default values) rather than derived. Ordered by severity.

1. **Big-letter glow/shadow stacking order is inverted between the two renderers.**
   - Canvas draws glow → shadow → fill, so the **shadow sits on top of the glow** (L704–739).
   - SVG `bigBoth` assembles `bigShadowOnly` then `bigGlowOnly` then text, so the **glow sits on top of the shadow** (L1271–1273).
   - §12 states the intended order is "glow → shadow → text" (canvas is correct; SVG is wrong). Name effects are consistent (both do shadow → glow → text, L753–799 / L1295–1297). The mismatch is masked today because the dark, offset shadow and the colored halo overlap only in a thin penumbra. It will become visible with larger blur/offset or opaque shadow colors. **Recommend flipping the SVG big-letter order to shadow-under-glow… no — flip to match canvas: glow under shadow.**

2. **Drop-shadow blur is ~2× softer on canvas than SVG for the same `blur` value.**
   - Canvas sets `ctx.shadowBlur = blur` directly (L729); browsers implement that as a Gaussian of ≈ `blur/2` stdDeviation.
   - SVG sets `feGaussianBlur stdDeviation = blur` (L1182/1245), i.e. ~2× wider.
   - The glow passes *do* reconcile this (canvas `shadowBlur = rad*2`, SVG `stdDeviation = rad`), but the shadow path does not. Fix: SVG shadow `stdDeviation = (blur)/2`, or canvas `shadowBlur = blur*2`. Pick one convention and apply everywhere.

3. **Per-pass glow opacities are unrelated between renderers — tuned by eye, not derived.**
   - Canvas halo alphas: `0.75 / 1.0 / 1.0` + a 4th opaque pass (L706, L715).
   - SVG flood-opacities: `0.12 / 0.20 / 0.35 / 0.50` (×intensity for passes 1–3) (L1189–1201).
   - Blur radii *do* line up 1:1 (canvas `shadowBlur/2` == SVG `stdDeviation`), so the geometry matches; only the opacity ramps are guesses. Any change to color, intensity, or pass count will desync them. A shared constant table (radii + opacities) consumed by both paths would make this robust.

4. **Glow intensity scaling diverges away from `intensity = 1.0`.**
   - Canvas 3rd/4th passes use fixed alpha `1.0` regardless of intensity (only blur scales).
   - SVG passes 1–3 scale flood-opacity by intensity, but the **4th pass is hardcoded** (`0.5` big, `0.4` name — L1201/1080/1114/1231) and not scaled.
   - Net: parity holds near the default `intensity = 1.0` and drifts at the extremes. This is the clearest "matches for the wrong reason."

5. **Canvas drop-shadow draws an extra sharp shadow-colored letter at the origin; SVG does not.**
   - Canvas shadow pass sets `fillStyle` to the shadow color *and* casts the offset shadow, so a crisp shadow-colored glyph is painted at the un-offset position (L727–734); it's normally hidden by the opaque main fill.
   - SVG uses `SourceAlpha` + `feOffset` only — no un-offset copy.
   - **This becomes a visible divergence the moment the letter fill is non-opaque** — i.e. exactly what **F16 (`letterOpacity`) introduces in Wave 7.** Flag: when implementing F16, drop the canvas un-offset shadow fill (draw the shadow via a separate offset pass that doesn't paint at origin) so the two renderers agree.

### Resolution status (2026-06-02)

| # | Issue | Status |
|---|---|---|
| 1 | Glow/shadow stacking order inverted | **Resolved** — both renderers now draw shadow → glow → emboss → fill. |
| 5 | Canvas origin-glyph shadow bleed under opacity | **Resolved (moot)** — shadow is now an independent layer; the letter composites opaque on top via the buffer/clip approach, so there's no reliance on covering an origin glyph. |
| 2 | Drop-shadow blur ~2× softer on canvas than SVG | **Open** (minor) — see roadmap. |
| 3 | Per-pass glow opacities hand-tuned per renderer | **Open** (minor) — see roadmap. |
| 4 | Glow 4th-pass blur/intensity scaling divergence | **Open** (minor) — see roadmap. |

### Remaining gaps / roadmap

These are cosmetic parity nuances only visible when blur or glow intensity is pushed to extremes and the canvas/PNG is compared side-by-side with the exported SVG. They are **not** causing visible problems at normal settings and are deliberately deferred. Tracked as roadmap item **R1 — "Glow/shadow parity hardening"** (see `Plan.md` and `Sprint-Plan.md`):

- **#2 — unify the shadow-blur convention.** Canvas `ctx.shadowBlur = blur` renders as a Gaussian of ≈ `blur/2`; SVG `feGaussianBlur stdDeviation = blur` is ~2× wider. Pick one convention (e.g. SVG `stdDeviation = blur/2`) and apply everywhere.
- **#3 — single source of truth for glow.** The per-pass glow radii line up 1:1, but the opacity ramps are hand-tuned separately in each renderer. Extract a shared `{ radius, canvasAlpha, svgOpacity }` pass table consumed by both paths.
- **#4 — consistent intensity scaling.** Scale every glow pass (including the 4th) by intensity identically in both renderers, so parity holds away from `intensity = 1.0`.

When R1 is picked up, do it as one refactor: derive both renderers' glow/shadow parameters from shared constants so parity is "by construction" rather than "matched at defaults."
