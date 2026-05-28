# Plan.md

Implementation checklist, testing criteria, and forward roadmap. Reflects state as of **2026-05-28**.

This document is the source of truth for *what's shipped*, *what's pending*, and *what passes*. Update it whenever you complete or change a feature.

---

## 1. Current state — what's shipped

### Core rendering

- [x] Canvas-based preview at 700x700 internal resolution
- [x] CSS-scaled responsive preview surface (200px mobile, up to 600px desktop)
- [x] Eight shapes: circle, square, rounded-square, diamond, hexagon, triangle, pentagram, hexagram
- [x] Five texture overlays: none, crosshatch, dots, grid, lines (canvas only)
- [x] Linear + radial gradient support for background, big letter, and name text
- [x] Independent gradient angle and stop position per channel
- [x] Outer ring (`ring1`) with toggle, color, width
- [x] Inner ring (`ring2`) with toggle, color, width
- [x] Gap-from-outer control for inner ring (`ring1Gap`)
- [x] Glow effect with toggle, color, intensity (0.2–3.0)
- [x] Outline stroke around shape perimeter (color + width)
- [x] Per-letter positional fine-tune (`letterOffsetX`, `letterOffsetY`, `nameOffsetY`)
- [x] Per-character letter spacing on name text
- [x] Eight fonts, self-hosted (woff2 in `fonts/`)
- [x] Font-loading gate to prevent fallback-font flash on preview and preset thumbnails

### Export

- [x] PNG download (700x700 raster) via `canvas.toDataURL`
- [x] SVG download (vector) via hand-built template string in `exportSVG`
- [x] Filename pattern: `monogram_<bigLetter>_<name>.<ext>`
- [ ] Texture overlays in SVG export — **not implemented**. Texture is canvas-only. (`Schemas.md` §4)

### Persistence

- [x] Auto-save to `localStorage["monogram-studio-settings"]` on every change
- [x] Shareable URL via `#config=<base64-encoded-JSON>` hash
- [x] JSON export / import with shape validation (`parsed.bgTop !== undefined` gate)
- [x] Initial-load precedence: URL hash > localStorage > `DEFAULT_SETTINGS`

### Presets

- [x] Six built-in presets: Royal Gold, Midnight Aurora, Crimson Ember, Arctic Silver, Forest Obsidian, Neon Noir
- [x] Preset thumbnails render the preset's actual design in miniature
- [x] Active-preset detection (`name === p.name && bgTop === p.bgTop`)
- [x] One-click apply via `applyPreset` with proper history push

### Workflow

- [x] Undo / redo with up to 100 history entries
- [x] Keyboard shortcuts: Ctrl/Cmd+Z (undo), Ctrl/Cmd+Y or Ctrl/Cmd+Shift+Z (redo)
- [x] Randomize button with HSL-based palette generation (analogous / complementary / monochromatic)
- [x] Toast feedback for share link copy
- [x] Inline error message on JSON import failure
- [x] Live JSON preview of current settings in Export/Import tab

### Layout

- [x] Desktop: CSS Grid app shell, 1fr + 380px right panel, full-height panel
- [x] Desktop: presets and action buttons share the left sidebar with a 22px gap between them
- [x] Desktop: preview goes up to 600px wide
- [x] Mobile (≤768px): single-column app, canvas-area on top, panel below
- [x] Mobile: centered 200px preview, horizontal action button row below
- [x] Mobile: presets render at the top of the panel and scroll with parameters
- [x] Mobile: presets hidden on Export/Import tab
- [x] Small-phone breakpoint (≤380px): preview shrinks to 180px
- [x] `display: contents` mobile sidebar collapse for clean action-button promotion

### Accessibility

- [x] `aria-label` on icon-only buttons
- [x] `htmlFor` linkage on all slider rows
- [x] `aria-pressed` on toggle and font/shape pickers
- [x] `aria-selected` on tab buttons
- [x] `role="status"` + `aria-live="polite"` on share toast
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
| B1 | Cycle each shape | All 8 shapes render. Each has a matching outline and rings. |
| B2 | Cycle each font | All 8 fonts visibly differ. None fall back to a generic system font. |
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
| E4 | Compare PNG and SVG side-by-side for the same complex design | No noticeable divergence in shape, color, gradient, ring, or glow placement. (Texture acceptable to differ — known canvas-only.) |

