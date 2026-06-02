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
| `shape` | enum | `none`, `circle`, `square`, `rounded-square`, `diamond`, `hexagon`, `triangle`, `pentagram`, `hexagram` | `"circle"` | `"none"` disables background, texture, rings, and outline. |
| `texture` | enum | `none`, `crosshatch`, `dots`, `grid`, `lines` | `"none"` | Low-opacity overlay drawn inside the shape clip. |
| `fontFamily` | enum | One of `FONTS[*].family` (see §2) | `"Poppins"` | Must match an entry in the `FONTS` array exactly. |

### Background

| Key | Type | Range / Values | Default | Notes |
|---|---|---|---|---|
| `bgMode` | enum | `solid`, `gradient` | `"gradient"` | When solid, only `bgTop` is used. |
| `bgTop` | hex color | `#rrggbb` | `"#160e3c"` | First stop (solid color or gradient start). |
| `bgBot` | hex color | `#rrggbb` | `"#0a0624"` | Second stop. Ignored when `bgMode === "solid"`. |
| `bgGradientType` | enum | `linear`, `radial` | `"linear"` | Ignored when `bgMode === "solid"`. |
| `bgGradientAngle` | number | 0–360 | `90` | Degrees. `0` = right, `90` = down. Ignored when solid. |
| `bgGradientStop` | number | 10–100 | `100` | Position of `bgBot` along the gradient, as a percentage. Ignored when solid. |

### Big-letter

| Key | Type | Range / Values | Default | Notes |
|---|---|---|---|---|
| `kMode` | enum | `solid`, `gradient` | `"gradient"` | When solid, only `kTop` is used as the fill color. |
| `kTop` | hex color | `#rrggbb` | `"#ffdc50"` | Top color (solid fill or gradient start). |
| `kBot` | hex color | `#rrggbb` | `"#c88c14"` | Bottom color. Ignored when `kMode === "solid"`. |
| `kGradientType` | enum | `linear`, `radial` | `"linear"` | Ignored when solid. |
| `kGradientAngle` | number | 0–360 | `90` | Ignored when solid. |
| `kGradientStop` | number | 10–100 | `100` | Ignored when solid. |

### Name-text

| Key | Type | Range / Values | Default | Notes |
|---|---|---|---|---|
| `nameMode` | enum | `solid`, `gradient` | `"gradient"` | When solid, only `nameTop` is used as the fill color. |
| `nameTop` | hex color | `#rrggbb` | `"#ffc828"` | Solid color or gradient start. |
| `nameBot` | hex color | `#rrggbb` | `"#ffc828"` | Gradient end. Ignored when `nameMode === "solid"`. |
| `nameGradientType` | enum | `linear`, `radial` | `"linear"` | Ignored when solid. |
| `nameGradientAngle` | number | 0–360 | `90` | Ignored when solid. |
| `nameGradientStop` | number | 10–100 | `100` | Ignored when solid. |

### Outline

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
| `ring1Gap` | number | 4–60 | `18` | Distance from outer to inner ring (px). |

### Glow

| Key | Type | Range | Default | Notes |
|---|---|---|---|---|
*(Glow, glow color, and glow intensity are now inside `effects.glow` — see the Effects section below.)*

### Typography

| Key | Type | Range | Default | Notes |
|---|---|---|---|---|
| `letterSize` | number | 300–700 (step 10) | `650` | Font size of `bigLetter` in canvas px. |
| `nameSize` | number | 48–160 (step 4) | `84` | Font size of `name`. |
| `letterSpacing` | number | 0–60 | `28` | Per-character spacing on `name`. |
| `letterOffsetX` | number | -200–200 (step 5) | `0` | Horizontal nudge of `bigLetter`. |
| `letterOffsetY` | number | -200–200 (step 5) | `0` | Vertical nudge of `bigLetter`. |
| `nameOffsetY` | number | -200–200 (step 5) | `0` | Vertical nudge of `name`. |

### Effects (`effects` sub-object)

Renderer reads from `settings.effects.*` exclusively. All glow and shadow settings live in this sub-object.

| Path | Type | Default | Notes |
|---|---|---|---|
| `effects.glow` | `{enabled,color,intensity}` | `{true,"#ffdc50",1.0}` | Big-letter glow. |
| `effects.nameGlow` | `{enabled,color,intensity}` | `{true,"#ffc828",1.0}` | Name glow. |
| `effects.shadow` | `{enabled,dx,dy,blur,color,opacity}` | `{true,4,4,8,"#000000",0.5}` | Big-letter drop shadow. |
| `effects.nameShadow` | `{enabled,dx,dy,blur,color,opacity}` | `{true,3,3,6,"#000000",0.5}` | Name drop shadow. |
| `effects.emboss` | `{enabled,depth,angle,highlight,shadow,opacity}` | `{false,3,135,"#ffffff","#000000",0.6}` | Two-offset emboss (F18). |

