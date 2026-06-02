# Plan.md

Implementation checklist, testing criteria, and forward roadmap. Reflects state as of **2026-06-01 (post-Wave 4 F13b)**.

This document is the source of truth for *what's shipped*, *what's pending*, and *what passes*. Update it whenever you complete or change a feature.

---

## 1. Current state — what's shipped

### Core rendering

- [x] Canvas-based preview at 700x700 internal resolution
- [x] CSS-scaled responsive preview surface (200px mobile, up to 600px desktop)
- [x] Nine shapes: none, circle, square, rounded-square, diamond, hexagon, triangle, pentagram, hexagram
- [x] "None" shape option — transparent background (no shape, texture, rings, or outline)
- [x] Five texture overlays: none, crosshatch, dots, grid, lines
- [x] Linear + radial gradient support for background, big letter, and name text
- [x] Independent gradient angle and stop position per channel
- [x] Solid/gradient toggle per color channel (`bgMode`, `kMode`, `nameMode`) — renderer branches on mode key
- [x] Outer ring (`ring1`) with toggle, color, width
- [x] Inner ring (`ring2`) with toggle, color, width
- [x] Gap-from-outer control for inner ring (`ring1Gap`)
- [x] Glow effect with toggle, color, intensity (0.2–3.0)
- [x] Drop shadow effect with toggle, offset X/Y, blur, color, opacity (big letter + name text)
- [x] Name glow effect with toggle, color, intensity
- [x] Name shadow effect with toggle, offset X/Y, blur, color, opacity
- [x] Outline stroke around shape perimeter (color + width)
- [x] Per-letter positional fine-tune (`letterOffsetX`, `letterOffsetY`, `nameOffsetY`)
- [x] Per-character letter spacing on name text
- [x] Twenty-four fonts, self-hosted (woff2 in `fonts/`) — all SIL Open Font License
- [x] Font-loading gate to prevent fallback-font flash on preview and preset thumbnails

### Export

- [x] PNG download (700×700 / 1400×1400 / 2800×2800) via `drawMonogram` + `canvas.toDataURL`
- [x] SVG download (vector) via hand-built template string in `exportSVG`
- [x] Font-independent SVG export — text converted to `<path>` elements at export time via Typr.js; opens identically in any browser/design tool with no font dependency
- [x] Filename pattern: `monogram_<bigLetter>_<name>.<ext>`
- [x] Texture overlays in SVG export — crosshatch, dots, grid, lines (clipped to shape)

### Persistence

- [x] Auto-save to `localStorage["monogram-studio-settings"]` on every change (**throttled** — writes at most every 50ms via `useDebounce`)
- [x] Shareable URL via `#config=<base64-encoded-JSON>` hash
- [x] JSON export / import with shape validation (`parsed.bgTop !== undefined` gate)
- [x] Initial-load precedence: URL hash > localStorage > `DEFAULT_SETTINGS`

### Presets

- [x] Six shipped presets (Royal Gold, Midnight Aurora, Crimson Ember, Arctic Silver, Forest Obsidian, Neon Noir) rendered via `SHIPPED_PRESETS` constant
- [x] 10-slot preset store (`presetSlots` state) — up to 10 user-saved slots, first 6 pre-filled from shipped presets
- [x] Slot save/delete/reset-to-shipped actions with UI confirmation modal
- [x] Preset bundle JSON export (`exportPresets`) / import (`importPresets`)
- [x] Preset thumbnails render the preset's actual design in miniature (22×22px)
- [x] Active-preset detection (`name === p.name && bgTop === p.bgTop`)
- [x] One-click apply via `applyPreset` with proper history push

### Workflow