### F. Layout

| ID | Test | Pass criteria |
|---|---|---|
| F1 | Desktop @ 1440px width | Three-column-ish look: presets+actions sidebar (left), large preview (center-right), parameter panel (far right). |
| F2 | Desktop @ 1024px width | Same structure; nothing overflows the viewport. |
| F3 | Mobile @ 375px | Preview centered at top, action buttons horizontal row below, panel below that. |
| F4 | Mobile @ 375px, Export/Import tab | Presets are not rendered above the export controls. |
| F5 | Mobile @ 375px, Design tab | Presets are rendered at the top of the panel and scroll along with the sliders. |
| F6 | Small phone @ 360px | Preview shrinks to 180px, no horizontal scroll. |

### G. Randomize

| ID | Test | Pass criteria |
|---|---|---|
| G1 | Click Randomize 10 times | Each output is visually cohesive — not random clashing colors. |
| G2 | Inspect outputs | At least one example each of analogous (similar hues), complementary (high-contrast), and monochromatic (single hue) over 20 rolls. |
| G3 | Randomize, undo | History pops back to the pre-randomize state. |

---

## 3. Known issues / divergences

- **SVG texture export missing.** Texture overlays appear in the canvas preview and PNG but not in the SVG download. Decide before fix: do we match canvas exactly, or document the SVG as "no texture overlay (use the PNG for textured designs)"?
- **Texture overlay positioning** is computed in canvas pixels; behavior at extreme `letterOffsetX/Y` values has not been visually audited.
- **Auto-save is unthrottled.** Every keystroke writes localStorage. Fine in practice but a refactor candidate if state grows.
- **`useDebounce` delay is hardcoded to `0`.** The hook exists for future use; no effect today.
- **Share URLs can get long** for complex designs. No length check; URLs over the browser's hash limit may truncate silently. Worth adding a length warning if reports come in.
- **Mobile-presets-wrap rendering** duplicates the `PRESETS.map(...)` block. Acceptable, but if presets grow into a heavier component, extract a `PresetGrid` wrapper.

---

## 4. Roadmap — proposed work

### P0 — bugs and parity

1. **Fix SVG texture export** so PNG and SVG match for all texture choices. Touches `exportSVG` and `drawTexture` — likely needs a parallel `buildTextureSVGFragment(texture, W, H)` helper.
2. **Add a URL-length guard** on the Share action — if `encodeSettings(s)` exceeds ~8KB, surface a warning toast and offer JSON export as the fallback.

### P1 — UX improvements

3. **Persist tab choice** across reloads (currently always lands on "design").
4. **Add a "Reset to default" button** in the Design tab — clears overrides back to `DEFAULT_SETTINGS` with a single history step.
5. **Preset hover preview on desktop** — show the preset name and a 1-line description on hover.
6. **Keyboard shortcut hints** — small tooltip on the undo/redo icons displaying Ctrl/Cmd+Z behavior.

### P2 — new features

7. **Custom preset save** — let the user pin their current design as a named preset, stored in localStorage.
8. **Multi-letter typography** — kerning slider for the big letter when two characters are used.
9. **Image background** — option to upload a small image as the background (replaces gradient). Stays client-side as a data-URL embedded in `settings`.
10. **High-resolution PNG** — option to render at 1400x1400 or 2100x2100 for print-quality output.

### P3 — refactors

11. **Extract `<PresetGrid />`** if mobile and desktop preset rendering diverge.
12. **Migration framework** in `getInitialSettings` for handling renamed/removed settings keys.
13. **Type annotations** via JSDoc — would help downstream agentic coders without breaking the "no build step" constraint.

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
- [ ] Update PRESETS if affected
- [ ] Run acceptance smoke test (Agents.md §6)
- [ ] Run targeted acceptance criteria (Plan.md §2)
- [ ] Update Schemas.md
- [ ] Mark item complete in Plan.md §1
- [ ] If applicable, move item from §4 to §1
```

---

## 6. Migrations

Track any settings-key renames or removals here so old shared URLs continue to work.

*(none currently)*

When you add one, follow the format:

```
### 2026-MM-DD: rename oldKey -> newKey

Reason: ...
Migration: see getInitialSettings()
Deprecation window ends: 2027-MM-DD (remove migration after this date)
```
