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
6. **Incremental implementation.** Work on one function or subtask at a time. Never bulk-edit multiple sections in a single pass. After each major function is implemented, stop and document completion in the sprint progress tracker before moving to the next. This enables testing at each checkpoint and prevents cascading errors.

---

## 1. Where to find things in `index.html`

Use the section comment markers to navigate. They look like `// ── Section name ──`. Stable markers:

| Marker | Approx. role |
|---|---|
| `// ── Debounce hook ──` | The `useDebounce` custom hook (delay: 0). |
| `// ── Font catalogue ──` | The `FONTS` constant (24 families). |
| `const TEXTURES =` | Texture list. |
| `const SHIPPED_PRESETS =` | 6 built-in design presets (factory defaults for slots 0–5). |
| `const TOTAL_PRESET_SLOTS =` | 10-slot preset store constant. |
| `const PRESET_SLOTS_KEY =` | localStorage key for preset slots. |
| `const RANDOMIZER_CATEGORIES =` | Category-to-keys map for randomizer lock toggles. |
| `function Section()` | Collapsible section component (persisted via `useUIState`). |
| `function useUIState()` | UI state hook for collapsed sections / randomizer locks, stored in `monogram-studio-ui-state`. |
| `function FontDropdown()` | Custom font picker dropdown (replaces the old font grid). |
| `function setEffect()` | Writes to `effects.*` sub-object (glow/shadow). |
| `function rgbToHex()` | RGB-to-hex conversion (used by color picker). |
| `function hexToHsv()` | Hex-to-HSV conversion (used by color wheel). |
| `function hsvToHex()` | HSV-to-hex conversion (used by color wheel). |
| `function ColorPickerRow()` | HSV color wheel popover (swatch + HEX/R/G/B inputs). |
| `function ColorWheel()` | 200×200 HSV color wheel canvas component. |
| `function persistSlots()` | Saves `presetSlots` state to localStorage. |
| `function getOrInitSlots()` | Loads preset slots from localStorage (or initializes from shipped). |
| `function migrateSettings()` | Fills missing keys on external settings load (URL hash, JSON import). |
| `const SHAPES =` | Shape list. |
| `const DEFAULT_SETTINGS =` | Canonical settings. Includes `bgMode`, `kMode`, `nameMode`, `effects` sub-object. |
| `// ── Utility ──` | `hexToRgb`, `createDynamicGradient`. |
| `function drawTexture` | Texture renderer. |
| `function drawMonogram` | **Main canvas render.** Branches on `bgMode`/`kMode`/`nameMode` for solid vs gradient. Reads `effects.*` for glow/shadow. |
| `// ── SVG Export ──` | `buildShapeSVGPath`, `exportSVG`. Mode-branched fills; `<defs>` only generated for gradient modes. |
| `function clipShape` | Canvas shape helpers. |
| `// ── Preset thumbnail renderer ──` | `PresetThumb` component. |
| `// ── URL share encode/decode ──` | `encodeSettings`, `decodeSettings`. |
| `// ── HSL-to-HEX ──` | `hslToHex` utility. |
| `// ── UI Components ──` | `SliderRow`, `ColorPickerRow`, `ColorWheel`, `Section`. |
| `function App()` | Top-level component. Includes `presetSlots` state, slot action handlers, `overlayHidden`, `randomizerTick`. |
| `// ── Load from URL hash or localStorage ──` | Initial-settings resolver (calls `migrateSettings`). |
| `// ── Undo/redo history ──` | History refs and `set`/`undo`/`redo`. |
| `// ── Auto-save to localStorage ──` | Persistence effect (0ms debounce — writes every render). |
| `// ── Font loading ──` | Font-load effect. |
| `const handleRandomize` | Randomize generator (respects `randomizerLocks` from UI state). |

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
| `effects.glow.*` / `effects.shadow.*` / `effects.nameGlow.*` / `effects.nameShadow.*` | Centralised effects sub-object (glow, drop shadow, name glow, name shadow) on the big letter and name text. |
| `outlineColor` / `outlineWidth` | Stroke around the shape perimeter. |

Inside `drawMonogram`, short-name locals dominate (`cx`, `cy`, `R`, `W`, `H`, `ctx`). Keep that consistent — that function is dense and short variable names help readability when scanning the math.

---

## 3. Common tasks — step by step

### 3.1 Add a settings key