- [x] Undo / redo with up to 100 history entries
- [x] Keyboard shortcuts: Ctrl/Cmd+Z (undo), Ctrl/Cmd+Y or Ctrl/Cmd+Shift+Z (redo)
- [x] Randomize button with HSL-based palette generation (analogous / complementary / monochromatic)
- [x] Toast feedback for share link copy
- [x] Inline error message on JSON import failure
- [x] Live JSON preview of current settings in Export/Import tab
- [x] Reset-to-default button in header (left of undo/redo)
- [x] High-resolution PNG export (1×/2×/4× via offscreen render + upscale)
- [x] Keyboard shortcut hints on undo/redo (desktop only, CSS tooltip)
- [x] URL-length guard on Share (>8KB warns, offers JSON export fallback)

### Layout

- [x] Desktop: CSS Grid app shell, 1fr + 380px right panel, full-height panel
- [x] Desktop: presets in collapsible sidebar section (collapsed by default)
- [x] Desktop: preview goes up to 600px wide
- [x] Desktop: floating action overlay on canvas (PNG / SVG / Share), semi-transparent, darker on hover
- [x] Mobile (≤768px): single-column app, canvas-area on top, panel below
- [x] Mobile: centered 200px preview, floating action overlay at top-right (always opaque)
- [x] Mobile: tap-to-hide overlay on canvas (tap again to restore)
- [x] Mobile: presets render at the top of the panel in collapsible section (collapsed by default)
- [x] Mobile: presets hidden on Export/Import tab
- [x] Small-phone breakpoint (≤380px): preview shrinks to 180px
- [x] `display: contents` mobile sidebar collapse
- [x] Wide/low-viewport mobile breakpoint (≤768px and min-aspect-ratio ≥ 0.55)

### UI Components

- [x] `ColorRow` — label + native `<input type="color">` + hex readout (legacy; replaced by `ColorPickerRow` in UI)
- [x] `ColorPickerRow` — hex swatch + HSV wheel popover (200×200 canvas) + HEX/R/G/B inputs, click-outside/ESC to close
- [x] `SliderRow` — label + range input + numeric readout with unit
- [x] `PresetThumb` — miniature canvas render of a preset design (22×22px)
- [x] Collapsible `<Section>` component with persisted open/closed state via `useUIState`. **All sections default to collapsed** (`defaultOpen` defaults to `false`; no section overrides it). Manually expanded sections persist per-section in UI state.
- [x] Tab bar — internal keys (`design`, `colors`, `effects`, `randomizer`, `export/import`) mapped to concise display labels (`Design`, `Colors`, `Effects`, `Randomize`, and a two-line `Export / Import`). `.tab` uses `flex:1 1 0; min-width:0; white-space:nowrap` with reduced letter-spacing; the `export/import` tab gets `.tab-multiline` (9px, line-height 1.1) to wrap onto two lines. Mobile shrinks tabs to 9px.
- [x] Mobile Presets `<Section>` present at the top of **every** tab (Design, Colors, Effects, Randomize, Export/Import) for consistency.
- [x] Randomizer tab — per-category lock toggles (content, font, shape, texture, bgColor, letterColor, nameColor, outline, rings, typography, effects), persisted to `randomizerLocks` in UI state; `handleRandomize` strips locked categories; "Reset Locks" button clears all locks
- [x] Effects tab — houses Glow, Name Glow, Drop Shadow, Name Shadow

### Accessibility

- [x] `aria-label` on icon-only buttons
- [x] `htmlFor` linkage on all slider rows
- [x] `aria-pressed` on toggle and font/shape pickers
- [x] `aria-selected` on tab buttons
- [x] `role="status"` + `aria-live="polite"` on share toast
- [x] `role="alert"` on share warning (URL too long)
- [x] `role="alert"` on import error
- [x] Keyboard-accessible undo/redo

---

## 2. Acceptance testing criteria

Use these as pass/fail gates before merging changes. Manual; no automated runner.

### A. Initial state

| ID | Test | Pass criteria |
|---|---|---|
| A1 | Open `index.html` with empty localStorage and no URL hash | Preview shows `E EXAMPLE` in the default purple/gold style. No console errors. |
| A2 | Open with a valid URL hash | Preview reflects the hash's design, ignoring localStorage. |
| A3 | Open after a previous session | Preview reflects the localStorage state. |
| A4 | Open with an invalid URL hash (`#config=garbage`) | App falls through to localStorage or default; no crash. |