### Wave 7 keys

Now shipped. All defaults are identity/no-op so old data is unaffected (all defaulted in `migrateSettings`).

| Key | Type | Range | Default | Feature | Notes |
|---|---|---|---|---|---|
| `letterScaleX` | number | 0.5–2.0 (0.05) | `1.0` | F19 | Big-letter non-uniform X scale. |
| `letterScaleY` | number | 0.5–2.0 (0.05) | `1.0` | F19 | Big-letter non-uniform Y scale. |
| `letterSkewX` | number | -45–45 (1, °) | `0` | F19 | Big-letter horizontal skew. |
| `letterSkewY` | number | -45–45 (1, °) | `0` | F19 | Big-letter vertical skew. |
| `letterRotate` | number | -180–180 (1, °) | `0` | F19 | Big-letter rotation about draw anchor. |
| `effects.emboss` | `{enabled,depth,angle,highlight,shadow,opacity}` | `{false,3,135,"#ffffff","#000000",0.6}` | F18 | Two-offset emboss on big letter. |
| `letterBlendMode` | string | `BLEND_MODES` | `"normal"` | F16 | Canvas `globalCompositeOperation` / SVG `mix-blend-mode`. |
| `letterOpacity` | number | 0–1 (0.05) | `1.0` | F16 | Big-letter fill opacity. |
| `layers` | `[{id,label,visible}]` | canonical 4-layer array | all visible | F17 | Draw order + per-layer visibility. Background pinned to index 0 in UI. |

### Complete `DEFAULT_SETTINGS`

```js
const DEFAULT_SETTINGS = {
  bigLetter: "E",
  name: "EXAMPLE",
  shape: "circle",
  bgMode: "gradient",
  bgTop: "#160e3c",
  bgBot: "#0a0624",
  bgGradientType: "linear", bgGradientAngle: 90, bgGradientStop: 100,
  kMode: "gradient",
  kTop: "#ffdc50",
  kBot: "#c88c14",
  kGradientType: "linear", kGradientAngle: 90, kGradientStop: 100,
  nameMode: "gradient",
  nameTop: "#ffc828",
  nameBot: "#ffc828",
  nameGradientType: "linear", nameGradientAngle: 90, nameGradientStop: 100,
  outlineColor: "#100a2e", outlineWidth: 2,
  ring1: "#ffc828", ring2: "#643cb4", ring1Width: 5, ring2Width: 2, ring1Gap: 18,
  showRing1: true, showRing2: true,
  effects: {
    glow:       { enabled: true,  color: "#ffdc50", intensity: 1.0 },
    nameGlow:   { enabled: true,  color: "#ffc828", intensity: 1.0 },
    shadow:     { enabled: true,  dx: 4, dy: 4, blur: 8, color: "#000000", opacity: 0.5 },
    nameShadow: { enabled: true, dx: 3, dy: 3, blur: 6, color: "#000000", opacity: 0.5 },
    emboss:     { enabled: false, depth: 3, angle: 135, highlight: "#ffffff", shadow: "#000000", opacity: 0.6 }
  },
  letterSize: 650, nameSize: 84, letterSpacing: 28,
  nameOffsetY: 0, letterOffsetX: 0, letterOffsetY: 0,
  fontFamily: "Poppins", texture: "none",
  letterScaleX: 1.0, letterScaleY: 1.0, letterSkewX: 0, letterSkewY: 0, letterRotate: 0,
  letterBlendMode: "normal", letterOpacity: 1.0,
  layers: [
    { id: "background", label: "Background", visible: true },
    { id: "rings",      label: "Rings",      visible: true },
    { id: "bigLetter",  label: "Big Letter",  visible: true },
    { id: "name",       label: "Name",        visible: true },
  ],
};
```

---

## 2. `FONTS`