Use case: "I want a slider for `bigLetterRotation`."

1. **Add to `DEFAULT_SETTINGS`** with a safe default that produces no visual change.
2. **Read in `drawMonogram`** — destructure the key, apply it during render.
3. **Read in `exportSVG`** — same; mirror the visual effect in SVG.
4. **Add a UI control** in the appropriate tab (design / colors / export). Use `SliderRow` for numeric, `ColorPickerRow` for hex, `<button className="toggle">` for boolean, segmented button group for enum modes (e.g. `solid`/`gradient`).
5. **Add to `handleRandomize`** if it should be randomized; pick a sensible random distribution.
6. **Decide if it goes into `SHIPPED_PRESETS`** — if the new key meaningfully shifts the design, update `SHIPPED_PRESETS` so the presets remain visually faithful.
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

There are currently **24 families** in the catalogue. The font picker is a custom dropdown (`FontDropdown` component) — there is no font grid. Steps:

1. Drop the woff2 into `fonts/` — **and verify it has full basic-latin coverage** (see pitfall below).
2. Add an `@font-face` declaration to the `<style>` block in `<head>`.
3. Append an entry to `FONTS` (the dropdown reads from this array directly).
4. **Add the family → file path mapping to `FONT_FILES`** (just above `// ── SVG Export ──`). The SVG exporter uses this map to fetch and parse the font for path conversion. Omitting this step means the SVG export will silently fall back to Poppins for that font. The font-load effect also reads `FONT_FILES`, so a missing entry means the font never loads at all.
5. Verify the dropdown preview renders the new font's "Ag" sample *and* that exporting an SVG with the new font selected produces vector paths in the correct shape (open the SVG in a browser and confirm the text isn't system-font fallback).

Pitfall: the `family` field in `FONTS` must match the `font-family` in `@font-face` byte-for-byte. A space, casing difference, or invisible character will cause the preview to fall back to system fonts silently.

