# Agents.md

Operating guide for agentic coders (and humans) extending Monogram Studio. Read this **before** editing `index.html`.

This document is task-oriented. For *why* the code is shaped the way it is, read `Architecture.md`. For *what data flows through it*, read `Schemas.md`.

---

## 0. Ground rules

1. **The app is one file.** Do not split `index.html` into modules, do not introduce a build step, do not add a `package.json`. Portability is the product.
2. **No npm dependencies.** React, ReactDOM, and Babel come from `unpkg` CDN script tags. Anything else must be a single self-hosted asset (woff2 font, image, etc.).
3. **No tests run automatically.** There is no test runner. Verify changes against the acceptance checklist in `Plan.md` by hand.
4. **Backward compatibility is sticky.** Any old shared URL or exported JSON must continue to load. See "Renaming a settings key" below.
5. **Keep canvas and SVG outputs visually identical.** Two separate render paths exist (`drawMonogram` for canvas, `exportSVG` for SVG). Changes to one require mirroring changes in the other.

---

## 1. Where to find things in `index.html`

Use the section comment markers to navigate. They look like `// ── Section name ──`. Stable markers:

| Marker | Approx. role |
|---|---|
| `// ── Debounce hook ──` | The `useDebounce` custom hook. |
| `// ── Font catalogue ──` | The `FONTS` constant. |
| `const TEXTURES =` | Texture list. |
| `const PRESETS =` | Preset list. |
| `const SHAPES =` | Shape list. |
| `const DEFAULT_SETTINGS =` | Canonical settings. |
| `// ── Utility ──` | `hexToRgb`, `createDynamicGradient`. |
| `function drawTexture` | Texture renderer. |
| `function drawMonogram` | **Main canvas render.** |
| `// ── SVG Export ──` | `buildShapeSVGPath`, `exportSVG`. |
| `function clipShape` | Canvas shape helpers. |
| `// ── Preset thumbnail renderer ──` | `PresetThumb` component. |
| `// ── URL share encode/decode ──` | `encodeSettings`, `decodeSettings`. |
| `// ── HSL-to-HEX ──` | `hslToHex` utility. |
| `// ── UI Components ──` | `ColorRow`, `SliderRow`. |
| `function App()` | Top-level component. |
| `// ── Load from URL hash or localStorage ──` | Initial-settings resolver. |
| `// ── Undo/redo history ──` | History refs and `set`/`undo`/`redo`. |
| `// ── Auto-save to localStorage ──` | Persistence effect. |
| `// ── Font loading ──` | Font-load effect. |
| `const handleRandomize` | Randomize generator. |

CSS lives in two places:

- `<head><style>` at the top of the file — `@font-face` declarations only.
- `<style>{` template-string `}</style>` inside `App()`'s return — all layout / component CSS. Search for `/* ── Canvas area ── */` etc. to navigate.

---

## 2. Variable naming conventions

| Pattern | Meaning |
|---|---|
| `bg*` | Background. `bgTop`, `bgBot`, `bgGradientType`, `bgGradientAngle`, `bgGradientStop`. |
| `k*` | The "big letter" (`K` for "Kanji" historically — kept for compatibility). `kTop`, `kBot`, `kGradientType`, `kGradientAngle`, `kGradientStop`. **Do not rename.** Shared URLs depend on these literal keys. |
| `name*` | The lower-badge name text. `nameTop`, `nameBot`, `nameSize`, `nameOffsetY`, `nameGradient*`. |
| `ring1*` | Outer ring. `ring1`, `ring1Width`, `ring1Gap`, `showRing1`. |
| `ring2*` | Inner ring. `ring2`, `ring2Width`, `showRing2`. (`ring2Gap` does not exist — `ring1Gap` controls the spacing between outer and inner.) |
| `letterSize` / `letterSpacing` / `letterOffsetX` / `letterOffsetY` | Big-letter typography. |
| `glow` / `glowColor` / `glowIntensity` | Halo effect on the big letter. |
| `outlineColor` / `outlineWidth` | Stroke around the shape perimeter. |

Inside `drawMonogram`, short-name locals dominate (`cx`, `cy`, `R`, `W`, `H`, `ctx`). Keep that consistent — that function is dense and short variable names help readability when scanning the math.

---

## 3. Common tasks — step by step

### 3.1 Add a settings key

Use case: "I want a slider for `bigLetterRotation`."

1. **Add to `DEFAULT_SETTINGS`** with a safe default that produces no visual change.
2. **Read in `drawMonogram`** — destructure the key, apply it during render.
3. **Read in `exportSVG`** — same; mirror the visual effect in SVG.
4. **Add a UI control** in the appropriate tab (design / colors / export). Use `SliderRow` for numeric, `ColorRow` for hex, `<button className="toggle">` for boolean.
5. **Add to `handleRandomize`** if it should be randomized; pick a sensible random distribution.
6. **Decide if it goes into `PRESETS`** — if the new key meaningfully shifts the design, update presets so they remain visually faithful.
7. **Verify round-trip**: change → export JSON → reload page → import JSON → confirm value sticks. Change → Share → copy URL → open URL in new tab → confirm value sticks.
8. **Update `Schemas.md`** in the appropriate section.

