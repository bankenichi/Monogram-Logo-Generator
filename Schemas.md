# Schemas

Authoritative data shapes for Monogram Studio. Anything that crosses a persistence boundary (localStorage, URL hash, JSON file) goes through these schemas.

If a key isn't documented here, it doesn't exist in the application. If a value is out of the documented range, the canvas may render incorrectly or the SVG export may diverge from the PNG.

---

## 1. `Settings`

The single object that describes a complete design. Every key has a default in `DEFAULT_SETTINGS`. Every key participates in autosave, history, URL hash, and JSON export.

### Content

| Key | Type | Range / Values | Default | Notes |
|---|---|---|---|---|
| `bigLetter` | string | 1–2 chars | `"E"` | Upper-cased on input. Rendered as the centerpiece. |
| `name` | string | any | `"EXAMPLE"` | Lower-badge text. No case forcing on input. |

### Geometry

| Key | Type | Range / Values | Default | Notes |
|---|---|---|---|---|
| `shape` | enum | `circle`, `square`, `rounded-square`, `diamond`, `hexagon`, `triangle`, `pentagram`, `hexagram` | `"circle"` | Both clip and outer outline use this. |
| `texture` | enum | `none`, `crosshatch`, `dots`, `grid`, `lines` | `"none"` | Low-opacity overlay drawn inside the shape clip. |
| `fontFamily` | enum | One of `FONTS[*].family` (see §2) | `"Poppins"` | Must match an entry in the `FONTS` array exactly. |

### Background gradient

| Key | Type | Range | Default | Notes |
|---|---|---|---|---|
| `bgTop` | hex color | `#rrggbb` | `"#160e3c"` | First stop. |
| `bgBot` | hex color | `#rrggbb` | `"#0a0624"` | Second stop. |
| `bgGradientType` | enum | `linear`, `radial` | `"linear"` | Radial ignores angle. |
| `bgGradientAngle` | number | 0–360 | `90` | Degrees. `0` = right, `90` = down. |
| `bgGradientStop` | number | 10–100 | `100` | Position of `bgBot` along the gradient, as a percentage. |

### Big-letter gradient

| Key | Type | Range | Default |
|---|---|---|---|
| `kTop` | hex color | `#rrggbb` | `"#ffdc50"` |
| `kBot` | hex color | `#rrggbb` | `"#c88c14"` |
| `kGradientType` | enum | `linear`, `radial` | `"linear"` |
| `kGradientAngle` | number | 0–360 | `90` |
| `kGradientStop` | number | 10–100 | `100` |

### Name-text gradient

| Key | Type | Range | Default |
|---|---|---|---|
| `nameTop` | hex color | `#rrggbb` | `"#ffc828"` |
| `nameBot` | hex color | `#rrggbb` | `"#ffc828"` |
| `nameGradientType` | enum | `linear`, `radial` | `"linear"` |
| `nameGradientAngle` | number | 0–360 | `90` |
| `nameGradientStop` | number | 10–100 | `100` |

### Outline (perimeter stroke of the shape)

| Key | Type | Range | Default |
|---|---|---|---|
| `outlineColor` | hex color | `#rrggbb` | `"#100a2e"` |
| `outlineWidth` | number | 0–8 | `2` | px in the 700-unit canvas space. `0` hides it. |

### Rings

| Key | Type | Range / Values | Default | Notes |
|---|---|---|---|---|
| `showRing1` | boolean | — | `true` | Outer ring. |
| `showRing2` | boolean | — | `true` | Inner ring. |
| `ring1` | hex color | `#rrggbb` | `"#ffc828"` | Outer ring color. |
| `ring2` | hex color | `#rrggbb` | `"#643cb4"` | Inner ring color. |
| `ring1Width` | number | 1–20 | `5` | Outer ring stroke width. |
| `ring2Width` | number | 1–12 | `2` | Inner ring stroke width. |
| `ring1Gap` | number | 4–60 | `18` | Distance from outer to inner ring (px). Named `ring1Gap` for historical reasons — it sets the inner ring's offset from the outer. |

### Glow

| Key | Type | Range | Default | Notes |
|---|---|---|---|---|
| `glow` | boolean | — | `true` | Enable shadow passes around the big letter. |
| `glowColor` | hex color | `#rrggbb` | `"#ffdc50"` | Halo color. |
| `glowIntensity` | number | 0.2–3.0 (step 0.1) | `1.0` | Scales the number and blur radius of shadow passes. |