Pitfall: **the woff2 file must contain the basic-latin alphabet.** Wrong/partial subsets (e.g. a cyrillic-only or broken download) load with no error but render only stray glyphs while everything else falls back — it looks exactly like a code bug. Source files from `@fontsource/<family>` (npm registry is reachable from the sandbox; Google's CDN is not) and confirm coverage of `AEXMPLgaeo` with fontTools before committing. The `latin-400-normal` / `latin-700-normal` files in those packages are correct.

Pitfall: the woff2 weight in `FONT_FILES` should ideally match the heaviest weight used (the big-letter renders at weight 900 in canvas/SVG). For Poppins we point `FONT_FILES["Poppins"]` at the 900-weight file specifically; other fonts only have one weight and use that.

### 3.4 Add a new preset

1. **Build the design in the UI.**
2. **Open the Export/Import tab** — copy the "Current Settings" JSON view.
3. **Trim** to only the keys you want the preset to override (often everything except `bigLetter` and `name`, but presets typically override those too — they're meant to be a full identity, not a partial theme).
4. **Append to `SHIPPED_PRESETS`** in source order (presets render in array order).
5. **Pick a unique `name`** — see `Schemas.md` §5 for the active-detection rule. Two presets sharing both `name` and `bgTop` cause the second to never appear active.
6. **Verify the preset thumbnail renders correctly** — it uses a stripped-down render with the preset's colors and big letter. If the thumbnail looks wrong but the full preview is fine, the issue is in `PresetThumb`'s render path, not your data.
7. **Update `presetSlots` initialization** — shipped presets populate slots 0–5 by default. If you add a 7th preset, it becomes a new slot default; if you remove one, the slot becomes empty until the user saves into it.

### 3.5 Change layout / responsive breakpoints

The mobile breakpoint is `@media (max-width: 768px)`. The "very small phone" breakpoint is `@media (max-width: 380px)`. Both are inside the `<style>` template in `App()`.

Mobile-specific reminders:

- `.canvas-sidebar` uses `display: contents` on mobile so its child presets section is hidden. The floating action overlay stays inside `.canvas-wrap` and is always visible on mobile.
- Mobile presets render from a **second copy** of the `PRESETS.map(...)` inside a `<Section>` at the top of `.panel-content`. Desktop and mobile copies share the same `sectionKey="presets"` so collapse states are synced.
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

### 3.9 Add an effect (glow, shadow, emboss, etc.)

Effects live in `settings.effects.*`. Use this checklist when adding a new effect:

1. **Add to `DEFAULT_SETTINGS.effects`** — add a new sub-object with safe off-state defaults (e.g. `{ enabled: false, ... }`).
2. **Add migration in `migrateSettings`** — use `if (!s.effects.myEffect) s.effects.myEffect = { ... }` to handle old data.
3. **Canvas render** — render the effect inside `drawMonogram` using the appropriate big-letter or name-text block. Gate it behind `settings.effects.myEffect.enabled`.
4. **SVG export** — mirror the effect in `exportSVG`. Add `<filter>` elements to `<defs>` if needed. Gate behind the same `enabled` flag.
5. **Add `setEffect` write path** — if the effect needs a dedicated setter, add a new branch in the `setEffect` callback (or use `setSettings` directly if simple). No legacy-mirror is required unless backward-compat with old top-level keys is needed.
6. **UI controls** — add a `<Section title="My Effect" sectionKey="effects_myeffect" defaultOpen={false}>` in the Effects tab. Use `setEffect("myEffect", "field", value)` for writes. Gate controls behind the `enabled` toggle.
7. **Randomizer** — if the effect should be randomized, add its keys to the `effects` block in `handleRandomize`. Locking the `effects` category (via `RANDOMIZER_CATEGORIES`) will preserve it.
8. **Update docs** — add to `Schemas.md` §1 (Settings table), `Architecture.md` §4 (rendering pipeline), and this recipe.

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
|---|---|---|---|
| A render detail (canvas) | `drawMonogram` | `exportSVG`, then verify visually |
| A render detail (SVG only) | `exportSVG` | Note the divergence in `Plan.md` |
| Add/remove/rename a setting | `DEFAULT_SETTINGS` | `drawMonogram`, `exportSVG`, JSX, `handleRandomize`, `Schemas.md` |
| Fill mode (solid/gradient) | `drawMonogram` + `exportSVG` (branch on `bgMode`/`kMode`/`nameMode`); UI toggles in Colors tab |
| Add a font | `<head><style>` + `FONTS` + `FONT_FILES` | Verify in browser; check preset thumbnails AND SVG export with that font; add to `FontDropdown` list |
| Add a shape | `SHAPES`, `buildShapePath`, `buildShapeSVGPath` | Test every shape button |
| Add a preset | `SHIPPED_PRESETS` | Verify thumbnail, verify "active" detection; update preset slot initialization |
| Preset slots | `presetSlots` state, `persistSlots()`, `getOrInitSlots()` | Test save/delete/reset, export/import bundle, storage warning |
| Color picker popover | `ColorPickerRow`, `ColorWheel`, `hexToHsv`, `hsvToHex`, `rgbToHex` | Test HSV wheel drag, hex input, click-outside close, mobile bottom sheet |
| Collapsible sections | `Section` component, `useUIState` hook | Test collapse/expand, verify persistence across reload |
| Font dropdown | `FontDropdown` component, `FONTS` array | Test open/close, font selection, "Ag" preview renders in each font |
| Effects tab / an effect | `setEffect` helper + `effects.*` in `DEFAULT_SETTINGS` | `drawMonogram`, `exportSVG`, UI `<Section>` in Effects tab, `handleRandomize`, `Schemas.md` |
| Randomizer categories | `RANDOMIZER_CATEGORIES` map | `handleRandomize` lock-strip logic, `randomizerLocks` in UI state |
| Floating overlay | `.canvas-action-overlay` in `.canvas-wrap` | Test desktop hover, mobile tap-to-hide, PNG resolution text |
| Mobile layout | `@media (max-width: 768px)` block | Test in DevTools responsive mode at 320, 380, 600, 768 |
| Desktop layout | Base CSS (above the `@media`) | Test at 1024, 1200, 1440 |
| Persistence / migration | `migrateSettings` + `getInitialSettings` + `importJSON` | Round-trip test JSON + URL hash; test old-data loads |
| Undo/redo | History refs + every mutator | Test 5+ rapid edits + undo all + redo all |
| Letter transforms (F19) | `letterScaleX/Y`, `letterSkewX/Y`, `letterRotate` in `DEFAULT_SETTINGS` + `drawMonogram` (transform wrapper) + SVG `<g transform>` | Typography sliders in Design tab; randomizer wiring in `RANDOMIZER_CATEGORIES.typography` |
| Emboss effect (F18) | `effects.emboss` in `DEFAULT_SETTINGS` + `drawMonogram` + SVG offset paths | Effects tab section; randomizer wiring in `handleRandomize` effects block |
| Blend modes (F16) | `letterBlendMode`, `letterOpacity` in `DEFAULT_SETTINGS` + `drawMonogram` (globalCompositeOperation) + SVG `mix-blend-mode` | Colors → Big Letter section dropdown + slider; `BLEND_MODES` constant; `RANDOMIZER_CATEGORIES.letterColor` |
| Layer stack (F17) | `layers` array in `DEFAULT_SETTINGS` + `drawMonogram` (layer draw map) + SVG (layer fragment concat) | Design → Layers section; migration validates layer ids |

---

## 6. Acceptance smoke test (run before merge)

A minimal flight check. Open `index.html` in a fresh browser tab (clear localStorage first):

1. **Initial load** — Preview shows `E EXAMPLE` in Royal-Gold-style colors (the default).
2. **Edit text** — Change Big Letter to `RG`, Name to `Royal Gold`. Preview updates immediately.
3. **Click presets** — All 6 preset thumbnails render correctly; clicking each applies cleanly.
4. **Each tab loads** — Design, Colors, Export/Import all render without console errors.
5. **Shape picker** — Click each of the 9 shapes. Preview reflects each. "None" shows transparent background with no rings.
6. **Font picker** — Click each of the 24 fonts. Preview reflects each.
7. **Texture picker** — Cycle through 5 textures.
8. **Sliders** — Move letter size, glow intensity, gradient angle — preview updates live.
9. **Toggles** — Disable outer ring, disable inner ring, disable glow. Preview updates.
10. **Undo / redo** — Make 5 changes, press Ctrl+Z 5 times — back to start. Ctrl+Y forward 5 times.
11. **PNG download** — Click Download PNG. File saves with name `monogram_<letter>_<name>.png` and opens correctly. Change resolution to 2×, download again — file is 1400×1400 and pixel-perfect.
12. **SVG download** — Click Download SVG. File opens correctly in a browser; visually matches the canvas preview.
13. **JSON export → import** — Export JSON, refresh page, import the JSON. Design restored.
14. **Share link** — Click Share, open the copied URL in an incognito tab. Same design appears.
15. **Reset to default** — Make changes, click ↺ button in header. Design resets to defaults. Button is disabled when already at defaults.
16. **Randomize** — Click Randomize Design. Output is cohesive (not a random mess of clashing colors); colors fit one of: analogous / complementary / monochromatic.
17. **Color picker** — Click a color swatch in Colors tab. HSV wheel popover opens. Drag hue ring, drag inside square, type hex/RGB values. Click outside or press ESC to close. Preview updates live.
18. **Solid/gradient toggle** — In Background section, click "solid" → preview fills with top color only; bottom color and gradient controls hide. Click "gradient" → bottom color and controls reappear. Repeat for Big Letter and Name Text.
19. **Preset slots** — Apply a preset, click "Save to Slot" → slot picker grid shows. Save into slot 7 (empty). Click the saved slot → settings restore. Click reset arrow on slot 7 → restores to empty.
20. **Preset export/import** — Export presets bundle, clear all slots, import the bundle. Slots restored.
21. **Floating overlay PNG resolution** — Hover overlay, see "↓ PNG / 700×700". Change resolution selector to 2× → second line shows "1400×1400". Changes on slider drag.
22. **Share button** — Desktop: clipboard copy with toast. Mobile (via Web Share API): system share sheet appears. Share icon shows SVG share icon + "Share" label.
23. **Mobile (DevTools responsive @ 375px)** — Preview is centered, action buttons are a horizontal row below the preview at 6-1-6-6 ratio, presets appear at the top of the panel and scroll with parameters, presets are hidden on the Export/Import tab.
24. **Mobile (DevTools responsive @ 1024px)** — Presets list on the left, action buttons below it in the same column, large preview on the right.

Any failure here is a release blocker.

---

## 7. When in doubt

- Re-read the section of `Architecture.md` covering the area you're changing.
- Check `Schemas.md` for the source-of-truth data shape.
- Run the acceptance smoke test (§6) before declaring "done."
- If a behavior surprises you, suspect history sync first (`set` vs raw `setSettings`) and SVG/canvas drift second.