```js
const FONTS = [
  { label: "Poppins",           family: "Poppins",           weight: 700, style: "sans-serif" },
  { label: "Playfair SC",       family: "Playfair Display",  weight: 700, style: "serif" },
  { label: "Bebas Neue",        family: "Bebas Neue",        weight: 400, style: "display" },
  { label: "Cinzel",            family: "Cinzel",            weight: 700, style: "roman" },
  { label: "Dancing Script",    family: "Dancing Script",    weight: 700, style: "script" },
  { label: "Space Mono",        family: "Space Mono",        weight: 700, style: "mono" },
  { label: "Cormorant",         family: "Cormorant Upright",  weight: 700, style: "elegant" },
  { label: "Abril Fatface",     family: "Abril Fatface",     weight: 400, style: "display" },
  { label: "Cinzel Decorative", family: "Cinzel Decorative", weight: 700, style: "display" },
  { label: "UnifrakturCook",    family: "UnifrakturCook",    weight: 700, style: "blackletter" },
  { label: "Pirata One",        family: "Pirata One",        weight: 400, style: "display" },
  { label: "Audiowide",         family: "Audiowide",         weight: 400, style: "display" },
  { label: "Black Ops One",     family: "Black Ops One",     weight: 400, style: "display" },
  { label: "Russo One",         family: "Russo One",         weight: 400, style: "display" },
  { label: "Caveat",            family: "Caveat",            weight: 700, style: "handwriting" },
  { label: "Special Elite",     family: "Special Elite",     weight: 400, style: "display" },
  { label: "Pacifico",          family: "Pacifico",          weight: 400, style: "script" },
  { label: "Bungee",            family: "Bungee",            weight: 400, style: "signage" },
  { label: "Monoton",           family: "Monoton",           weight: 400, style: "retro" },
  { label: "Bangers",           family: "Bangers",           weight: 400, style: "comic" },
  { label: "Rye",               family: "Rye",               weight: 400, style: "western" },
  { label: "Silkscreen",        family: "Silkscreen",        weight: 400, style: "pixel" },
  { label: "Righteous",         family: "Righteous",         weight: 400, style: "display" },
  { label: "Merriweather",      family: "Merriweather",      weight: 700, style: "serif" },
];
```

- `label` — what the UI shows under the font preview.
- `family` — what gets used in CSS / canvas `font` strings. **Must match the `@font-face` `font-family` exactly.**
- `weight` — used when constructing the `FontFace` objects in the font-load effect (the big letter also loads a weight-900 variant) and inside `getFontString`.
- `style` — purely descriptive metadata; not used by the renderer.

---

## 3. `SHAPES`

```js
const SHAPES = ["none", "circle", "square", "rounded-square", "diamond", "hexagon", "triangle", "pentagram", "hexagram"];
```

`"none"` renders no background shape, texture, rings, or outline — only the big letter and name text on a transparent background.

Each shape has two parallel implementations:

- Canvas: `buildShapePath(ctx, shape, cx, cy, R)`, `clipShape(...)`, `strokeShape(...)`.
- SVG: `buildShapeSVGPath(shape, cx, cy, R)`.

---

## 4. `TEXTURES`

```js
const TEXTURES = ["none", "crosshatch", "dots", "grid", "lines"];
```

Drawn by `drawTexture(ctx, texture, W, H)` as low-alpha overlays inside the shape clip. `"none"` short-circuits and draws nothing.

Textures are exported to SVG via `buildTexturePattern()` inside `exportSVG`. Each texture maps to a `<pattern>` element with `patternUnits="userSpaceOnUse"`, clipped to the shape via `clip-path="url(#shapeClip)"`.

---

## 5. `SHIPPED_PRESETS` and `presetSlots`

### `SHIPPED_PRESETS`

Six built-in design presets that serve as the "factory defaults" for the first 6 slots. Each is a complete settings snapshot (all keys from `DEFAULT_SETTINGS` plus a `name` field). These are constants and cannot be modified by the user.

```js
const SHIPPED_PRESETS = [
  { name: "Royal Gold",      bigLetter: "RG", bgTop: "#160e3c", ... },
  { name: "Midnight Aurora",  bigLetter: "MA", bgTop: "#080c28", ... },
  { name: "Crimson Ember",    bigLetter: "CE", bgTop: "#500808", ... },
  { name: "Arctic Silver",    bigLetter: "AS", bgTop: "#e6ebf5", ... },
  { name: "Forest Obsidian",  bigLetter: "FO", bgTop: "#0a1e10", ... },
  { name: "Neon Noir",        bigLetter: "NN", bgTop: "#060408", ... },
];
```

Active detection: a preset is visually marked active when `settings.name === p.name && settings.bgTop === p.bgTop`.

### `presetSlots`

A React state object managing up to 10 user-configurable preset slots. The first 6 slots are pre-filled from `SHIPPED_PRESETS` on first load (or reset).

**Shape:**
```js
{
  slotIndex: 0–9,        // currently selected slot
  slots: [
    {
      name: string,      // preset display name
      data: Settings,    // full settings snapshot (null if slot is empty)
      shipped: boolean,  // true if this slot matches a shipped preset (cannot be deleted, only reset)
    },
    ...
  ],
  saveTarget: number|null,  // slot index being targeted for save (triggers modal)
}
```