### 3.2 Add a new shape

1. **`SHAPES` const** — append the shape name.
2. **`buildShapePath(ctx, shape, cx, cy, R)`** — add a `case` (or `else if`) that draws the path to the canvas context.
3. **`clipShape` and `strokeShape`** — verify they delegate to `buildShapePath` (they do; no change needed).
4. **`buildShapeSVGPath(shape, cx, cy, R)`** — add the SVG path string for the same shape.
5. **Test all 8 shape buttons** in the Design tab — clicking each should produce the matching shape in the preview and in the SVG export.
6. **Update `Schemas.md` §3** with the new shape name.

### 3.3 Add a new font

Five steps now (the SVG path-conversion pipeline needs to know about the font file too):

1. Drop the woff2 into `fonts/`.
2. Add an `@font-face` declaration to the `<style>` block in `<head>`.
3. Append an entry to `FONTS`.
4. **Add the family → file path mapping to `FONT_FILES`** (just above `// ── SVG Export ──`). The SVG exporter uses this map to fetch and parse the font for path conversion. Omitting this step means the SVG export will silently fall back to Poppins for that font.
5. Verify font preview thumbnails render *and* that exporting an SVG with the new font selected produces vector paths in the correct shape (open the SVG in a browser and confirm the text isn't system-font fallback).

Pitfall: the `family` field in `FONTS` must match the `font-family` in `@font-face` byte-for-byte. A space, casing difference, or invisible character will cause the preview to fall back to system fonts silently.

Pitfall: the woff2 weight in `FONT_FILES` should ideally match the heaviest weight used (the big-letter renders at weight 900 in canvas/SVG). For Poppins we point `FONT_FILES["Poppins"]` at the 900-weight file specifically; other fonts only have one weight and use that.

### 3.4 Add a new preset

1. **Build the design in the UI.**
2. **Open the Export/Import tab** — copy the "Current Settings" JSON view.
3. **Trim** to only the keys you want the preset to override (often everything except `bigLetter` and `name`, but presets typically override those too — they're meant to be a full identity, not a partial theme).
4. **Append to `PRESETS`** in source order (presets render in array order).
5. **Pick a unique `name`** — see `Schemas.md` §5 for the active-detection rule. Two presets sharing both `name` and `bgTop` cause the second to never appear active.
6. **Verify the preset thumbnail renders correctly** — it uses a stripped-down render with the preset's colors and big letter. If the thumbnail looks wrong but the full preview is fine, the issue is in `PresetThumb`'s render path, not your data.

### 3.5 Change layout / responsive breakpoints

The mobile breakpoint is `@media (max-width: 768px)`. The "very small phone" breakpoint is `@media (max-width: 380px)`. Both are inside the `<style>` template in `App()`.

Mobile-specific reminders:

- `.canvas-sidebar` uses `display: contents` on mobile so its child `.canvas-actions` promotes to a direct child of `.canvas-row`. If you change the sidebar's children, **make sure the desktop preset bits (`presets-label`, `presets`) are still hidden on mobile** via the `.canvas-sidebar > .presets-label, .canvas-sidebar > .presets { display: none }` selector.
- Mobile presets render from a **second copy** of the `PRESETS.map(...)` inside `.panel-content`. The desktop copy in `.canvas-sidebar` is hidden via `display: none` on `.mobile-presets-wrap`. If you change the preset rendering, change both.
- The mobile presets are conditionally rendered: `{tab !== "export/import" && (...)}`. Leave that guard in place.

### 3.6 Change rendering behavior

Anything that changes how the design looks must update **both**:

- `drawMonogram(canvas, settings)` — Canvas rendering.
- `exportSVG(settings)` — SVG rendering. **Note: this function is async** because it fetches and parses the font with Typr.js to convert text to vector paths. If you add a new caller, `await` it.

Visual diff workflow (no automation; do this by eye):

1. Set up a recognizable test design (e.g., apply Royal Gold preset, set `bigLetter` to `XO`).
2. Take a screenshot of the canvas preview.
3. Click "Download SVG" and open the SVG in a browser tab.
4. Compare side-by-side.
5. Repeat for a complex design (apply Neon Noir, enable both rings + glow, change shape to hexagram).

If you only need to change canvas (e.g., a performance tweak), leave a comment noting that SVG was intentionally not changed.

### 3.7 Rename a settings key

**Avoid.** Renames break old shared URLs and old JSON exports.

If you must:

1. Add the new key to `DEFAULT_SETTINGS`.
2. In `getInitialSettings()` (and `importJSON`), add a migration:

   ```js
   if (parsed.oldKey !== undefined && parsed.newKey === undefined) {
     parsed.newKey = parsed.oldKey;
     delete parsed.oldKey;
   }
   ```

3. Remove the old key from `DEFAULT_SETTINGS` only after a deprecation window.
4. Update all references in `drawMonogram`, `exportSVG`, JSX, and `handleRandomize`.
5. Update `Schemas.md` and add a note in `Plan.md`'s "migrations" section.

### 3.8 Mutate `settings` (any code path)

Every code path that mutates `settings` must:

1. **Slice** forward history: `historyRef.current = historyRef.current.slice(0, historyIndexRef.current + 1);`
2. **Push** the new snapshot.
3. **Cap** history at 100 entries: `if (historyRef.current.length > 100) historyRef.current.shift();`
4. **Advance** the pointer: `historyIndexRef.current = historyRef.current.length - 1;`

This pattern lives in `set`, `applyPreset`, `importJSON`, and `handleRandomize`. If you add a new mutator, repeat it. Failure to slice forward history will cause undo/redo to behave bizarrely (redo past a "fork" point).

---

## 4. Things you must not do

- **Don't introduce a build step.** No Webpack, no Vite, no `package.json`, no TypeScript compiler.
- **Don't use external CSS frameworks.** No Tailwind, no Bootstrap. The CSS is hand-rolled and lives in the file.
- **Don't fetch data over the network at runtime** beyond the existing CDN scripts. The app must work offline once loaded.
- **Don't write to `localStorage` outside the existing autosave effect.** All state flows through `setSettings`.
- **Don't bypass the history machinery** when mutating `settings`. Direct `setSettings(x)` without history-push will silently break undo/redo.
- **Don't add tracking, analytics, or telemetry.** The "no server-side processing" claim in the README is a load-bearing user promise.
- **Don't rename the `k*` keys** (`kTop`, `kBot`, `kGradientType`, etc.). They're old shorthand but every shared URL in the wild references them.
- **Don't change the canvas internal resolution from 700x700** without re-tuning every default in `DEFAULT_SETTINGS`. The slider ranges are calibrated for this coordinate space.

---

## 5. Quick reference: file targets

| If you are changing... | Edit... | Then update... |
|---|---|---|
| A render detail (canvas) | `drawMonogram` | `exportSVG`, then verify visually |
| A render detail (SVG only) | `exportSVG` | Note the divergence in `Plan.md` |
| Add/remove/rename a setting | `DEFAULT_SETTINGS` | `drawMonogram`, `exportSVG`, JSX, `handleRandomize`, `Schemas.md` |
| Add a font | `<head><style>` + `FONTS` + `FONT_FILES` | Verify in browser; check preset thumbnails AND SVG export with that font |
| Add a shape | `SHAPES`, `buildShapePath`, `buildShapeSVGPath` | Test every shape button |
| Add a preset | `PRESETS` | Verify thumbnail, verify "active" detection |
| Mobile layout | `@media (max-width: 768px)` block | Test in DevTools responsive mode at 320, 380, 600, 768 |
| Desktop layout | Base CSS (above the `@media`) | Test at 1024, 1200, 1440 |
| Persistence behavior | `getInitialSettings`, `importJSON`, `useEffect [settings]` | Round-trip test JSON + URL hash |
| Undo/redo | History refs + every mutator | Test 5+ rapid edits + undo all + redo all |

---

## 6. Acceptance smoke test (run before merge)

A minimal flight check. Open `index.html` in a fresh browser tab (clear localStorage first):

1. **Initial load** — Preview shows `E EXAMPLE` in Royal-Gold-style colors (the default).
2. **Edit text** — Change Big Letter to `RG`, Name to `Royal Gold`. Preview updates immediately.
3. **Click presets** — All 6 preset thumbnails render correctly; clicking each applies cleanly.
4. **Each tab loads** — Design, Colors, Export/Import all render without console errors.
5. **Shape picker** — Click each of the 8 shapes. Preview reflects each.
6. **Font picker** — Click each of the 8 fonts. Preview reflects each.
7. **Texture picker** — Cycle through 5 textures.
8. **Sliders** — Move letter size, glow intensity, gradient angle — preview updates live.
9. **Toggles** — Disable outer ring, disable inner ring, disable glow. Preview updates.
10. **Undo / redo** — Make 5 changes, press Ctrl+Z 5 times — back to start. Ctrl+Y forward 5 times.
11. **PNG download** — Click Download PNG. File saves with name `monogram_<letter>_<name>.png` and opens correctly.
12. **SVG download** — Click Download SVG. File opens correctly in a browser; visually matches the canvas preview.
13. **JSON export → import** — Export JSON, refresh page, import the JSON. Design restored.
14. **Share link** — Click Share, open the copied URL in an incognito tab. Same design appears.
15. **Randomize** — Click Randomize Design. Output is cohesive (not a random mess of clashing colors); colors fit one of: analogous / complementary / monochromatic.
16. **Mobile (DevTools responsive @ 375px)** — Preview is centered, action buttons are a horizontal row below the preview, presets appear at the top of the panel and scroll with parameters, presets are hidden on the Export/Import tab.
17. **Mobile (DevTools responsive @ 1024px)** — Presets list on the left, action buttons below it in the same column, large preview on the right.

Any failure here is a release blocker.

---

## 7. When in doubt

- Re-read the section of `Architecture.md` covering the area you're changing.
- Check `Schemas.md` for the source-of-truth data shape.
- Run the acceptance smoke test (§6) before declaring "done."
- If a behavior surprises you, suspect history sync first (`set` vs raw `setSettings`) and SVG/canvas drift second.