### B. Rendering

| ID | Test | Pass criteria |
|---|---|---|
| B1 | Cycle each shape | All 9 shapes render. "None" shows transparent background. Others have matching outline and rings. |
| B2 | Cycle each font | All 24 fonts visibly differ. None fall back to a generic system font (each woff2 must have full basic-latin coverage). |
| B3 | Cycle each texture | 4 textures show a clearly different overlay pattern; `none` shows no overlay. |
| B4 | Toggle outer ring | Ring appears / disappears. Inner ring's gap reference behavior is unaffected. |
| B5 | Toggle inner ring | Inner ring appears / disappears. |
| B6 | Toggle glow | Halo appears / disappears around the big letter. |
| B7 | Move letter offset sliders | Big letter shifts smoothly within its range. |
| B8 | Move gradient angle | Linear gradient rotates visibly; radial unchanged. |
| B9 | Set big letter to 1 char, then 2 chars | Both render correctly; 2-char layout stays centered. |

### C. Persistence

| ID | Test | Pass criteria |
|---|---|---|
| C1 | Change a setting, refresh page | Setting is preserved. |
| C2 | Click Share, paste URL in new incognito tab | Same design renders. |
| C3 | Export JSON, clear localStorage, refresh, import JSON | Design fully restored. |
| C4 | Import malformed JSON file | Inline error appears, no crash. |
| C5 | Import valid JSON missing some keys | App merges with defaults; no missing-value error. |

### D. Undo / redo

| ID | Test | Pass criteria |
|---|---|---|
| D1 | Make 5 changes, undo 5 times | Returns to initial state. |
| D2 | Make 5 changes, undo 3 times, make a new change | Forward history is sliced — redo button is disabled. |
| D3 | Apply a preset, undo | Returns to pre-preset state. |
| D4 | Randomize, undo | Returns to pre-randomize state. |
| D5 | Make 110 changes | History caps at 100; oldest entries shifted out cleanly. |
| D6 | Ctrl+Z / Ctrl+Y / Ctrl+Shift+Z | All three keyboard shortcuts work. |

### E. Export

| ID | Test | Pass criteria |
|---|---|---|
| E1 | Download PNG with default design | File saves as `monogram_E_EXAMPLE.png` and opens at 700x700. |
| E2 | Download SVG with default design | File saves as `.svg`; opens in a browser tab; visually matches the canvas. |
| E3 | Download SVG with complex design (Neon Noir + hexagram + glow) | Same visual match. |
| E4 | Compare PNG and SVG side-by-side for the same complex design | No noticeable divergence in shape, color, gradient, ring, glow, or texture placement. |

### F. Layout

| ID | Test | Pass criteria |
|---|---|---|
| F1 | Desktop @ 1440px width | Three-column-ish look: presets+actions sidebar (left), large preview (center-right), parameter panel (far right). |
| F2 | Desktop @ 1024px width | Same structure; nothing overflows the viewport. |
| F3 | Mobile @ 375px | Preview centered at top, action buttons as horizontal row below preview, panel below. |
| F4 | Mobile @ 375px, Export/Import tab | Presets are not rendered above the export controls. |
| F5 | Mobile @ 375px, Design tab | Presets are rendered at the top of the panel and scroll along with the sliders. |
| F6 | Small phone @ 360px | Preview shrinks to 180px, no horizontal scroll. |

### G. Randomize

| ID | Test | Pass criteria |
|---|---|---|
| G1 | Click Randomize 10 times | Each output is visually cohesive — not random clashing colors. |
| G2 | Inspect outputs | At least one example each of analogous (similar hues), complementary (high-contrast), and monochromatic (single hue) over 20 rolls. |
| G3 | Randomize, undo | History pops back to the pre-randomize state. |
| G4 | Lock one or more categories, randomize | Locked categories keep their exact values; unlocked categories reroll. Locks persist across reload. |
| G5 | Lock categories, click "Reset Locks" | All toggles flip off; persists across reload. Button is disabled/greyed when nothing is locked. |