### Typography

| Key | Type | Range | Default | Notes |
|---|---|---|---|---|
| `letterSize` | number | 300–700 (step 10) | `650` | Font size of `bigLetter` in canvas px. |
| `nameSize` | number | 48–160 (step 4) | `84` | Font size of `name`. |
| `letterSpacing` | number | 0–60 | `28` | Per-character spacing on `name`. |
| `letterOffsetX` | number | -200–200 (step 5) | `0` | Horizontal nudge of `bigLetter` from center. |
| `letterOffsetY` | number | -200–200 (step 5) | `0` | Vertical nudge of `bigLetter` from center. |
| `nameOffsetY` | number | -200–200 (step 5) | `0` | Vertical nudge of `name` from its computed baseline. |

### Complete `DEFAULT_SETTINGS`

```js
const DEFAULT_SETTINGS = {
  bigLetter: "E",
  name: "EXAMPLE",
  shape: "circle",
  bgTop: "#160e3c",
  bgBot: "#0a0624",
  bgGradientType: "linear",
  bgGradientAngle: 90,
  bgGradientStop: 100,
  kTop: "#ffdc50",
  kBot: "#c88c14",
  kGradientType: "linear",
  kGradientAngle: 90,
  kGradientStop: 100,
  nameTop: "#ffc828",
  nameBot: "#ffc828",
  nameGradientType: "linear",
  nameGradientAngle: 90,
  nameGradientStop: 100,
  outlineColor: "#100a2e",
  ring1: "#ffc828",
  ring2: "#643cb4",
  ring1Width: 5,
  ring2Width: 2,
  ring1Gap: 18,
  showRing1: true,
  showRing2: true,
  glow: true,
  glowColor: "#ffdc50",
  glowIntensity: 1.0,
  letterSize: 650,
  nameSize: 84,
  letterSpacing: 28,
  outlineWidth: 2,
  nameOffsetY: 0,
  letterOffsetX: 0,
  letterOffsetY: 0,
  fontFamily: "Poppins",
  texture: "none",
};
```

---

## 2. `FONTS`

```js
const FONTS = [
  { label: "Poppins",        family: "Poppins",           weight: 700, style: "sans-serif" },
  { label: "Playfair SC",    family: "Playfair Display",  weight: 700, style: "serif" },
  { label: "Bebas Neue",     family: "Bebas Neue",        weight: 400, style: "display" },
  { label: "Cinzel",         family: "Cinzel",            weight: 700, style: "roman" },
  { label: "Dancing Script", family: "Dancing Script",    weight: 700, style: "script" },
  { label: "Space Mono",     family: "Space Mono",        weight: 700, style: "mono" },
  { label: "Cormorant",      family: "Cormorant Upright", weight: 700, style: "elegant" },
  { label: "Abril Fatface",  family: "Abril Fatface",     weight: 400, style: "display" },
];
```

- `label` — what the UI shows under the font preview.
- `family` — what gets used in CSS / canvas `font` strings. **Must match the `@font-face` `font-family` exactly.**
- `weight` — used when calling `document.fonts.load(...)` and inside `getFontString`.
- `style` — purely descriptive metadata; not used by the renderer.

To add a font:

1. Drop the woff2 into `fonts/`.
2. Add an `@font-face` declaration to the `<style>` block in `<head>`.
3. Append an entry to `FONTS`.
4. Verify font preview thumbnails render.

---

## 3. `SHAPES`

```js
const SHAPES = ["circle", "square", "rounded-square", "diamond", "hexagon", "triangle", "pentagram", "hexagram"];
```

Each shape has two parallel implementations:

- Canvas: `buildShapePath(ctx, shape, cx, cy, R)`, `clipShape(...)`, `strokeShape(...)`.
- SVG: `buildShapeSVGPath(shape, cx, cy, R)`.

Adding a shape means updating **both**. See `Agents.md` §"Add a new shape" for the step-by-step.

---

## 4. `TEXTURES`

```js
const TEXTURES = ["none", "crosshatch", "dots", "grid", "lines"];
```

Drawn by `drawTexture(ctx, texture, W, H)` as low-alpha overlays inside the shape clip. `"none"` short-circuits and draws nothing.

Textures are exported to SVG via `buildTexturePattern()` inside `exportSVG`. Each texture maps to a `<pattern>` element with `patternUnits="userSpaceOnUse"`, clipped to the shape via `clip-path="url(#shapeClip)"`. The SVG texture overlay matches the canvas rendering.