**Constants:**
- `TOTAL_PRESET_SLOTS = 10`
- `SHIPPED_SLOT_COUNT = 6`
- `PRESET_SLOTS_KEY = "monogram-studio-preset-slots"`

**Persistence:** stored in `localStorage["monogram-studio-preset-slots"]`. Managed by `persistSlots()` and `getOrInitSlots()` top-level helper functions.

**Actions:**
- `handleSaveToSlot(slotIndex)` — saves current settings to the given slot
- `handleClearSlot(slotIndex)` — empties a slot (sets `data: null`)
- `handleResetSlotToShipped(slotIndex)` — restores a slot to its original shipped preset data
- `commitSaveToSlot()` — confirms save after modal confirmation

**Export/Import:** `exportPresets()` exports all non-empty slots as a JSON bundle. `importPresets()` imports a bundle and merges into current slots.

**Migration:** `migrateSettings()` ensures old shared URLs/JSON files without `bgMode`/`kMode`/`nameMode` keys get safe defaults (all `"gradient"`) without losing existing settings.

### User presets persistence

Preset slots are stored under a **separate** localStorage key from design settings, so they survive reset-to-default and JSON import.

| Key | Format | Written by | Initialised by |
|---|---|---|---|
| `monogram-studio-preset-slots` | JSON array of 10 slot objects | `persistSlots()` on any save/clear/reset | `getOrInitSlots()` on mount |

Each slot object:
```js
{
  slot:         0–9,               // index; never changes
  name:         string | null,     // null means empty
  settings:     object | null,     // full settings snapshot; null if empty
  isShipped:    boolean,           // true for slots 0–5 (factory presets)
  originalName: string | null,     // shipped preset name for reset label
  modifiedAt:   number | null      // Date.now() of last user write; null if untouched
}
```

Bootstrap: `getOrInitSlots()` populates slots 0–5 from `SHIPPED_PRESETS` and slots 6–9 as empty. If a valid 10-slot array already exists in localStorage, it is returned as-is (no re-bootstrap).

---

## 6. URL hash format

```
#config=<base64>
```

- `<base64>` = `btoa(unescape(encodeURIComponent(JSON.stringify(settings))))`.
- Decoded by `decodeSettings(str) = JSON.parse(decodeURIComponent(escape(atob(str))))`, wrapped in try/catch.
- The decoded object is merged with `DEFAULT_SETTINGS` via spread: `{ ...DEFAULT_SETTINGS, ...decoded }`.
- Priority: URL hash > localStorage > `DEFAULT_SETTINGS`.

---

## 6b. Migrations

Track settings-key additions so old shared URLs / JSON files continue to load without error.

All migrations run via `migrateSettings(s)` which is called from both `getInitialSettings()` (page load) and `importJSON` (file import). The function mutates the incoming object and returns it.

| ID | Sprint | Keys | Behaviour |
|---|---|---|---|
| M1 | F12 | `bgMode`, `kMode`, `nameMode` | Defaults to `"gradient"` when absent. Old data with no mode key renders as a gradient (identical to pre-F12 behavior). |
| M2 | F14 | `effects` sub-object | Initialises `effects.glow`, `effects.shadow`, `effects.nameGlow`, and `effects.nameShadow` sub-objects with safe defaults when absent. |

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
- Import: parsed, validated (`parsed.bgTop !== undefined` gate), merged with current settings via spread.

---

## 8. localStorage entries

| Key | Value | Written by | Read by |
|---|---|---|---|
| `monogram-studio-settings` | `JSON.stringify(settings)` | `useEffect` with **50ms debounce** via `useDebounce` | `getInitialSettings()` on mount |
| `monogram-studio-ui-state` | `JSON.stringify({ sections: { [key]: boolean } })` | `useUIState` hook (on section toggle) | `useUIState` hook on mount |
| `monogram-studio-preset-slots` | `JSON.stringify({ slotIndex, slots, saveTarget })` | `persistSlots()` (on any preset slot change) | `getOrInitSlots()` on mount |

---

## 9. Quick validation checklist

When you change anything in `Settings`, verify:

- [ ] Key is in `DEFAULT_SETTINGS`.
- [ ] Key has a UI control somewhere in the design / colors / export panel.
- [ ] Key is read by `drawMonogram`.
- [ ] Key is read by `exportSVG`.
- [ ] Range matches the slider min/max in the design panel JSX.
- [ ] `handleRandomize` emits a sane value for the key (or you've decided to skip randomizing it).
- [ ] A round-trip JSON export + import preserves the value.
- [ ] A share-link round-trip preserves the value.