---

## 3. Known issues / divergences

- **~~Auto-save is unthrottled.~~** **Fixed in F2: 50ms debounce.**
- **~~Typr.js loaded from `@master`.~~** **Fixed in F1: pinned to `02c12105`.**
- **No mobile native share.** Share button only copies to clipboard, even on mobile.
- **Undo/redo/reset buttons have no visual accent** — ~~they blend in with generic icon buttons.~~ **Fixed in F4: yellow accent.**
- **Mobile presets render from a second copy** of `PRESETS.map(...)`. Acceptable, but duplicates code.
- **Glow color is inline in the Colors tab** (Big Letter section) rather than grouped with glow controls.

---

## 4. Roadmap

### Wave 1 — Hygiene + small wins

| # | Feature | Effort | Status |
|---|---|---|---|
| F1 | Pin Typr.js to a commit hash (`02c12105`) | S | [x] |
| F2 | Throttle `localStorage` writes (300ms debounce) | S | [x] |
| F3 | Web Share API on mobile | S | [x] |
| F4 | Yellow accent highlight on undo / redo / reset | S | [x] |

### Wave 2 — Layout restructure

| # | Feature | Effort | Status |
|---|---|---|---|
| F5 | Pinned floating action panel on canvas (PNG / SVG / Share) | M | [x] |
| F6 | Collapsible accordion options sections (persisted) | M | [x] |
| F7 | Smaller, collapsible presets list | S | [x] |
| F8a | Randomizer tab — placeholder | S | [x] |

### Wave 3 — Preset system + color system

| # | Feature | Effort | Status |
|---|---|---|---|
| F10 | 10-slot preset system — replace shipped defaults, save, import/export bundle, storage warning | L | [x] |
| F11 | Color picker wheel + hex/RGB inputs | L | [x] |
| F12 | Gradient ↔ solid toggle per color slot | M | [x] |

### Wave 4 — Typography polish

| # | Feature | Effort | Status |
|---|---|---|---|
| F13a | Font grid → 3-column compact button grid | S | [x] |
| F13b | Expand to 24 curated fonts (SIL OFL); re-sourced corrupt woff2 files; swapped 5 low-variety fonts (Montserrat/Oswald/Lobster/Raleway/Josefin) for Bungee/Monoton/Bangers/Rye/Silkscreen | M | [x] |

### Wave 5 — Effects foundation

| # | Feature | Effort | Status |
|---|---|---|---|
| F14 | Effects tab — move Glow into it | M | [x] |
| F15 | Drop shadow effect | M | [x] |

**Wave 5 post-implementation fixes (2026-06-02):**
- SVG filter chaining was broken — nested `<g filter>` caused shadow `feFlood` to tint glow output. Fixed by rendering 3 independent layers (shadow-only, glow-only, text) when both effects are active.
- SVG glow intensity didn't match canvas — `feFlood` + `feComposite IN` creates solid colored text shape, unlike canvas transparent fill. Fixed with `feComposite operator="out"` to subtract source shape from blur, leaving only halo.
- SVG glow was clipping — filter region too small. Fixed by increasing to `x="-200%" y="-200%" width="500%" height="500%"`.
- Canvas glow too intense / too soft — bumped alpha values (0.75–1.0 per pass) to match SVG `operator="out"` concentration.
- Name shadow thickness mismatch — SVG `SourceAlpha` includes stroke (too thick), canvas `fillText` excludes stroke (too thin). Fixed with half-stroke (`outlineWidth`) in both.
- Name shadow/glow stacking — swapped canvas draw order so glow renders on top of shadow.
- Canvas name shadow missing stroke — added `strokeText` before `fillText` in shadow pass.