---

## 5. `PRESETS`

Array of partial `Settings` objects. Each preset only specifies the keys it cares about; missing keys keep the user's current value. Order matters — it's the display order in both the desktop sidebar and the mobile panel.

```js
const PRESETS = [
  { bigLetter: "RG", name: "Royal Gold",     shape: "circle", bgTop: "#160e3c", ... },
  { bigLetter: "MA", name: "Midnight Aurora", ... },
  { bigLetter: "CE", name: "Crimson Ember",   ... },
  { bigLetter: "AS", name: "Arctic Silver",   ... },
  { bigLetter: "FO", name: "Forest Obsidian", ... },
  { bigLetter: "NN", name: "Neon Noir",       ... },
];
```

Every preset must include at minimum:

- `bigLetter` — the preview thumbnail uses this.
- `name` — used as the preset's display label *and* as the lower-badge text when applied. Treat it as user-visible.
- `bgTop` — the active-preset check in JSX uses `settings.name === p.name && settings.bgTop === p.bgTop`. If two presets share both, only the first will appear active.

A preset's `name` doubles as its identity key. Don't change a preset's name after release — it will silently break shareable URLs that depend on equality with the active-state check.

**Active-preset detection rule** (literal code):

```js
active={settings.name === p.name && settings.bgTop === p.bgTop}
```

---

## 6. URL hash format

```
#config=<base64>
```

- `<base64>` = `btoa(JSON.stringify(settings))`.
- Decoded by `decodeSettings(str) = JSON.parse(atob(str))`, wrapped in try/catch.
- The decoded object is merged with `DEFAULT_SETTINGS`:

  ```js
  const decoded = decodeSettings(hash.slice(7));
  if (decoded && decoded.bgTop) return { ...DEFAULT_SETTINGS, ...decoded };
  ```

  The `decoded.bgTop` truthy check is a cheap "this looks like a real settings object" guard. **Do not remove `bgTop` from `DEFAULT_SETTINGS` or the share-link load logic will reject all incoming hashes.**

- Priority: URL hash > localStorage > `DEFAULT_SETTINGS`.
- The hash is **not** updated as the user edits — it's only written when the user clicks Share. Visiting a shared link then making edits invisibly shifts you off the hash; refreshing brings the original hash back.

### Forward compatibility

When you add a new key to `DEFAULT_SETTINGS`, old shared links will continue to work because the merge with `DEFAULT_SETTINGS` fills in the missing key. **Removing or renaming a key breaks old links.** If you must rename, write a migration in `getInitialSettings()` that maps the old key to the new one.

---

## 7. JSON export format

```json
{
  "bigLetter": "E",
  "name": "EXAMPLE",
  "shape": "circle",
  ...all keys from DEFAULT_SETTINGS...
}
```

- Pretty-printed with `JSON.stringify(settings, null, 2)`.
- File name: `monogram_<bigLetter>_<name>.json`.
- Import path: file is read as text, parsed, validated (`parsed.bgTop !== undefined`), then merged with current settings via `{...current, ...parsed}` and pushed onto the history.
- Import rejects on parse error or missing `bgTop` and surfaces `importError` in the UI.

The JSON export is the most stable persistence channel — there's no base64 encoding to break, no URL length limit, and no localStorage capacity limit. Recommend it as the canonical backup format.

---

## 8. localStorage entry

| Key | Value | Written by | Read by |
|---|---|---|---|
| `monogram-studio-settings` | `JSON.stringify(settings)` | `useEffect [settings]` | `getInitialSettings()` on mount |

The write is unthrottled — every keystroke that changes `settings` writes to localStorage. This is fine in practice (the writes are small and localStorage is synchronous-fast) but could become a hot spot if `settings` grows substantially.

---

## 9. Quick validation checklist

When you change anything in `Settings`, verify:

- [ ] Key is in `DEFAULT_SETTINGS`.
- [ ] Key has a UI control somewhere in the design / colors / export panel.
- [ ] Key is read by `drawMonogram`.
- [x] Key is read by `exportSVG` (all visual settings including textures are now exported).
- [ ] Range matches the slider min/max in the design panel JSX.
- [ ] `handleRandomize` emits a sane value for the key (or you've decided to skip randomizing it).
- [ ] A round-trip JSON export + import preserves the value.
- [ ] A share-link round-trip preserves the value.