See `Architecture.md` §12 for full lessons learned.

### Wave 6 — Randomizer completion

| # | Feature | Effort | Status |
|---|---|---|---|
| F8b | Randomizer tab — full toggle UI (one switch per randomizable category) | L | [x] |
| F9 | Lock-colors toggle on Randomize (folded into F8b) | (in F8b) | [x] |
| F8c | Randomizer tab — "Reset Locks" button (clears all category locks) | S | [x] |

### Wave 7 — Bonus features (now in-scope; detailed subtasks in `Sprint-Plan.md` §5b)

Promoted from "deferred" after Waves 1–6 completed in a single day. Implement in order F19 → F18 → F16 → F17 (ascending render-path risk).

| # | Feature | Effort | Status |
|---|---|---|---|
| F19 | Letter transformations (scale X/Y, skew X/Y, rotate) | M | [ ] |
| F18 | Emboss / bevel effect (big letter) | M | [ ] |
| F16 | Blend mode + opacity on big letter | M | [ ] |
| F17 | Layer stack — visibility toggles + reorder | XL | [ ] |

### Deferred beyond sprint

| # | Feature | Why deferred |
|----|---|---|
| F20 | Optional: `fonts/manifest.json` for drop-in font workflow | Nice-to-have; doesn't block sprint goals |
| F21 | Multi-line `name` text + leading control | Schema reservation only; build deferred |
| R1 | Glow/shadow parity hardening (canvas vs SVG) | Cosmetic-only; current output matches at normal settings. Unify shadow-blur convention (#2), drive glow from a shared `{radius, canvasAlpha, svgOpacity}` table (#3), and scale all glow passes by intensity identically (#4). See `Architecture.md` §14. |

---

## 5. Implementation checklist template

When picking up a roadmap item, work through this checklist:

```
- [ ] Read the relevant section of Architecture.md
- [ ] Check Schemas.md for any data shape changes needed
- [ ] Make canvas changes (drawMonogram)
- [ ] Mirror in SVG (exportSVG) — if visual change
- [ ] Add/update UI controls in the appropriate tab
- [ ] Update DEFAULT_SETTINGS if a new key was added
- [ ] Update handleRandomize if the new key should be randomized
- [ ] Update SHIPPED_PRESETS if affected (note: preset slots handle user customizations)
- [ ] Run acceptance smoke test (Agents.md §6)
- [ ] Run targeted acceptance criteria (Plan.md §2)
- [ ] Update Schemas.md
- [ ] Mark item complete in Plan.md §1
- [ ] If applicable, move item from §4 to §1
```

---

## 6. Migrations

Track any settings-key additions or renames here so old shared URLs continue to work.

| ID | Feature | Keys | Behaviour |
|---|---|---|---|
| M1 | F12 | `bgMode`, `kMode`, `nameMode` | Defaults each to `"gradient"` when absent. Old data renders identically (gradient was the only mode pre-F12). |
| M2 | F14 | `effects` sub-object (`glow`, `shadow`, `nameGlow`, `nameShadow`) | Initialises all four effect sub-objects with safe defaults when absent. Effect settings are the sole source of truth — legacy top-level `glow`/`glowColor`/`glowIntensity` keys are no longer referenced. |
| M3 | F19 | `letterScaleX`, `letterScaleY`, `letterSkewX`, `letterSkewY`, `letterRotate` | Defaults each to identity (1.0 or 0) when absent. Old data renders identically (no transform). |
| M4 | F18 | `effects.emboss` | Initialises emboss sub-object with safe defaults when absent. Emboss off by default. |
| M5 | F16 | `letterBlendMode`, `letterOpacity` | Defaults to `"normal"` / `1.0` when absent. Old data renders identically (no blend, full opacity). |
| M6 | F17 | `layers` | Validates and normalizes the 4-layer array: drops unknown ids, appends missing layers, defaults all visible. |
