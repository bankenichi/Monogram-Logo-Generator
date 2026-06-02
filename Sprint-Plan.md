---
render_with_liquid: false
---

# Sprint-Plan.md — 2026-06 (revised)

A planning document for the next major batch of work on Monogram Studio. **Source of truth** for everything being built in this sprint. Once features ship, the canonical info migrates to `Architecture.md`, `Schemas.md`, `Plan.md`, and `Agents.md`.

**Status:** Approved scope. Awaiting implementation by downstream agentic coder (opencode).

> **For the implementer:** every feature is broken into ordered subtasks (F1.1, F1.2, …). Execute in order within a feature. Each subtask names the exact file, the insertion / search anchor, the change to make, and the acceptance criteria. Do not skip subtasks. Do not reorder waves. Update the `[ ]` checkbox in §1 from `[ ]` to `[x]` as you complete each one and commit.

---

## 0. Sprint goal

Add real depth to the design controls — a fully managed 10-slot preset system with import/export and a browser-storage safety warning, a color wheel, gradient/solid toggles, an Effects tab with drop shadow, a much better font picker with more fonts, an overhauled layout (floating action panel, accordion sections, smaller presets, dedicated Randomizer tab), and the small-fix list — without breaking the "single HTML file, no build step" promise.

Heavier items (layer stack, blend modes, emboss/bevel, letter transformations) are **bonus Wave 7 features**: in-scope but implemented last, only after Waves 1–6 pass all acceptance criteria. See §6.

---

## 1. Scope summary

Numbered for reference throughout the doc. **Effort** is rough: S = under an hour, M = a few hours, L = a half-day, XL = a full day.

Tick each `[ ]` as it lands.

| #     | Feature                                                        | Wave | Effort | Status |
|-------|-----------------------------------------------------------------|------|--------|--------|
| F1    | Pin Typr.js to a commit hash                                    | 1    | S      | [x]    |
| F2    | Throttle `localStorage` writes (300ms debounce)                 | 1    | S      | [x]    |
| F3    | Web Share API on mobile                                         | 1    | S      | [x]    |
| F4    | Yellow accent highlight on undo / redo / reset                  | 1    | S      | [x]    |
| F5    | Pinned floating action panel on canvas (PNG / SVG / Share)      | 2    | M      | [x]    |
| F6    | Collapsible accordion options sections (persisted)              | 2    | M      | [x]    |
| F7    | Smaller, collapsible presets list                               | 2    | S      | [x]    |
| F8a   | Randomizer tab — placeholder                                    | 2    | S      | [x]    |
| F10   | 10-slot preset system — replace shipped defaults, save, import/export bundle, storage warning | 3 | L | [x] |
| F11   | Color picker wheel + hex/RGB inputs                             | 3    | L      | [x]    |
| F12   | Gradient ↔ solid toggle per color slot                         | 3    | M      | [x]    |
| F13a  | Custom font dropdown with per-option preview                    | 4    | M      | [x]    |
| F13b  | Expand to 24 curated fonts (re-sourced corrupt files; swapped 5 for more variety) | 4 | S | [x] |
| F14   | Effects tab — move Glow into it                                | 5    | M      | [x]    |
| F15   | Drop shadow effect                                              | 5    | M      | [x]    |
| F8b   | Randomizer tab — full toggle UI (one switch per randomizable category) | 6 | L | [x] |
| F9    | Lock-colors toggle on Randomize (folded into F8b)               | 6    | (in F8b) | [x]  |
| F8c   | Randomizer tab — "Reset Locks" button (clears all category locks) | 6  | S      | [x]    |
| F19   | Letter transformations (scale X/Y, skew X/Y)                   | 7    | M      | [x]    |
| F18   | Emboss / bevel effect                                           | 7    | M      | [x]    |
| F16   | Blend modes on big letter                                       | 7    | M      | [x]    |
| F17   | Layer stack — visibility toggles + reorder                      | 7    | XL     | [x]    |

**Total rough effort:** ~3-4 dev days. Ship one wave at a time, commit between waves. Wave 7 is bonus — do not start it until Waves 1–6 pass all acceptance tests in §7.

---

## 2. Wave sequencing

Each wave is independently mergeable. After every wave, the app should boot, render, export, and pass the existing acceptance smoke test (`Agents.md` §6).

### Wave 1 — Hygiene + small wins
F1, F2, F3, F4. Pure value, zero architecture risk. Ship first to clear the small-stuff backlog before touching layout.

### Wave 2 — Layout restructure
F5, F6, F7, F8a. Establishes the new UI skeleton. F8a is a deliberate **placeholder** for the new Randomizer tab — it just shows the existing Randomize button moved into its own tab. The full toggle UI (F8b) comes in Wave 6.

### Wave 3 — Preset system + color system
F10, F11, F12. F10 is the biggest structural change in this wave — the 10-slot preset system replaces the old split between hardcoded presets and a user-preset key. Complete F10 before F11/F12. Color wheel is the biggest user-facing improvement.

### Wave 4 — Typography polish
F13a (dropdown UI), F13b (add fonts). Quick wave; could ship same day as Wave 3 if time permits.

### Wave 5 — Effects foundation
F14 (Effects tab + move Glow), F15 (drop shadow). Sets up the per-effect pattern. Limited to two effects (Glow + Drop Shadow) so the pattern is concrete but the surface area stays small.

### Wave 6 — Randomizer completion
F8b. Now that Design / Colors / Effects all exist with their respective controls, the Randomizer tab gets per-category toggles letting the user pin specific dimensions while randomizing others. F9 (lock-colors) is just a category toggle in this UI.

### Wave 7 — Bonus features (implement last)
F19, F18, F16, F17 — in that order. Do not begin Wave 7 until all §7 acceptance tests for Waves 1–6 pass. Each feature is independently mergeable. F17 (layer stack) is the highest-risk; skip it if time is short. See §6 for full specs.

---

## 3. Architecture decisions

These are the calls that cross feature boundaries. Implement them as specified.

### 3.1 Accordion section persistence (used by F6, F7, parts of F8b)

UI state lives in a **separate** localStorage key, distinct from design settings:

```
monogram-studio-ui-state = {
  sections: {
    content: true,
    font: true,
    shape: true,
    typography: true,
    texture: true,
    rings: true,
    glow: true,
    presets: false,        // closed by default
    bg: true,
    bigLetterColor: true,
    nameColor: true,
    ringColors: true,
    effects_glow: true,
    effects_shadow: false,  // closed by default until enabled
    randomizer: true,
    exportSavePreset: true,
    exportDownload: true,
    exportShare: true,
    exportJson: true,
    exportPreview: false      // JSON debug view closed by default
  },
  randomizerLocks: {        // populated in Wave 6 (F8b)
    content: false,
    font: false,
    shape: false,
    ...
  }
}
```

Why separate from `monogram-studio-settings`:
- UI state is per-device, not per-design.
- Reset-to-default should NOT collapse sections.
- Importing a JSON design should NOT change sections.
- Loading a shared URL should NOT change sections.

### 3.2 Floating canvas action panel (F5)

Markup: a new `<div className="canvas-action-overlay">` rendered as the **last** child of `.canvas-wrap` (after `<canvas>`).

Visual contract:
- **Position:** absolute, top: 12px, right: 12px, z-index: 2.
- **Layout:** vertical flex column on desktop, horizontal row on mobile (because vertical icons would crowd a 200px-wide canvas).
- **Buttons:** PNG, SVG, Share — labels with small icons.

CSS states:
- **Desktop default:** Each button is semi-transparent `background: rgba(20,15,36,0.4)`, text `rgba(255,255,255,0.7)`, no container border.
- **Desktop hover on the overlay:** Container gets a darker background `rgba(20,15,36,0.85)`, subtle border `1px solid rgba(255,255,255,0.1)`, rounded corners; buttons go fully opaque.
- **Mobile:** Container always `rgba(20,15,36,0.7)`, smaller padding, no hover behavior. Smaller icon-only labels OK (just the glyph + tooltip via `title`).

**Mobile tap-to-hide behavior (per user spec):** Tapping on the canvas (anywhere inside `.canvas-wrap` but outside the overlay itself) toggles the overlay visibility. This lets the user see the unobstructed design with a single tap, and bring the controls back with another tap. Implemented via a `useState` flag (`overlayHidden`) + a `touchend` listener on `.canvas-wrap`. The overlay gets `.hidden` class which sets `opacity: 0; pointer-events: none;` with a 150ms transition. Tap-target detection: ignore taps that originate inside `.canvas-action-overlay` so the user can still tap PNG/SVG/Share without the panel disappearing on them. Desktop is not affected — hover behavior remains the only show/hide trigger.

Removed in this wave:
- The `<div className="canvas-actions">` block currently inside `.canvas-sidebar` (desktop). This is the column of PNG/SVG/Share buttons next to the presets.
- The mobile-promoted version of `.canvas-actions` (it's the same DOM element via `display: contents` — when we remove the desktop instance, the mobile one goes with it).

After this change, `.canvas-sidebar` on desktop contains ONLY the presets list. On mobile, it dissolves entirely via existing `display: contents` rule and `.canvas-actions` no longer exists in DOM.

### 3.3 Color slot mode (F12)

Each color slot gets a `mode` field controlling whether it renders as a gradient or a flat color.

New Settings keys:
```js
bgMode: "gradient",     // or "solid"
kMode: "gradient",
nameMode: "gradient",
```

When `*Mode === "solid"`:
- Renderer uses `*Top` as a flat fill.
- UI hides `*Bot`, `*GradientType`, `*GradientAngle`, `*GradientStop` controls.

When `*Mode === "gradient"` (default):
- Current behavior preserved exactly.

Migration: missing `*Mode` keys default to `"gradient"` so all existing designs (localStorage, shared URLs, JSON files) render identically.

### 3.4 Effect data model (F14, F15)

Effects live in their own top-level Settings sub-object, keyed by effect name. No layer stack in this sprint — effects always target the big letter (Glow stays where it is, Drop Shadow adds the same way).

New Settings sub-object:
```js
effects: {
  glow:   { enabled: true,  color: "#ffdc50", intensity: 1.0 },
  shadow: { enabled: false, dx: 4, dy: 4, blur: 8, color: "#000000", opacity: 0.5 }
}
```

Migration: top-level `glow`, `glowColor`, `glowIntensity` keys are mirrored into `effects.glow` on read. Both stay in the Settings object for backward compat; reads prefer `effects.glow.*`, writes update both. Old shared URLs continue to work.

When the deferred layer stack ships (next sprint), `effects` moves to per-layer.

### 3.5 Custom font dropdown (F13a)

Native `<select>` won't render `<option>` text in a custom font (browsers strip most styling inside `<option>`). Build a custom React dropdown.

Component contract:
```jsx
<FontDropdown
  value={settings.fontFamily}
  onChange={(family) => set("fontFamily", family)}
  fonts={FONTS}
  fontLoaded={fontLoaded}
/>
```

Internal structure:
- **Closed:** A button showing the currently selected font's "Ag" preview (rendered in that font) + the font label.
- **Click trigger:** Opens a popover below.
- **Popover:** Scrollable list, each row = "Ag" preview (in the font's family + weight) + label text.
- **Click row:** Calls `onChange(family)`, closes the popover.
- **Click outside:** Closes the popover.
- **ESC:** Closes the popover.
- **Active item:** Highlighted (use existing `.font-btn.active` styling).
- **`fontLoaded` gating:** If `false`, dropdown still works but previews fall back to system fonts until ready.

Replaces the existing `<div className="font-grid">` grid layout in the Design tab Font section.

### 3.6 Preset slot system (F10)

The old split model — shipped presets hardcoded in the `PRESETS` constant, user presets in a separate localStorage key — is replaced by a single unified 10-slot store. This is a full rewrite of F10; ignore any notes in earlier drafts.

**Terminology:**
- **Slot** — one of 10 numbered positions (0–9). Each slot is either occupied (has a name + settings snapshot) or empty.
- **Shipped slot** (0–5) — pre-populated from the 6 built-in presets on first load. The user can overwrite, clear, or reset any shipped slot. The shipped data is never gone — the code constant is the source of truth for resets.
- **Extra slot** (6–9) — starts empty. User fills it by saving a design to it.

**Rename the `PRESETS` constant to `SHIPPED_PRESETS`.** It stays in the source file verbatim as a read-only constant. It is used only for (a) bootstrapping the slot store on first load and (b) resetting a shipped slot back to its original state. It is no longer rendered directly anywhere — all rendering goes through the slot store.

**localStorage key:** `monogram-studio-preset-slots`

**`monogram-studio-user-presets` is obsolete** — do not create it. If the key exists from a prior build, ignore it.

**Slot object schema:**
```
{
  slot:         integer 0–9      — index; never changes
  name:         string | null    — null means empty slot
  settings:     object | null    — full settings snapshot; null if empty
  isShipped:    boolean          — true for slots 0–5; never changes per slot
  originalName: string | null    — name of the shipped preset this slot was initialized from;
                                   null for extra slots; never changes; used for reset UI label
  modifiedAt:   number | null    — Date.now() timestamp of the last user write;
                                   null if never user-modified (shipped slots start as null)
}
```

**Initialization — `getOrInitSlots()`:** On load, read `monogram-studio-preset-slots` from localStorage and validate that it is an array of exactly 10 entries with slot indices 0–9. If valid, return it. If missing or malformed, bootstrap: create slots 0–5 from `SHIPPED_PRESETS` (each with `isShipped: true`, `modifiedAt: null`, `originalName` set to the preset name) and slots 6–9 as empty (`name: null`, `settings: null`, `isShipped: false`, `originalName: null`, `modifiedAt: null`). Write the bootstrapped array to localStorage and return it.

**Slot operations:**
- **Save to slot** — write current `settings` and a user-provided `name` into a specific slot index; set `modifiedAt` to `Date.now()`. Persist to localStorage. Available for all 10 slots.
- **Clear slot** — set `name: null`, `settings: null`, `modifiedAt: Date.now()` for that slot. Persist. Works on shipped and extra slots.
- **Reset shipped slot** — look up `SHIPPED_PRESETS` by `originalName`; restore `name` and `settings` from the match; set `modifiedAt: null`. Persist. Only valid when `isShipped === true`.
- **Apply slot** — call `applyPreset(slot.settings)`. Identical to the existing applyPreset path.

**Storage warning trigger:** Show the warning whenever `slots.some(s => s.modifiedAt !== null || (s.isShipped === false && s.name !== null))`. In other words: any slot the user has ever written into. The warning text reads: *"⚠ Your custom presets are stored in browser storage and will be lost if you clear your site data or history. Export a preset bundle from the Export / Import tab to keep a backup."*

The warning appears in two places:
1. At the top of the Presets section (sidebar / mobile), inside the expanded section body.
2. As a yellow-tinted banner at the top of the "Manage Presets" section in the Export / Import tab.

**Preset section UI (sidebar / mobile):** Renders all 10 slots in order. Occupied slots show a `PresetThumb` button (apply on click) plus action icons: × to clear, ↺ to reset (↺ only shown when `isShipped && modifiedAt !== null`). Empty slots show a muted placeholder row — not clickable, visually distinct from occupied slots (dashed border, reduced opacity). The save action is **not** in this section.

**Save flow (Export / Import tab → "Manage Presets" section):**
- A "Save current design to a slot" button opens an inline 10-slot picker.
- The picker shows all 10 rows: slot number, current name (or "— empty —"), and a "Default ★" badge on unmodified shipped slots.
- User clicks a row to select it. The selected row gets a yellow-accent highlight border.
- Below the picker, a name input appears pre-filled with the current `settings.name` value.
- A "Save" button (or "Replace [slot name]" if the chosen slot is occupied) and a "Cancel" button.
- On Save: validate name is non-empty (trim whitespace) and not a duplicate of another occupied slot's name. On pass, call save-to-slot. On fail, show inline error. No second confirmation dialog.
- On Cancel or after a successful save: collapse the picker.

**Preset bundle format (`monogram-presets.json`):** This is a distinct format from the design export JSON. The design export contains one settings object. The preset bundle contains all 10 slots.

```
{
  "app": "Monogram Studio",
  "type": "preset-bundle",
  "version": 1,
  "exportedAt": <timestamp>,
  "slotCount": 10,
  "slots": [ ...all 10 slot objects... ]
}
```

**Export all presets:** Serialise the slot store to a `monogram-presets.json` bundle and trigger a download. Always available regardless of customization state.

**Import presets:** Accept a JSON file. Validate: `type === "preset-bundle"`, `version === 1`, `slots` is an array of 10 objects each with a numeric `slot` key 0–9. On pass, replace the slot store in both state and localStorage. Show a success message with the count of occupied slots. On fail, show a clear error describing why the file was rejected. Do not modify state on failure.

**Reset-to-default does NOT clear preset slots.** Preset slots live in their own localStorage key and survive any settings-level reset. Only the per-slot clear (×) or a preset import overwrites them.

### 3.7 Settings migration framework (F12, F14, F16, F17, F18, F19)

This sprint introduces 6 migrations total. Wire `migrateSettings` into `getInitialSettings()` and `importJSON` — call it immediately after parsing any external settings object.

```js
function migrateSettings(s) {
  // M1: color slot modes (F12)
  if (s.bgMode === undefined)   s.bgMode = "gradient";
  if (s.kMode === undefined)    s.kMode = "gradient";
  if (s.nameMode === undefined) s.nameMode = "gradient";

  // M2: effects sub-object — mirrors top-level glow keys for backward compat (F14)
  if (!s.effects) s.effects = {};
  if (!s.effects.glow) {
    s.effects.glow = {
      enabled:   s.glow          ?? true,
      color:     s.glowColor     ?? "#ffdc50",
      intensity: s.glowIntensity ?? 1.0
    };
  }
  if (!s.effects.shadow) {
    s.effects.shadow = { enabled: false, dx: 4, dy: 4, blur: 8, color: "#000000", opacity: 0.5 };
  }

  // M3: letter transform defaults (F19 — Wave 7)
  if (s.letterScaleX === undefined) s.letterScaleX = 1.0;
  if (s.letterScaleY === undefined) s.letterScaleY = 1.0;
  if (s.letterSkewX  === undefined) s.letterSkewX  = 0;
  if (s.letterSkewY  === undefined) s.letterSkewY  = 0;

  // M4: emboss defaults (F18 — Wave 7)
  if (!s.effects.emboss) {
    s.effects.emboss = { enabled: false, depth: 3, angle: 135,
                         highlight: "#ffffff", shadow: "#000000", opacity: 0.6 };
  }

  // M5: blend mode default (F16 — Wave 7)
  if (s.letterBlendMode === undefined) s.letterBlendMode = "source-over";

  // M6: layer stack default (F17 — Wave 7)
  if (!s.layers) {
    s.layers = [
      { id: "bg",     label: "Background", visible: true },
      { id: "rings",  label: "Rings",      visible: true },
      { id: "letter", label: "Letter",     visible: true },
      { id: "name",   label: "Name",       visible: true },
    ];
  }

  return s;
}
```

M3–M6 are no-ops until Wave 7 features are built. They should still be present from the start so that any settings object entering the app gets all keys, eliminating conditional-key bugs later.

---

## 4. Schema additions consolidated

All keys default to values that produce **no visual change** vs. current behaviour.

```js
// Append to DEFAULT_SETTINGS:
{
  // Color slot modes (F12)
  bgMode:   "gradient",
  kMode:    "gradient",
  nameMode: "gradient",

  // Effects sub-object (F14, F15; extended in F18, F16 — Wave 7)
  effects: {
    glow:   { enabled: true,  color: "#ffdc50", intensity: 1.0 },
    shadow: { enabled: false, dx: 4, dy: 4, blur: 8, color: "#000000", opacity: 0.5 },
    emboss: { enabled: false, depth: 3, angle: 135, highlight: "#ffffff", shadow: "#000000", opacity: 0.6 }
  },

  // Letter transform (F19 — Wave 7)
  letterScaleX: 1.0,
  letterScaleY: 1.0,
  letterSkewX:  0,
  letterSkewY:  0,

  // Blend mode (F16 — Wave 7)
  letterBlendMode: "source-over",

  // Layer stack (F17 — Wave 7)
  layers: [
    { id: "bg",     label: "Background", visible: true },
    { id: "rings",  label: "Rings",      visible: true },
    { id: "letter", label: "Letter",     visible: true },
    { id: "name",   label: "Name",       visible: true },
  ]
}
```

New localStorage keys:
```
monogram-studio-ui-state       # accordion open/closed + randomizer locks
monogram-studio-preset-slots   # unified 10-slot preset array (replaces old user-presets key)
```

---

## 5. Per-feature implementation specs

For each feature, **subtasks are ordered** and **must be completed in order**. Each subtask names: target file(s), where to make the change (search anchor or section name), the change itself, and the acceptance criterion.

### F1 — Pin Typr.js to a commit hash

**Goal:** Replace `@master` with a pinned commit SHA so a force-push on the photopea repo can't silently regress SVG export.

#### F1.1 — Look up the latest stable Typr.js commit
- **Files:** none yet.
- **Action:** Visit `https://github.com/photopea/Typr.js/commits/main`. Copy the SHA of the most recent commit. If the repo has a tagged release, prefer the tag SHA.
- **Acceptance:** SHA is recorded for use in F1.2.

#### F1.2 — Replace both `@master` references in `<head>`
- **File:** `index.html`
- **Search anchor:** `cdn.jsdelivr.net/gh/photopea/Typr.js@master`
- **Edit:** Replace `@master` with `@<sha>` in both `<script>` tags (Typr.js and Typr.U.js).
- **Acceptance:** `grep "@master" index.html` returns no Typr.js matches.

#### F1.3 — Reload + verify SVG export
- **Action:** Hard-reload the app, open DevTools console, export an SVG.
- **Acceptance:** No console error during page load. SVG file downloads. Opening the SVG in a new tab shows text rendered as `<path>` elements (use "View Source" in the SVG tab; search for `<path d="M`). No `<text>` elements in the exported SVG.

---

### F2 — Throttle `localStorage` writes

**Goal:** Coalesce autosave writes so rapid input doesn't hit localStorage on every keystroke.

#### F2.1 — Add a debounce ref + setter helper
- **File:** `index.html`
- **Location:** Inside `function App()`, just above the existing `// ── Auto-save to localStorage ──` comment.
- **Insert:**
  ```js
  // Debounced localStorage writer — coalesces rapid edits into a single write
  const saveTimerRef = useRef(null);
  ```
- **Acceptance:** Variable declared; no syntax errors.

#### F2.2 — Replace the unthrottled autosave effect
- **File:** `index.html`
- **Search anchor:** `localStorage.setItem("monogram-studio-settings"`
- **Locate the surrounding `useEffect(() => { try { localStorage.setItem(...) } catch {} }, [settings]);`**
- **Replace the effect body with:**
  ```js
  useEffect(() => {
    if (saveTimerRef.current) clearTimeout(saveTimerRef.current);
    saveTimerRef.current = setTimeout(() => {
      try { localStorage.setItem("monogram-studio-settings", JSON.stringify(settings)); } catch {}
    }, 300);
    return () => { if (saveTimerRef.current) clearTimeout(saveTimerRef.current); };
  }, [settings]);
  ```
- **Acceptance:** App still autosaves; rapid typing in the Name field shows only one localStorage write per ~300ms of inactivity (verifiable in DevTools → Application → LocalStorage).

#### F2.3 — Flush on unload
- **File:** `index.html`
- **Location:** Inside `App()`, after the F2.2 effect.
- **Insert:**
  ```js
  // Flush any pending localStorage write on unload
  useEffect(() => {
    const handler = () => {
      if (saveTimerRef.current) {
        try { localStorage.setItem("monogram-studio-settings", JSON.stringify(settings)); } catch {}
      }
    };
    window.addEventListener("beforeunload", handler);
    return () => window.removeEventListener("beforeunload", handler);
  }, [settings]);
  ```
- **Acceptance:** Type rapidly then immediately close the tab. Reopen — last value persists.

---

### F3 — Web Share API on mobile

**Goal:** On mobile, the Share button opens the native share sheet instead of just copying to clipboard.

#### F3.1 — Update `shareURL` to branch on capability + viewport
- **File:** `index.html`
- **Search anchor:** `const shareURL = () => {`
- **Replace the body** (keep the same signature):
  ```js
  const shareURL = () => {
    const encoded = encodeSettings(settings);
    if (!encoded) return;
    const url = `${window.location.href.split("#")[0]}#config=${encoded}`;

    const isMobile = window.matchMedia("(max-width: 768px)").matches;
    const canNativeShare = typeof navigator !== "undefined" && typeof navigator.share === "function";

    if (isMobile && canNativeShare) {
      navigator.share({
        title: "Monogram Studio design",
        text: `${settings.bigLetter} — ${settings.name}`,
        url
      }).catch(() => {
        // User cancelled or share failed — silent
      });
      return;
    }

    if (navigator.clipboard) {
      navigator.clipboard.writeText(url).then(() => {
        setShareToast(true);
        setTimeout(() => setShareToast(false), 2500);
      });
    } else {
      window.prompt("Copy this link:", url);
    }
  };
  ```
- **Acceptance:** Desktop unchanged. In DevTools mobile mode + a browser that supports `navigator.share` (Chrome iOS/Android emulation), clicking Share triggers the share sheet. Cancelling the share sheet does not show an error.

---

### F4 — Yellow accent highlight on undo / redo / reset

**Goal:** Make these three buttons visually consistent with the Ko-fi gold accent (`#ffc828`) so they read as "primary actions" rather than icon noise.

#### F4.1 — Add a styled `:hover` rule for `.icon-btn`
- **File:** `index.html`
- **Search anchor:** `.icon-btn:hover:not(:disabled)`
- **Replace** the matching rule with:
  ```css
  .icon-btn:hover:not(:disabled) {
    background: rgba(255,200,40,0.12);
    border-color: rgba(255,200,40,0.4);
    color: #ffc828;
  }
  .icon-btn:active:not(:disabled) {
    background: rgba(255,200,40,0.22);
  }
  ```
- **Acceptance:** Hovering undo / redo / reset shows a yellow tint. Disabled buttons (e.g., undo at history start) remain dim and do NOT highlight on hover.

---

### F5 — Pinned floating action panel on canvas

**Goal:** Move PNG / SVG / Share off the sidebar and overlay them on the canvas, semi-transparent by default, darker on hover (desktop) or always slightly opaque (mobile).

#### F5.1 — Add the overlay JSX inside `.canvas-wrap`
- **File:** `index.html`
- **Search anchor:** `<canvas ref={canvasRef} />`
- **Replace the surrounding `<div className="canvas-wrap" role="img">` block with:**
  ```jsx
  <div className="canvas-wrap" role="img">
    <canvas ref={canvasRef} />
    <div className="canvas-action-overlay">
      <button className="overlay-btn" onClick={downloadPNG} aria-label="Download PNG" title="Download PNG">
        <span aria-hidden="true">↓</span> PNG
      </button>
      <button className="overlay-btn" onClick={downloadSVG} aria-label="Download SVG" title="Download SVG">
        <span aria-hidden="true">↓</span> SVG
      </button>
      <button className="overlay-btn" onClick={shareURL} aria-label="Copy share link" title="Share">
        <span aria-hidden="true">⎘</span>
      </button>
    </div>
  </div>
  ```
- **Acceptance:** Buttons appear overlaid on the canvas. No JS errors.

#### F5.2 — Add overlay CSS (desktop)
- **File:** `index.html`
- **Location:** Inside the inline `<style>` block in `App()`. Place near the existing `.canvas-wrap` rule.
- **Insert:**
  ```css
  /* Floating action panel overlay */
  .canvas-wrap { position: relative; }
  .canvas-action-overlay {
    position: absolute;
    top: 12px;
    right: 12px;
    z-index: 2;
    display: flex;
    flex-direction: column;
    gap: 6px;
    padding: 6px;
    border-radius: 10px;
    background: transparent;
    border: 1px solid transparent;
    transition: background 0.15s, border-color 0.15s;
  }
  .canvas-action-overlay:hover {
    background: rgba(20,15,36,0.85);
    border-color: rgba(255,255,255,0.08);
  }
  .overlay-btn {
    background: rgba(20,15,36,0.4);
    color: rgba(255,255,255,0.75);
    border: 1px solid rgba(255,255,255,0.08);
    border-radius: 8px;
    padding: 6px 12px;
    font-family: 'Poppins', sans-serif;
    font-size: 11px;
    font-weight: 600;
    letter-spacing: 0.06em;
    cursor: pointer;
    transition: all 0.15s;
    display: inline-flex;
    align-items: center;
    gap: 6px;
    -webkit-tap-highlight-color: transparent;
  }
  .canvas-action-overlay:hover .overlay-btn {
    background: rgba(255,255,255,0.08);
    color: #fff;
  }
  .overlay-btn:hover {
    background: rgba(255,200,40,0.18) !important;
    border-color: rgba(255,200,40,0.4);
    color: #ffc828 !important;
  }
  ```
- **Acceptance:** Overlay buttons visible. Hovering the overlay area darkens the container. Hovering an individual button shows the yellow accent.

#### F5.3 — Add overlay CSS (mobile)
- **File:** `index.html`
- **Location:** Inside `@media (max-width: 768px)`.
- **Insert:**
  ```css
  .canvas-action-overlay {
    top: 8px;
    right: 8px;
    flex-direction: row;
    gap: 4px;
    padding: 4px;
    background: rgba(20,15,36,0.7);
    border-color: rgba(255,255,255,0.08);
  }
  .canvas-action-overlay:hover {
    background: rgba(20,15,36,0.7);
  }
  .overlay-btn {
    padding: 5px 8px;
    font-size: 10px;
    background: rgba(255,255,255,0.06);
  }
  ```
- **Acceptance:** On mobile, overlay is always slightly opaque and sits horizontally at the top-right of the preview.

#### F5.4 — Remove the legacy `.canvas-actions` from JSX
- **File:** `index.html`
- **Search anchor:** `<div className="canvas-actions">` (the one inside `.canvas-sidebar`)
- **Delete** the entire `<div className="canvas-actions">...</div>` block including the three buttons inside.
- **Acceptance:** `.canvas-sidebar` now contains only the presets list. The desktop sidebar still renders correctly (presets visible). The mobile layout no longer shows a separate action button row.

#### F5.5 — Clean up dead `.canvas-actions` CSS
- **File:** `index.html`
- **Search anchor:** `/* ── Canvas actions (inside .canvas-sidebar on desktop) ── */`
- **Delete** the `.canvas-actions` rule(s) at this anchor.
- **Also delete** the mobile `.canvas-actions` rule inside `@media (max-width: 768px)`.
- **Acceptance:** No `.canvas-actions` selectors remain (`grep "\.canvas-actions" index.html` empty). Sidebar layout still renders correctly on both breakpoints.

#### F5.6 — Mobile tap-to-hide overlay
- **File:** `index.html`
- **Location:** Inside `App()`, near other useState declarations.
- **Insert:**
  ```js
  const [overlayHidden, setOverlayHidden] = useState(false);
  ```
- **Update F5.1 markup** — change `<div className="canvas-wrap" role="img">` block so it adds the conditional class to the overlay AND attaches a `onTouchEnd` to `.canvas-wrap`. Final shape:
  ```jsx
  <div
    className="canvas-wrap"
    role="img"
    onTouchEnd={(e) => {
      // Ignore taps that originate inside the overlay itself —
      // we don't want PNG/SVG/Share buttons to hide the panel.
      if (e.target.closest('.canvas-action-overlay')) return;
      setOverlayHidden(h => !h);
    }}
  >
    <canvas ref={canvasRef} />
    <div className={`canvas-action-overlay ${overlayHidden ? "hidden" : ""}`}>
      {/* …PNG/SVG/Share buttons from F5.1… */}
    </div>
  </div>
  ```
- **Add CSS** inside the `@media (max-width: 768px)` block:
  ```css
  .canvas-action-overlay.hidden { opacity: 0; pointer-events: none; }
  .canvas-action-overlay { transition: opacity 0.15s, background 0.15s, border-color 0.15s; }
  ```
- **Acceptance:** On a touch device (or DevTools touch emulation), tapping the canvas preview hides the overlay. Tapping again brings it back. Tapping PNG / SVG / Share triggers their normal actions without hiding the overlay. Desktop unaffected — `overlayHidden` defaults to `false` and the toggle handler only fires on `touchend`, not `click` / `mouseup`.

---

### F6 — Collapsible accordion options sections

**Goal:** Wrap each existing `.section` in a collapsible header. Open/closed state persists in localStorage per-section.

#### F6.1 — Add `useUIState` hook
- **File:** `index.html`
- **Location:** Just below the existing `useDebounce` hook (near the top of the Babel script).
- **Insert:**
  ```js
  // UI state (collapsed sections, randomizer locks, etc.) — per-device, distinct from design settings
  const UI_STATE_KEY = "monogram-studio-ui-state";
  function loadUIState() {
    try {
      const raw = localStorage.getItem(UI_STATE_KEY);
      if (raw) return JSON.parse(raw);
    } catch {}
    return {};
  }
  function useUIState() {
    const [state, setState] = useState(loadUIState);
    const update = useCallback((path, value) => {
      setState(prev => {
        const next = { ...prev };
        // Shallow nested update; path is a 2-element array like ["sections", "content"]
        const [bucket, key] = path;
        next[bucket] = { ...(next[bucket] || {}), [key]: value };
        try { localStorage.setItem(UI_STATE_KEY, JSON.stringify(next)); } catch {}
        return next;
      });
    }, []);
    return [state, update];
  }
  ```
- **Acceptance:** Hook compiles. `loadUIState()` returns `{}` on first load.

#### F6.2 — Add `<Section>` component
- **File:** `index.html`
- **Location:** Just below the `useUIState` hook from F6.1.
- **Insert:**
  ```jsx
  // Collapsible section header. children are the section body.
  // defaultOpen lets specific sections (e.g., Presets) start collapsed.
  function Section({ title, sectionKey, defaultOpen = true, children }) {
    const [uiState, updateUI] = useUIState();
    const stored = uiState.sections?.[sectionKey];
    const isOpen = stored === undefined ? defaultOpen : stored;
    const toggle = () => updateUI(["sections", sectionKey], !isOpen);
    return (
      <div className={`section section-collapsible ${isOpen ? "open" : "closed"}`}>
        <button
          type="button"
          className="section-header"
          onClick={toggle}
          aria-expanded={isOpen}
          aria-controls={`section-body-${sectionKey}`}
        >
          <span className="section-title">{title}</span>
          <span className="section-chevron" aria-hidden="true">{isOpen ? "▾" : "▸"}</span>
        </button>
        <div id={`section-body-${sectionKey}`} className="section-body" hidden={!isOpen}>
          {children}
        </div>
      </div>
    );
  }
  ```
- **Acceptance:** Component compiles. Not yet rendered anywhere.

#### F6.3 — Add CSS for collapsible sections
- **File:** `index.html`
- **Location:** Inside the inline `<style>`, near the existing `.section` and `.section-title` rules.
- **Insert:**
  ```css
  .section-collapsible { display: flex; flex-direction: column; }
  .section-header {
    display: flex;
    align-items: center;
    justify-content: space-between;
    background: none;
    border: none;
    padding: 0 0 6px 0;
    margin: 0;
    cursor: pointer;
    width: 100%;
    border-bottom: 1px solid rgba(255,255,255,0.06);
    font-family: inherit;
    -webkit-tap-highlight-color: transparent;
  }
  .section-header:hover .section-title { color: rgba(255,255,255,0.45); }
  .section-header:focus-visible { outline: 2px solid #ffc828; outline-offset: 2px; }
  .section-chevron {
    color: rgba(255,255,255,0.35);
    font-size: 11px;
    margin-left: 8px;
    transition: transform 0.15s;
  }
  .section-collapsible.open .section-body { display: flex; flex-direction: column; gap: 10px; padding-top: 10px; }
  .section-collapsible.closed .section-body { display: none; }
  /* Inline the section-title styling so the existing rule still applies when used inside the new header */
  .section-header .section-title { padding-bottom: 0; border-bottom: none; }
  ```
- **Acceptance:** CSS rules in place. No visual change yet until F6.4 wraps existing sections.

#### F6.4 — Wrap every Design tab section with `<Section>`
- **File:** `index.html`
- **Location:** Inside `{tab === "design" && (<>` block.
- **For each existing `<div className="section">` in the Design tab**, replace `<div className="section"><div className="section-title">XXX</div>...</div>` with `<Section title="XXX" sectionKey="<key>">...</Section>` using these keys:
  | Existing title | sectionKey      | defaultOpen |
  |---|---|---|
  | Content     | `content`     | true |
  | Font        | `font`        | true |
  | Shape       | `shape`       | true |
  | Typography  | `typography`  | true |
  | Texture     | `texture`     | true |
  | Rings       | `rings`       | true |
  | Glow        | `glow`        | true |
- **Note:** Glow will be removed entirely in F14 (moved to Effects tab), so the `glow` section will be deleted then. Wrap it now for consistency.
- **Acceptance:** Each section in the Design tab has a clickable header. Clicking toggles collapse. Refreshing the page preserves collapsed states.

#### F6.5 — Wrap every Colors tab section with `<Section>`
- **File:** `index.html`
- **Location:** Inside `{tab === "colors" && (<>` block.
- **Apply the same pattern:**
  | Existing title | sectionKey       |
  |---|---|
  | Background      | `bg`              |
  | Big Letter      | `bigLetterColor`  |
  | Name Text       | `nameColor`       |
  | Rings           | `ringColors`      |
- **Acceptance:** Same as F6.4 but for Colors tab.

#### F6.6 — Wrap Export/Import tab sections with `<Section>`
- **File:** `index.html`
- **Location:** Inside `{tab === "export/import" && (`.
- **Apply the same pattern:**
  | Existing title       | sectionKey         | defaultOpen |
  |---|---|---|
  | Download             | `exportDownload`   | true |
  | Share                | `exportShare`      | true |
  | Save / Load JSON     | `exportJson`       | true |
  | Current Settings     | `exportPreview`    | false |
- **Note:** The "Save as preset" section is added in F10.3b, NOT here. It will live between Share and Save/Load JSON, with sectionKey `exportSavePreset`.
- **Acceptance:** Same as F6.4 but for Export tab. The Current Settings (JSON debug view) section starts collapsed since it's mostly noise for typical users.

#### F6.7 — Final test
- **Acceptance:** All section headers clickable. Open/closed state persists per-section across reload. Reset-to-default does NOT change section open/closed state. Importing a JSON does NOT change section state.

---

### F7 — Smaller, collapsible presets

**Goal:** Presets take less vertical space and the whole list can be collapsed.

#### F7.1 — Wrap the desktop presets list in `<Section defaultOpen={false}>`
- **File:** `index.html`
- **Search anchor:** `<div className="canvas-sidebar">`
- **Inside the sidebar, the presets-label + presets-list block — replace with:**
  ```jsx
  <Section title="Presets" sectionKey="presets" defaultOpen={false}>
    <div className="presets" role="list" aria-label="Design presets">
      {PRESETS.map(p => (
        <PresetThumb
          key={p.name}
          preset={p}
          active={settings.name === p.name && settings.bgTop === p.bgTop}
          onClick={() => applyPreset(p)}
          fontLoaded={fontLoaded}
        />
      ))}
    </div>
  </Section>
  ```
- **Delete** the old separate `.presets-label` div.
- **Acceptance:** Desktop presets section starts collapsed; clicking the header expands it.

#### F7.2 — Wrap the mobile presets list in `<Section defaultOpen={false}>`
- **File:** `index.html`
- **Search anchor:** `<div className="mobile-presets-wrap">`
- **Replace** the `mobile-presets-wrap` block with:
  ```jsx
  {tab !== "export/import" && (
    <Section title="Presets" sectionKey="presets" defaultOpen={false}>
      <div className="presets mobile-presets" role="list" aria-label="Design presets">
        {PRESETS.map(p => (
          <PresetThumb
            key={p.name}
            preset={p}
            active={settings.name === p.name && settings.bgTop === p.bgTop}
            onClick={() => applyPreset(p)}
            fontLoaded={fontLoaded}
          />
        ))}
      </div>
    </Section>
  )}
  ```
- **Note:** The desktop and mobile copies share the same `sectionKey="presets"` so their collapse states are synced (one toggle controls both — acceptable).
- **Acceptance:** Mobile presets section starts collapsed; expands when tapped.

#### F7.3 — Shrink preset button + thumbnail
- **File:** `index.html`
- **Search anchor:** `.preset-btn { display: flex; align-items: center; gap: 9px; padding:`
- **Replace** the rule with:
  ```css
  .preset-btn { display: flex; align-items: center; gap: 7px; padding: 4px 7px 4px 4px; border-radius: 8px; border: 1px solid rgba(255,255,255,0.1); background: rgba(255,255,255,0.04); color: rgba(255,255,255,0.6); font-family: 'Poppins', sans-serif; font-size: 10px; font-weight: 500; cursor: pointer; transition: all 0.15s; letter-spacing: 0.04em; -webkit-tap-highlight-color: transparent; touch-action: manipulation; text-align: left; }
  ```
- **Search anchor:** `.preset-thumb { width: 30px; height: 30px;`
- **Replace** with: `.preset-thumb { width: 22px; height: 22px; border-radius: 4px; display: block; flex-shrink: 0; }`
- **Acceptance:** Presets visibly smaller. Layout stays correct on both desktop and mobile.

---

### F8a — Randomizer tab (placeholder)

**Goal:** Add a Randomizer tab to the tab list. Move the existing Randomize button into it. Per-category toggles come in Wave 6 (F8b).

#### F8a.1 — Add "randomizer" to the tabs array
- **File:** `index.html`
- **Search anchor:** `{["design","colors","export/import"].map(t => (`
- **Replace** with: `{["design","colors","randomizer","export/import"].map(t => (`
- **Acceptance:** A new "RANDOMIZER" tab appears between Colors and Export/Import.

#### F8a.2 — Add the randomizer tab pane
- **File:** `index.html`
- **Location:** Inside the `panel-content` block, AFTER `{tab === "colors" && (<>...</>)}` and BEFORE `{tab === "export/import" && (`.
- **Insert:**
  ```jsx
  {tab === "randomizer" && (
    <div className="randomizer-tab">
      <p className="export-desc">
        Generate a new design. Per-category lock toggles coming soon — for now this randomizes everything.
      </p>
      <button
        className="randomize-global-btn"
        onClick={handleRandomize}
        style={{
          width: '100%', padding: '14px', marginTop: '12px',
          background: 'rgba(255,200,40,0.06)',
          border: '1px solid rgba(255,200,40,0.2)',
          borderRadius: '8px', color: '#ffc828',
          fontFamily: "'Poppins', sans-serif", fontSize: '14px', fontWeight: '600',
          cursor: 'pointer', display: 'flex', alignItems: 'center',
          justifyContent: 'center', gap: '8px', transition: 'all 0.2s ease'
        }}
        onMouseOver={(e) => e.currentTarget.style.background = 'rgba(255,200,40,0.12)'}
        onMouseOut={(e) => e.currentTarget.style.background = 'rgba(255,200,40,0.06)'}
      >
        🎲 Randomize Design
      </button>
    </div>
  )}
  ```
- **Acceptance:** Selecting the Randomizer tab shows a placeholder note and the Randomize button. Clicking it works exactly like the old global Randomize button.

#### F8a.3 — Remove the global Randomize button at the bottom of panel-content
- **File:** `index.html`
- **Search anchor:** `🎲 Randomize Design`
- **Delete** the entire `<button className="randomize-global-btn" ...>🎲 Randomize Design</button>` block at the bottom of `panel-content` (the one outside any tab condition).
- **Acceptance:** The Randomize button only appears inside the Randomizer tab. No other location.

---

### F10 — 10-slot preset system

**Goal:** Replace the split model (hardcoded `PRESETS` + separate user-presets key) with a unified 10-slot store. Slots 0–5 start with the 6 shipped defaults and are user-replaceable. Slots 6–9 start empty. All saves, resets, and bundle import/export flow through this single store. See §3.6 for the full architecture.

**Wave 3 execution order within F10:** F10.1 → F10.2 → F10.3 → F10.4 → F10.5 → F10.6 → F10.7. Do not jump ahead — each subtask depends on the previous.

---

#### F10.1 — Rename `PRESETS` to `SHIPPED_PRESETS` and add storage constants

- **File:** `index.html`
- **Search anchor:** `const PRESETS = [`
- **Change:** Rename the constant to `SHIPPED_PRESETS`. Verify this is the only declaration (it is). Add three constants immediately after the closing `];` of `SHIPPED_PRESETS`:
  - `PRESET_SLOTS_KEY` — the string `"monogram-studio-preset-slots"`
  - `TOTAL_PRESET_SLOTS` — the integer `10`
  - `SHIPPED_SLOT_COUNT` — the integer `6`
- **Also update** the `ui-state` sections map in §3.1 to add `managePresets: true` (the new section that will be added in F10.5). This keeps the UI-state schema current.
- **Acceptance:** `grep "const PRESETS " index.html` returns nothing. `grep "SHIPPED_PRESETS" index.html` returns at least 2 hits (declaration + usages in F7 sidebar renders, which will be updated in F10.4). No runtime errors.

---

#### F10.2 — Add `getOrInitSlots` and `persistSlots` helpers

- **File:** `index.html`
- **Location:** Just below the three new constants from F10.1, still at the top-level script scope (not inside `App()`).
- **Insert two functions:**
  - `persistSlots(slots)` — serialises the array to JSON and writes it to `PRESET_SLOTS_KEY` in localStorage, wrapped in try/catch.
  - `getOrInitSlots()` — reads and validates `PRESET_SLOTS_KEY` from localStorage. Valid means: array, length exactly 10, `slots[0].slot === 0`. If valid, returns the parsed array. If missing or invalid, builds the bootstrap array (slots 0–5 from `SHIPPED_PRESETS` with `isShipped: true, modifiedAt: null, originalName: <preset.name>`; slots 6–9 empty with `isShipped: false, name: null, settings: null, originalName: null, modifiedAt: null`), calls `persistSlots`, and returns it.
- **Acceptance:** Calling `getOrInitSlots()` in the browser console on a fresh session returns a 10-element array. Calling it again returns the same array (reads from localStorage, does not re-bootstrap). The bootstrapped slots 0–5 match `SHIPPED_PRESETS` names and settings.

---

#### F10.3 — Add slot state and operation handlers in `App()`

- **File:** `index.html`
- **Location:** Inside `App()`, near the other `useState` declarations.
- **Insert:**
  - A `presetSlots` state initialised by calling `getOrInitSlots` (lazy init, no argument).
  - A `hasCustomized` derived boolean computed from `presetSlots`: true when any slot has `modifiedAt !== null` OR (`isShipped === false && name !== null`). Compute this as a const, not a state — it derives from `presetSlots`.
- **Also insert three `useCallback` handlers** (each updates `presetSlots` state and calls `persistSlots`):
  - `handleSaveToSlot(slotIndex, name)` — writes current `settings` snapshot and the given `name` into `presetSlots[slotIndex]`; sets `modifiedAt` to `Date.now()`.
  - `handleClearSlot(slotIndex)` — sets `name: null, settings: null, modifiedAt: Date.now()` on that slot.
  - `handleResetSlotToShipped(slotIndex)` — valid only when `presetSlots[slotIndex].isShipped === true`; looks up the matching entry in `SHIPPED_PRESETS` by `originalName` and restores `name`, `settings`, and `modifiedAt: null`.
- **Also insert save-picker UI state:**
  - `saveSlotMode` boolean (picker open/closed), default false.
  - `selectedSlotIndex` number or null, default null.
  - `saveSlotName` string, default `""`.
  - `saveSlotError` string or null, default null.
- **Also insert `commitSaveToSlot` handler:** validates `selectedSlotIndex !== null`, name non-empty after trim, name not duplicated in any other occupied slot (check both `presetSlots` and `SHIPPED_PRESETS` names to catch cross-slot collisions). On pass, calls `handleSaveToSlot`. On fail, sets `saveSlotError`.
- **Acceptance:** All handlers compile. `hasCustomized` is `false` on a fresh load. Calling `handleSaveToSlot(6, "Test")` in a component test updates slot 6 in both state and localStorage.

---

#### F10.4 — Rewrite the Presets section renders (desktop sidebar + mobile)

The old renders use `PRESETS.map(...)`. Replace both with the new 10-slot render driven by `presetSlots`.

- **File:** `index.html`
- **Desktop anchor:** `<div className="presets-label" aria-hidden="true">Presets</div>` (inside `.canvas-sidebar`, inside the `<Section title="Presets" sectionKey="presets" defaultOpen={false}>` that F7.1 will have added).
- **Mobile anchor:** the inner `<div className="presets mobile-presets" role="list">` (inside the mobile `<Section title="Presets" ...>` from F7.2).
- **For both locations, replace** the `{PRESETS.map(p => (<PresetThumb .../>))}` block with a single render of all 10 `presetSlots`:
  - If `slot.name` is null: render a muted placeholder element — not a button, not interactive. Visually: a row with a dashed border at reduced opacity, labelled "— Empty slot —". Use `aria-hidden="true"` on it.
  - If `slot.name` is non-null: render a flex row containing:
    1. A `<PresetThumb>` that calls `applyPreset(slot.settings)` on click. Pass `preset={{ ...slot.settings, name: slot.name }}`.
    2. A ↺ reset button (only when `slot.isShipped && slot.modifiedAt !== null`) — calls `handleResetSlotToShipped(slot.slot)`. Title: `"Restore to shipped \"${slot.originalName}\""`.
    3. A × clear button — calls `handleClearSlot(slot.slot)`. Title: `"Clear this slot"`.
- **Also insert** the storage warning just inside the expanded section body, before the slot list: render the warning text and a link/note pointing to the Export / Import tab when `hasCustomized` is true. The warning does NOT contain a button itself here — just text directing the user to Export / Import. On mobile the warning and on desktop the warning are identical text; no need to branch.
- **Remove** the old `PRESETS.map(...)` calls and the `presets-label` div they are attached to — the section header from F7 already provides the "Presets" label via `<Section title="Presets" ...>`.
- **Acceptance:** Desktop: 10 rows visible when Presets section is expanded. 6 occupied (shipped), 4 empty placeholders. Clicking an occupied row applies the preset. Clicking × clears the slot (row becomes a placeholder). ↺ appears on shipped slots after they have been saved over, and clicking it restores the shipped preset. Storage warning appears only when `hasCustomized` is true. Mobile: same behaviour.

---

#### F10.5 — Add `exportPresets` and `importPresets` functions

- **File:** `index.html`
- **Location:** Near `exportJSON` and `importJSON`.
- **`exportPresets`:** Builds the preset bundle object (see §3.6 bundle format: `app`, `type: "preset-bundle"`, `version: 1`, `exportedAt`, `slotCount: 10`, `slots`). Creates a Blob, generates an object URL, triggers a download of `monogram-presets.json`, revokes the URL.
- **`importPresets`:** Accepts a file input change event. Reads the file as text. Parses JSON. Validates: `type === "preset-bundle"` AND `version === 1` AND `slots` is an array of length 10 AND every entry has a numeric `slot` key. On pass: calls `persistSlots` with the imported slots array, calls `setPresetSlots` to update state, sets `importPresetsSuccess` to true (auto-clears after 3 seconds, shows occupied slot count). On fail: sets `importPresetsError` to a clear message; does NOT modify state.
- **Also add** `importPresetsError` and `importPresetsSuccess` state variables (string|null and boolean respectively).
- **Acceptance:** Export downloads a valid JSON file. Re-importing that file restores all 10 slots. Importing a design-export JSON (which has `type` undefined, not `"preset-bundle"`) shows the error message and leaves slots unchanged.

---

#### F10.6 — Add "Manage Presets" section to Export / Import tab

- **File:** `index.html`
- **Location:** Inside the `{tab === "export/import" && (` block. Insert as the **first** `<Section>` inside the `<div className="export-wrap">`, before the existing Download section. This positioning matters — users see preset management before they see the design download buttons, reinforcing the preset workflow.
- **Section key:** `managePresets`. Default open: `true`.
- **Contents of the section body (in order):**

  1. **Storage warning banner** — shown when `hasCustomized` is true. A yellow-tinted card with ⚠ icon and the warning text from §3.6. The warning must be visually prominent, not just a small note. Include the text: *"Your custom presets are stored in browser storage and will be lost if you clear your site data or history. Export a preset bundle now to keep a permanent backup."*

  2. **"Export all presets" button** — always visible regardless of `hasCustomized`. Primary style. Downloads `monogram-presets.json` via `exportPresets()`. Button label: `"↓ Export all presets"`.

  3. **"Import presets from file" label/input** — secondary style. Hidden file input (`accept=".json,application/json"`), wrapped in a `<label>`. Calls `importPresets` via `onChange`. Button label: `"↑ Import presets from file"`. Error and success messages render below this button when set.

  4. **A visual divider** (`.export-divider`).

  5. **"Save current design to a slot" sub-section.** A short description paragraph: `"Choose a preset slot to save the current canvas state. Slots 1–6 are the shipped defaults — they can be freely replaced. Slots 7–10 are yours to fill."` Then:
     - When `saveSlotMode` is false: a "Save current design as preset" button (primary style) that sets `saveSlotMode = true`, pre-fills `saveSlotName` with `settings.name`, resets `selectedSlotIndex` and `saveSlotError`.
     - When `saveSlotMode` is true: the inline slot picker (see below) and a Cancel button. Cancel sets `saveSlotMode = false` and clears error.

- **Slot picker:** A grid of 10 rows (2 columns or 1 column, whichever fits the panel width cleanly). Each row is a button with: the slot number (1-indexed in the label, 0-indexed internally), the slot's current name or "— empty —", and a "Default ★" badge when `isShipped && modifiedAt === null`. Clicking a row sets `selectedSlotIndex`. The selected row gets a yellow-accent border. When a slot is selected: a text input appears below the grid (pre-filled with `saveSlotName`, editable), a Save button (label: `"Save"` when target is empty, `"Replace \"${presetSlots[selectedSlotIndex].name}\""` when occupied), and a Cancel button. The Enter key on the input triggers Save. The Escape key triggers Cancel. `saveSlotError` renders below the Save button as an error message when set.

- **Acceptance:** "Manage Presets" section is the first thing visible in Export / Import tab. Warning banner visible with yellow tint when `hasCustomized` is true. Export downloads correctly. Import restores slots with a success message. Save flow: button → picker → select slot → enter name → save → picker closes → Presets section reflects the new slot. Error states: empty name rejected, duplicate name rejected. Occupied-slot replacement label shows the right name.

---

#### F10.7 — CSS for new preset slot UI

- **File:** `index.html`
- **Location:** Inside the inline `<style>`, near existing `.preset-btn` and `.export-wrap` rules.
- **Add rules for:**
  - `.preset-slot-row` — flex row wrapping `PresetThumb` + action icons. `PresetThumb` gets `flex: 1; min-width: 0`.
  - `.preset-slot-empty` — placeholder for empty slots. Dashed border (`rgba(255,255,255,0.08)`), `opacity: 0.4`, `border-radius: 8px`, font size 10px, not focusable.
  - `.slot-action-btn` — base for ↺ and × buttons. No background, no border, small padding, `border-radius: 4px`, `transition: all 0.15s`. Flex-shrink: 0.
  - `.slot-reset-btn` — inherits from `.slot-action-btn`. Default color `rgba(255,200,40,0.45)`. Hover: `color: #ffc828`, subtle yellow background.
  - `.slot-delete-btn` — inherits from `.slot-action-btn`. Default color `rgba(255,255,255,0.25)`. Hover: `color: #ff7070`, subtle red background.
  - `.preset-storage-warn` — flex row, yellow-tinted background (`rgba(255,200,40,0.06)`), yellow border (`rgba(255,200,40,0.15)`), border-radius 7px, font-size 11px, line-height 1.55. The version in the Presets sidebar is compact. The version in Manage Presets has slightly more padding.
  - `.slot-picker-grid` — 2-column grid with `gap: 5px`, `margin-bottom: 10px`.
  - `.slot-picker-item` — button, flex row, `padding: 7px 9px`, `border-radius: 7px`, `border: 1px solid rgba(255,255,255,0.08)`, `background: rgba(255,255,255,0.03)`. Hover: slightly brighter background. Selected state (`.slot-picker-item.selected`): yellow accent border and background tint. Default-star badge (`.slot-picker-item.is-default`): slot name in muted yellow.
  - `.slot-picker-name` — truncated text, `font-size: 10px`, `overflow: hidden`, `text-overflow: ellipsis`, `white-space: nowrap`.
  - `.save-slot-row` — flex row with `gap: 6px`. The text input gets `flex: 1; min-width: 0`.
- **Acceptance:** All new elements render correctly at 280px panel width (mobile) and 320px (desktop panel). No overflow. Placeholder rows are clearly visually distinct from occupied rows. Action icons appear on hover (desktop) or always (mobile).

---

#### F10.8 — Update F8b Randomizer to source presets from slot store (not PRESETS)

The Randomizer's `handleRandomize` builds a random settings object. It previously referenced `PRESETS` for palette sampling or as a reset target. Update any reference to `PRESETS` inside `handleRandomize` to use `SHIPPED_PRESETS` instead.

- **File:** `index.html`
- **Search anchor:** `const handleRandomize = () => {`
- **Action:** Inside the function body, replace `PRESETS` with `SHIPPED_PRESETS` wherever it appears (typically for colour palette sampling or random preset selection). If `PRESETS` does not appear inside `handleRandomize`, this subtask is a no-op — verify and mark done.
- **Acceptance:** Randomizer still functions. `grep "PRESETS\b" index.html` returns only `SHIPPED_PRESETS` (no bare `PRESETS`).

---

### F11 — Color picker wheel + hex/RGB inputs

**Goal:** Replace the existing `ColorRow` component with a custom popover that includes an HSV color wheel + hex input + R/G/B numeric inputs. Live updates.

> This is the biggest single subtask in the sprint. Implementer should commit after F11.3 (component built but not yet swapped in) so it can be reviewed before the swap.

#### F11.1 — Add a `<ColorWheel>` component (drawn via canvas)
- **File:** `index.html`
- **Location:** Just below the existing `ColorRow` function declaration.
- **Insert:** a new component that renders:
  - A 200×200 `<canvas>` with the HSV hue ring + saturation/value square.
  - A draggable thumb showing the current selection.
  - Calls `onChange(hex)` on drag.
- **Implementation reference:** Use the HSV color model. Hue is the outer ring (angle = hue 0-360). Inside the ring, a centered square represents saturation (x: 0-100) and value (y: 0-100, top = max). Convert HSV→hex for `onChange`. Convert incoming `value` hex → HSV to position the thumb.
- **Acceptance:** Component renders a wheel. Click + drag updates `onChange` with a hex value. Imperative `value` prop change repositions the thumb.

#### F11.2 — Helper: hex ↔ rgb ↔ hsv conversions
- **File:** `index.html`
- **Location:** Near the existing `hexToRgb` and `hslToHex` helpers.
- **Insert:**
  ```js
  function rgbToHex(r, g, b) {
    const h = n => Math.max(0, Math.min(255, Math.round(n))).toString(16).padStart(2, "0");
    return `#${h(r)}${h(g)}${h(b)}`;
  }
  function hexToHsv(hex) {
    const [r, g, b] = hexToRgb(hex);
    const rn = r/255, gn = g/255, bn = b/255;
    const max = Math.max(rn, gn, bn), min = Math.min(rn, gn, bn);
    const d = max - min;
    let h = 0;
    if (d !== 0) {
      if (max === rn) h = ((gn - bn) / d) % 6;
      else if (max === gn) h = (bn - rn) / d + 2;
      else h = (rn - gn) / d + 4;
      h = (h * 60 + 360) % 360;
    }
    const s = max === 0 ? 0 : (d / max) * 100;
    const v = max * 100;
    return { h, s, v };
  }
  function hsvToHex(h, s, v) {
    s /= 100; v /= 100;
    const c = v * s;
    const hp = h / 60;
    const x = c * (1 - Math.abs(hp % 2 - 1));
    let r = 0, g = 0, b = 0;
    if (0 <= hp && hp < 1) [r,g,b] = [c, x, 0];
    else if (1 <= hp && hp < 2) [r,g,b] = [x, c, 0];
    else if (2 <= hp && hp < 3) [r,g,b] = [0, c, x];
    else if (3 <= hp && hp < 4) [r,g,b] = [0, x, c];
    else if (4 <= hp && hp < 5) [r,g,b] = [x, 0, c];
    else if (5 <= hp && hp < 6) [r,g,b] = [c, 0, x];
    const m = v - c;
    return rgbToHex((r + m) * 255, (g + m) * 255, (b + m) * 255);
  }
  ```
- **Acceptance:** Roundtrip `hexToHsv("#ffc828") → hsvToHex(...)` returns `#ffc828` (within 1 hex digit rounding).

#### F11.3 — Replace `ColorRow` with `ColorPickerRow`
- **File:** `index.html`
- **Search anchor:** `function ColorRow({ label, value, onChange }) {`
- **Replace** the entire `ColorRow` function with `ColorPickerRow`:
  ```jsx
  function ColorPickerRow({ label, value, onChange }) {
    const [open, setOpen] = useState(false);
    const ref = useRef(null);

    useEffect(() => {
      if (!open) return;
      const close = (e) => {
        if (ref.current && !ref.current.contains(e.target)) setOpen(false);
      };
      const esc = (e) => { if (e.key === "Escape") setOpen(false); };
      window.addEventListener("mousedown", close);
      window.addEventListener("keydown", esc);
      return () => {
        window.removeEventListener("mousedown", close);
        window.removeEventListener("keydown", esc);
      };
    }, [open]);

    const [r, g, b] = hexToRgb(value);
    return (
      <div className="control-row color-picker-row" ref={ref}>
        <label>{label}</label>
        <div className="color-picker-wrap">
          <button
            type="button"
            className="color-swatch"
            style={{ background: value }}
            onClick={() => setOpen(o => !o)}
            aria-label={`${label}: ${value}`}
            aria-expanded={open}
          />
          <span className="hex-label">{value.toUpperCase()}</span>
          {open && (
            <div className="color-picker-popover" role="dialog" aria-label={`Pick ${label}`}>
              <ColorWheel value={value} onChange={onChange} />
              <div className="color-picker-inputs">
                <label className="color-input-label">HEX
                  <input
                    type="text"
                    value={value}
                    onChange={e => {
                      const v = e.target.value.trim();
                      if (/^#[0-9a-fA-F]{6}$/.test(v)) onChange(v);
                    }}
                    maxLength={7}
                  />
                </label>
                <label className="color-input-label">R
                  <input type="number" min="0" max="255" value={r}
                    onChange={e => onChange(rgbToHex(Number(e.target.value), g, b))}/>
                </label>
                <label className="color-input-label">G
                  <input type="number" min="0" max="255" value={g}
                    onChange={e => onChange(rgbToHex(r, Number(e.target.value), b))}/>
                </label>
                <label className="color-input-label">B
                  <input type="number" min="0" max="255" value={b}
                    onChange={e => onChange(rgbToHex(r, g, Number(e.target.value)))}/>
                </label>
              </div>
            </div>
          )}
        </div>
      </div>
    );
  }
  ```
- **Acceptance:** Component compiles. Not yet swapped in.

#### F11.4 — Swap every `<ColorRow>` usage for `<ColorPickerRow>`
- **File:** `index.html`
- **Search anchor:** `<ColorRow `
- **Replace** every occurrence with `<ColorPickerRow ` (preserving props).
- **Acceptance:** All color controls in the Colors tab open a wheel popover. Clicking outside closes. ESC closes. Wheel drag updates the canvas live. Hex/RGB inputs sync with the wheel.

#### F11.5 — CSS for swatch + popover
- **File:** `index.html`
- **Location:** Inside the inline `<style>`, replace the existing `input[type=color]` rule with:
  ```css
  .color-picker-row { position: relative; }
  .color-picker-wrap { display: flex; align-items: center; gap: 8px; position: relative; }
  .color-swatch {
    width: 40px; height: 34px; border: 1px solid rgba(255,255,255,0.15);
    border-radius: 6px; cursor: pointer; padding: 0;
    transition: border-color 0.15s, transform 0.1s;
  }
  .color-swatch:hover { border-color: rgba(255,200,40,0.5); transform: scale(1.04); }
  .color-swatch:focus-visible { outline: 2px solid #ffc828; outline-offset: 2px; }
  .color-picker-popover {
    position: absolute; top: calc(100% + 8px); right: 0;
    background: #14112a; border: 1px solid rgba(255,255,255,0.1);
    border-radius: 10px; padding: 14px;
    box-shadow: 0 12px 40px rgba(0,0,0,0.55);
    z-index: 10; display: flex; flex-direction: column; gap: 12px;
  }
  .color-picker-inputs { display: grid; grid-template-columns: 1fr 1fr; gap: 8px; }
  .color-input-label {
    display: flex; flex-direction: column; gap: 3px;
    font-size: 10px; color: rgba(255,255,255,0.4); letter-spacing: 0.1em; text-transform: uppercase;
  }
  .color-input-label input {
    background: rgba(255,255,255,0.06);
    border: 1px solid rgba(255,255,255,0.1);
    border-radius: 5px; padding: 6px 8px;
    color: #e8e0ff; font-family: 'DM Mono', monospace; font-size: 11px;
    outline: none;
  }
  .color-input-label input:focus { border-color: rgba(255,200,40,0.5); }
  ```
- **Acceptance:** Popover styled, doesn't bleed past viewport edges on desktop. Mobile popover repositions via F11.6.

#### F11.6 — Mobile popover positioning
- **File:** `index.html`
- **Location:** Inside `@media (max-width: 768px)`.
- **Insert:**
  ```css
  .color-picker-popover {
    position: fixed; top: auto; right: 12px; bottom: 12px; left: 12px;
    max-width: none;
  }
  ```
- **Acceptance:** On mobile, picker popover sits at the bottom of the screen and spans full width (minus 12px gutter).

---

### F12 — Gradient ↔ solid toggle per color slot

**Goal:** Each of the three color slots (background, big letter, name) gets a Solid / Gradient segmented control. When Solid, the second color + angle + stop are hidden and the renderer uses the top color as a flat fill.

#### F12.1 — Add new keys to `DEFAULT_SETTINGS`
- **File:** `index.html`
- **Search anchor:** `texture: "none",`
- **Insert before** that line:
  ```js
  bgMode: "gradient",
  kMode: "gradient",
  nameMode: "gradient",
  ```
- **Acceptance:** Three new keys present in `DEFAULT_SETTINGS`.

#### F12.2 — Migration function
- **File:** `index.html`
- **Location:** Just above `function App()`.
- **Insert:**
  ```js
  // Settings migration — runs on every external load to fill in newly added keys.
  function migrateSettings(s) {
    if (!s) return s;
    if (s.bgMode === undefined)   s.bgMode = "gradient";
    if (s.kMode === undefined)    s.kMode = "gradient";
    if (s.nameMode === undefined) s.nameMode = "gradient";
    return s;
  }
  ```
- **Note:** This will grow in F14 to handle effects migration.
- **Acceptance:** Function present.

#### F12.3 — Wire migration into load paths
- **File:** `index.html`
- **Search anchor:** `const getInitialSettings = () => {`
- **Find** every place inside `getInitialSettings` that returns a settings object. **Wrap** the return value with `migrateSettings(...)`:
  ```js
  if (decoded && decoded.bgTop) return migrateSettings({ ...DEFAULT_SETTINGS, ...decoded });
  // ...
  if (parsed.bgTop) return migrateSettings({ ...DEFAULT_SETTINGS, ...parsed });
  // ...
  return migrateSettings({ ...DEFAULT_SETTINGS });
  ```
- **Also wrap** the import-JSON parse path: in `importJSON`, after `JSON.parse`, call `migrateSettings(parsed)` before the bgTop check.
- **Acceptance:** Old shared URLs and JSON files load with new mode keys defaulted to "gradient" — no visual change.

#### F12.4 — Update `drawMonogram` to branch on mode
- **File:** `index.html`
- **Search anchor:** inside `drawMonogram`, the three places where `createDynamicGradient` is called for background, big letter, name.
- **For each**, wrap with a mode branch:
  ```js
  // Background
  if (bgMode === "solid") {
    ctx.fillStyle = bgTop;
  } else {
    ctx.fillStyle = createDynamicGradient(ctx, bgGradientType, bgGradientAngle, bgGradientStop, bgTop, bgBot, cx, cy, H*0.7, H);
  }
  // ... existing fill call ...
  ```
- **Apply the same pattern** to big letter (uses `kMode`, `kTop`/`kBot`) and name (uses `nameMode`, `nameTop`/`nameBot`).
- **Don't forget** to destructure the new keys at the top of `drawMonogram`.
- **Acceptance:** Switching mode in UI (added in F12.6) instantly updates the canvas. Default behavior unchanged when mode is "gradient".

#### F12.5 — Update `exportSVG` to branch on mode
- **File:** `index.html`
- **Search anchor:** inside `exportSVG`, the three `buildGradientDef(...)` calls.
- **For each slot**, branch:
  ```js
  const bgFill = bgMode === "solid" ? bgTop : `url(#bgGrad)`;
  const kFill  = kMode === "solid"  ? kTop  : `url(#kGrad)`;
  const nFill  = nameMode === "solid" ? nameTop : `url(#nameGrad)`;
  ```
- **Use these in the SVG template** instead of the hardcoded `url(#bgGrad)` etc.
- **Skip the `<linearGradient>` / `<radialGradient>` def entirely** for solid slots (otherwise the SVG `<defs>` is bigger than needed, but it's also harmless to keep — keeping them is fine).
- **Don't forget** to destructure the new mode keys at the top of `exportSVG`.
- **Acceptance:** SVG export of a solid-mode slot uses a flat hex fill, not a gradient.

#### F12.6 — Add segmented control to each color section
- **File:** `index.html`
- **Location:** Inside each color section in the Colors tab (Background, Big Letter, Name Text — NOT the Rings section).
- **At the top of each section's body**, insert a mode toggle. Example for Background:
  ```jsx
  <div className="shape-picker" role="group" aria-label="Background fill mode">
    {["solid", "gradient"].map(m => (
      <button
        key={m}
        type="button"
        className={`shape-btn ${settings.bgMode === m ? "active" : ""}`}
        onClick={() => set("bgMode", m)}
        aria-pressed={settings.bgMode === m}
      >{m}</button>
    ))}
  </div>
  ```
- **Wrap** the existing gradient-only controls (bottom color, type, angle, stop) in:
  ```jsx
  {settings.bgMode !== "solid" && (
    <>
      <ColorPickerRow label="Bottom color" value={settings.bgBot} onChange={v=>set("bgBot",v)} />
      {/* ...existing gradient type / angle / stop controls... */}
    </>
  )}
  ```
- **Apply the same pattern** to the Big Letter section (`kMode`, controls hide bottom color + type + angle + stop when solid) and Name Text section (`nameMode`).
- **Acceptance:** Each color section has a Solid / Gradient toggle. Switching to Solid hides the bottom color, type, angle, stop controls. Switching back to Gradient restores them with the same values (state is preserved because we never deleted the keys).

---

### F13a — Custom font dropdown with per-option preview

**Goal:** Replace the 3-column font grid with a dropdown that shows each font's "Ag" preview inline.

#### F13a.1 — Add `<FontDropdown>` component
- **File:** `index.html`
- **Location:** Just below `function PresetThumb`.
- **Insert:**
  ```jsx
  function FontDropdown({ value, onChange, fonts, fontLoaded }) {
    const [open, setOpen] = useState(false);
    const ref = useRef(null);
    const current = fonts.find(f => f.family === value) || fonts[0];

    useEffect(() => {
      if (!open) return;
      const close = (e) => { if (ref.current && !ref.current.contains(e.target)) setOpen(false); };
      const esc = (e) => { if (e.key === "Escape") setOpen(false); };
      window.addEventListener("mousedown", close);
      window.addEventListener("keydown", esc);
      return () => {
        window.removeEventListener("mousedown", close);
        window.removeEventListener("keydown", esc);
      };
    }, [open]);

    return (
      <div className="font-dropdown" ref={ref}>
        <button
          type="button"
          className="font-dropdown-trigger"
          onClick={() => setOpen(o => !o)}
          aria-haspopup="listbox"
          aria-expanded={open}
          aria-label={`Selected font: ${current.label}`}
        >
          <span className="font-dropdown-preview" style={{
            fontFamily: `'${current.family}', sans-serif`, fontWeight: current.weight
          }}>Ag</span>
          <span className="font-dropdown-label">{current.label}</span>
          <span className="font-dropdown-chevron" aria-hidden="true">▾</span>
        </button>
        {open && (
          <ul className="font-dropdown-list" role="listbox">
            {fonts.map(f => (
              <li
                key={f.family}
                role="option"
                aria-selected={f.family === value}
                className={`font-dropdown-option ${f.family === value ? "active" : ""}`}
                onClick={() => { onChange(f.family); setOpen(false); }}
              >
                <span className="font-dropdown-preview" style={{
                  fontFamily: `'${f.family}', sans-serif`, fontWeight: f.weight
                }}>Ag</span>
                <span className="font-dropdown-label">{f.label}</span>
              </li>
            ))}
          </ul>
        )}
      </div>
    );
  }
  ```
- **Acceptance:** Component compiles.

#### F13a.2 — Swap the font grid for the dropdown
- **File:** `index.html`
- **Search anchor:** `<div className="font-grid" role="group" aria-label="Font selection">`
- **Replace** the entire `<div className="font-grid">...</div>` block (including all `FONTS.map(...)` and the closing tag) with:
  ```jsx
  <FontDropdown
    value={settings.fontFamily}
    onChange={(family) => set("fontFamily", family)}
    fonts={FONTS}
    fontLoaded={fontLoaded}
  />
  ```
- **Acceptance:** Font picker is now a single dropdown showing "Ag" + label of the selected font. Opening it shows all fonts each with their own preview.

#### F13a.3 — Add CSS for the dropdown
- **File:** `index.html`
- **Location:** Inside the inline `<style>`, near the existing `.font-grid` rule (which can be deleted).
- **Delete** the `.font-grid`, `.font-btn`, `.font-btn .font-preview`, `.font-btn .font-label`, `.font-btn.active`, `.font-btn:hover:not(.active)`, `.font-btn:focus-visible` rules (no longer used).
- **Insert:**
  ```css
  .font-dropdown { position: relative; width: 100%; }
  .font-dropdown-trigger {
    width: 100%; display: flex; align-items: center; gap: 12px;
    padding: 10px 14px; border-radius: 8px;
    border: 1px solid rgba(255,255,255,0.1);
    background: rgba(255,255,255,0.04);
    color: rgba(255,255,255,0.7);
    font-family: 'Poppins', sans-serif; font-size: 13px;
    cursor: pointer; transition: all 0.15s;
    -webkit-tap-highlight-color: transparent;
  }
  .font-dropdown-trigger:hover { background: rgba(255,255,255,0.08); }
  .font-dropdown-trigger:focus-visible { outline: 2px solid #ffc828; outline-offset: 2px; }
  .font-dropdown-preview { font-size: 22px; line-height: 1; min-width: 30px; }
  .font-dropdown-label { flex: 1; text-align: left; font-size: 12px; }
  .font-dropdown-chevron { font-size: 10px; color: rgba(255,255,255,0.4); }
  .font-dropdown-list {
    position: absolute; top: calc(100% + 6px); left: 0; right: 0;
    margin: 0; padding: 6px; list-style: none;
    background: #14112a; border: 1px solid rgba(255,255,255,0.1);
    border-radius: 10px; box-shadow: 0 12px 40px rgba(0,0,0,0.55);
    z-index: 10; max-height: 320px; overflow-y: auto;
    scrollbar-width: thin; scrollbar-color: rgba(255,255,255,0.15) transparent;
  }
  .font-dropdown-option {
    display: flex; align-items: center; gap: 12px;
    padding: 8px 12px; border-radius: 6px; cursor: pointer;
    color: rgba(255,255,255,0.65); transition: background 0.1s;
  }
  .font-dropdown-option:hover { background: rgba(255,255,255,0.06); color: #fff; }
  .font-dropdown-option.active { background: rgba(255,200,40,0.15); color: #ffc828; }
  ```
- **Acceptance:** Dropdown trigger looks clean. Open list shows scrollable previews. Active option highlighted.

---

### F13b — Add 8 new curated fonts

**Goal:** Expand the font catalogue from 8 to 16 fonts with a range of styles useful for tabletop / branding / streamer use cases.

**Curated additions (all Google-Fonts-origin, free for commercial use):**

| Family               | Weight | File name                                       | Style hint  |
|---|---|---|---|
| Cinzel Decorative    | 700    | `cinzel-decorative-v18-latin-700.woff2`         | ornate roman |
| UnifrakturCook       | 700    | `unifrakturcook-v22-latin-700.woff2`            | blackletter  |
| Pirata One           | 400    | `pirata-one-v22-latin-regular.woff2`            | medieval     |
| Audiowide            | 400    | `audiowide-v20-latin-regular.woff2`             | sci-fi       |
| Black Ops One        | 400    | `black-ops-one-v20-latin-regular.woff2`         | stencil      |
| Russo One            | 400    | `russo-one-v20-latin-regular.woff2`             | bold geo     |
| Caveat               | 700    | `caveat-v18-latin-700.woff2`                    | handwritten  |
| Special Elite        | 400    | `special-elite-v18-latin-regular.woff2`         | typewriter   |

#### F13b.1 — Download woff2 files
- **Tool:** Visit `https://gwfh.mranftl.com/fonts` (Google Webfonts Helper) — select each family, "latin" subset only, weight as per the table, "Modern Browsers" option which produces woff2.
- **Place** each downloaded `.woff2` file in the `fonts/` directory using the exact filename from the table.
- **Acceptance:** All 8 new files present in `fonts/`. Total file count in `fonts/` is 18 (10 existing + 8 new).

#### F13b.2 — Add `@font-face` declarations in `<head>`
- **File:** `index.html`
- **Search anchor:** `@font-face { font-family: 'Abril Fatface';`
- **Append immediately after** that line (still inside the `<style>` block in `<head>`):
  ```css
  @font-face { font-family: 'Cinzel Decorative'; font-style: normal; font-weight: 700; src: url('./fonts/cinzel-decorative-v18-latin-700.woff2') format('woff2'); }
  @font-face { font-family: 'UnifrakturCook';    font-style: normal; font-weight: 700; src: url('./fonts/unifrakturcook-v22-latin-700.woff2') format('woff2'); }
  @font-face { font-family: 'Pirata One';        font-style: normal; font-weight: 400; src: url('./fonts/pirata-one-v22-latin-regular.woff2') format('woff2'); }
  @font-face { font-family: 'Audiowide';         font-style: normal; font-weight: 400; src: url('./fonts/audiowide-v20-latin-regular.woff2') format('woff2'); }
  @font-face { font-family: 'Black Ops One';     font-style: normal; font-weight: 400; src: url('./fonts/black-ops-one-v20-latin-regular.woff2') format('woff2'); }
  @font-face { font-family: 'Russo One';         font-style: normal; font-weight: 400; src: url('./fonts/russo-one-v20-latin-regular.woff2') format('woff2'); }
  @font-face { font-family: 'Caveat';            font-style: normal; font-weight: 700; src: url('./fonts/caveat-v18-latin-700.woff2') format('woff2'); }
  @font-face { font-family: 'Special Elite';     font-style: normal; font-weight: 400; src: url('./fonts/special-elite-v18-latin-regular.woff2') format('woff2'); }
  ```
- **Acceptance:** `@font-face` count matches font file count.

#### F13b.3 — Extend `FONTS` array
- **File:** `index.html`
- **Search anchor:** `{ label: "Abril Fatface",`
- **Append after** the Abril Fatface entry (still inside `const FONTS = [...]`):
  ```js
  { label: "Cinzel Decorative", family: "Cinzel Decorative", weight: 700, style: "ornate" },
  { label: "UnifrakturCook",    family: "UnifrakturCook",    weight: 700, style: "blackletter" },
  { label: "Pirata One",        family: "Pirata One",        weight: 400, style: "medieval" },
  { label: "Audiowide",         family: "Audiowide",         weight: 400, style: "sci-fi" },
  { label: "Black Ops One",     family: "Black Ops One",     weight: 400, style: "stencil" },
  { label: "Russo One",         family: "Russo One",         weight: 400, style: "geometric" },
  { label: "Caveat",            family: "Caveat",            weight: 700, style: "handwritten" },
  { label: "Special Elite",     family: "Special Elite",     weight: 400, style: "typewriter" },
  ```
- **Acceptance:** `FONTS.length === 16`.

#### F13b.4 — Extend `FONT_FILES` map (for SVG export)
- **File:** `index.html`
- **Search anchor:** `const FONT_FILES = {`
- **Add** entries for each new font:
  ```js
  "Cinzel Decorative": "./fonts/cinzel-decorative-v18-latin-700.woff2",
  "UnifrakturCook":    "./fonts/unifrakturcook-v22-latin-700.woff2",
  "Pirata One":        "./fonts/pirata-one-v22-latin-regular.woff2",
  "Audiowide":         "./fonts/audiowide-v20-latin-regular.woff2",
  "Black Ops One":     "./fonts/black-ops-one-v20-latin-regular.woff2",
  "Russo One":         "./fonts/russo-one-v20-latin-regular.woff2",
  "Caveat":            "./fonts/caveat-v18-latin-700.woff2",
  "Special Elite":     "./fonts/special-elite-v18-latin-regular.woff2",
  ```
- **Acceptance:** `FONT_FILES` has 16 entries.

#### F13b.5 — Verify each new font renders in canvas + SVG export
- **Action:** For each new font, select it in the button grid, verify the preview renders in the right typeface, export an SVG, open the SVG in a new tab, verify the `<path>` glyphs match the canvas.
- **Acceptance:** All 24 fonts render correctly in both canvas and SVG.

#### F13b — Completion notes (2026-06-01)
- **Final catalogue: 24 fonts.** After expansion, 5 low-variety fonts (Montserrat, Oswald, Lobster, Raleway, Josefin Sans — all near-duplicates of existing sans/script faces) were swapped out for **Bungee** (signage), **Monoton** (retro neon), **Bangers** (comic), **Rye** (western), **Silkscreen** (pixel).
- **Root-cause fix — corrupt woff2 files.** Several newly added woff2 files were wrong/partial subsets missing the basic-latin alphabet; they loaded without error but rendered only stray glyphs (everything else fell back). Symptom looked like "only the first ~10 fonts work." Re-sourced all affected files as proper latin-subset woff2 from the Fontsource npm packages (`@fontsource/<family>`) since Google's CDN is unreachable from the build sandbox. **Acceptance now includes:** every woff2 must cover `AEXMPLgaeo` (full basic latin) — verify before committing.
- **Loader hardened.** The font-load effect was rewritten to build `FontFace` objects explicitly from `FONT_FILES` and load both the declared weight and weight-900 (the big letter renders at 900), instead of relying on lazy `@font-face` + `document.fonts.load`. See `Architecture.md` §8.
- **Pirata One filename:** file on disk is `pirata-one-v23-...`; the three `index.html` references were aligned to `v23`.
- **Backups retained:** `index.html.bak`, `index.html.inc1`, and the `# Edit conflict #` copy are intentionally kept until further notice (do not delete). The 6 orphaned woff2 files from the removed fonts also remain in `fonts/` (unused; safe to leave).

---

### F14 — Effects tab + move Glow into it

**Goal:** Add an Effects tab. Move the existing Glow controls out of the Design tab into the Effects tab. Migrate the schema to a nested `effects` object.

#### F14.1 — Extend `DEFAULT_SETTINGS` with `effects` sub-object
- **File:** `index.html`
- **Search anchor:** `nameMode: "gradient",` (from F12.1)
- **Insert after** that line:
  ```js
  effects: {
    glow:   { enabled: true,  color: "#ffdc50", intensity: 1.0 },
    shadow: { enabled: false, dx: 4, dy: 4, blur: 8, color: "#000000", opacity: 0.5 }
  },
  ```
- **Acceptance:** Key present.

#### F14.2 — Extend `migrateSettings` to mirror top-level glow into `effects.glow`
- **File:** `index.html`
- **Search anchor:** `function migrateSettings(s) {`
- **Add** before the `return s`:
  ```js
  if (!s.effects) s.effects = {};
  if (!s.effects.glow) {
    s.effects.glow = {
      enabled:   s.glow ?? true,
      color:     s.glowColor ?? "#ffdc50",
      intensity: s.glowIntensity ?? 1.0
    };
  }
  if (!s.effects.shadow) {
    s.effects.shadow = { enabled: false, dx: 4, dy: 4, blur: 8, color: "#000000", opacity: 0.5 };
  }
  ```
- **Acceptance:** Old shared URLs / JSON files load with `effects.glow` populated from the legacy top-level keys.

#### F14.3 — Mirror effect writes back to legacy keys (deprecation window)
- **File:** `index.html`
- **Search anchor:** the `set` helper inside `App()`.
- **Add** a small mirror helper after `set` is defined:
  ```js
  // While the legacy top-level glow keys are still supported for backward compat,
  // mirror effects.glow.* writes back to glow / glowColor / glowIntensity.
  // This keeps shared URLs identical for users on old vs new versions.
  const setEffect = useCallback((effectName, field, value) => {
    setSettings(s => {
      const nextEffects = {
        ...s.effects,
        [effectName]: { ...(s.effects?.[effectName] || {}), [field]: value }
      };
      const next = { ...s, effects: nextEffects };
      // Mirror glow for backcompat
      if (effectName === "glow") {
        if (field === "enabled")   next.glow         = value;
        if (field === "color")     next.glowColor    = value;
        if (field === "intensity") next.glowIntensity = value;
      }
      historyRef.current = historyRef.current.slice(0, historyIndexRef.current + 1);
      historyRef.current.push(next);
      if (historyRef.current.length > 100) historyRef.current.shift();
      historyIndexRef.current = historyRef.current.length - 1;
      return next;
    });
  }, []);
  ```
- **Acceptance:** `setEffect("glow", "enabled", false)` updates both `settings.effects.glow.enabled` and `settings.glow`.

#### F14.4 — Update `drawMonogram` to read from `effects.glow`
- **File:** `index.html`
- **Search anchor:** `glow, glowColor, glowIntensity,` (the destructure inside `drawMonogram`).
- **Replace** with:
  ```js
  // Read from effects sub-object, falling back to legacy keys for safety
  const glowEffect = settings.effects?.glow || {};
  const glow         = glowEffect.enabled   ?? settings.glow;
  const glowColor    = glowEffect.color     ?? settings.glowColor;
  const glowIntensity = glowEffect.intensity ?? settings.glowIntensity;
  ```
- **Remove** `glow, glowColor, glowIntensity,` from the existing destructure.
- **Acceptance:** Toggling glow in the new UI (added next) updates the canvas correctly.

#### F14.5 — Same change in `exportSVG`
- **File:** `index.html`
- **Apply the same source change** for the glow destructure in `exportSVG`. Read from `settings.effects.glow` with legacy fallback.
- **Acceptance:** SVG export glow behavior unchanged.

#### F14.6 — Add "effects" to the tabs array
- **File:** `index.html`
- **Search anchor:** `{["design","colors","randomizer","export/import"].map(t => (`
- **Replace** with: `{["design","colors","effects","randomizer","export/import"].map(t => (`
- **Acceptance:** A new "EFFECTS" tab appears between Colors and Randomizer.

#### F14.7 — Add the Effects tab pane
- **File:** `index.html`
- **Location:** In `panel-content`, AFTER `{tab === "colors" && (<>...</>)}` and BEFORE `{tab === "randomizer" && (`.
- **Insert:**
  ```jsx
  {tab === "effects" && (
    <>
      <Section title="Glow" sectionKey="effects_glow">
        <div className="toggle-row">
          <label>Enable glow</label>
          <button
            className={`toggle ${settings.effects.glow.enabled ? "on" : ""}`}
            onClick={() => setEffect("glow", "enabled", !settings.effects.glow.enabled)}
            aria-pressed={settings.effects.glow.enabled}
            aria-label="Toggle glow"
          />
        </div>
        {settings.effects.glow.enabled && (
          <>
            <SliderRow label="Intensity" value={settings.effects.glow.intensity}
              min={0.2} max={3} step={0.1}
              onChange={v => setEffect("glow", "intensity", v)} />
            <ColorPickerRow label="Glow color" value={settings.effects.glow.color}
              onChange={v => setEffect("glow", "color", v)} />
          </>
        )}
      </Section>

      {/* Drop shadow added in F15. Placeholder section now: */}
      <Section title="Drop Shadow" sectionKey="effects_shadow" defaultOpen={false}>
        <p className="export-desc">Coming up next.</p>
      </Section>
    </>
  )}
  ```
- **Acceptance:** Effects tab renders with Glow section. Glow toggle / intensity / color all work and update the canvas live.

#### F14.8 — Remove Glow from the Design tab
- **File:** `index.html`
- **Search anchor:** `<Section title="Glow" sectionKey="glow">` (the one in the Design tab from F6.4).
- **Delete** the entire Glow `<Section>...</Section>` block from the Design tab.
- **Also remove** the Big Letter color section's `{settings.glow && <div style={{marginTop: 8}}><ColorPickerRow label="Glow color" ...` block in the Colors tab — glow color is now edited in the Effects tab.
- **Acceptance:** Design tab no longer shows Glow. Colors tab no longer shows the inline Glow color picker on the Big Letter section.

---

### F15 — Drop shadow effect

**Goal:** Add a per-design Drop Shadow effect for the big letter. Renderable in both canvas and SVG. Controls: enable, dx, dy, blur, color, opacity.

#### F15.1 — Update `drawMonogram` to apply drop shadow
- **File:** `index.html`
- **Location:** Inside `drawMonogram`, just before the big letter rendering block (the block that uses `letterFont` / `kX` / `kY`).
- **Insert:**
  ```js
  // Drop shadow — drawn as a single tinted shadow pass under the big letter
  const shadowEffect = settings.effects?.shadow || {};
  if (shadowEffect.enabled) {
    const [sr, sg, sb] = hexToRgb(shadowEffect.color || "#000000");
    ctx.save();
    ctx.fillStyle = `rgba(${sr}, ${sg}, ${sb}, ${shadowEffect.opacity ?? 0.5})`;
    ctx.shadowColor = `rgba(${sr}, ${sg}, ${sb}, ${shadowEffect.opacity ?? 0.5})`;
    ctx.shadowBlur = shadowEffect.blur ?? 8;
    ctx.shadowOffsetX = shadowEffect.dx ?? 4;
    ctx.shadowOffsetY = shadowEffect.dy ?? 4;
    ctx.font = letterFont;
    ctx.textAlign = "center";
    ctx.fillText(bigLetter, kX, kY);
    ctx.restore();
  }
  ```
- **Acceptance:** When shadow is enabled, the big letter casts a soft shadow at the configured offset / blur.

#### F15.2 — Update `exportSVG` to emit drop-shadow filter
- **File:** `index.html`
- **Location:** Inside `exportSVG`, near where `glowFilter` is built.
- **Insert:**
  ```js
  const shadowEffect = settings.effects?.shadow || {};
  const shadowFilter = shadowEffect.enabled ? `
    <filter id="shadowFilter" x="-50%" y="-50%" width="200%" height="200%">
      <feGaussianBlur in="SourceAlpha" stdDeviation="${shadowEffect.blur ?? 8}"/>
      <feOffset dx="${shadowEffect.dx ?? 4}" dy="${shadowEffect.dy ?? 4}" result="offsetblur"/>
      <feFlood flood-color="${shadowEffect.color || '#000000'}" flood-opacity="${shadowEffect.opacity ?? 0.5}"/>
      <feComposite in2="offsetblur" operator="in"/>
      <feMerge>
        <feMergeNode/>
        <feMergeNode in="SourceGraphic"/>
      </feMerge>
    </filter>` : "";
  ```
- **Then update the big-letter `<g>` element** to chain both filters when both are enabled:
  ```js
  // Existing: glow ? 'filter="url(#glowFilter)"' : ""
  // Replace with:
  const filterChain = [];
  if (shadowEffect.enabled) filterChain.push("url(#shadowFilter)");
  if (glow)                 filterChain.push("url(#glowFilter)");
  const bigLetterFilterAttr = filterChain.length ? `filter="${filterChain.join(' ')}"` : "";
  ```
- **Replace** the inline `${glow ? 'filter="url(#glowFilter)"' : ""}` for the big letter with `${bigLetterFilterAttr}`.
- **Add** `${shadowFilter}` to the `<defs>` block where `${glowFilter}` is.
- **Acceptance:** SVG export with shadow enabled shows the shadow correctly. Shadow + glow both enabled both render.

#### F15.3 — Add Drop Shadow controls to the Effects tab
- **File:** `index.html`
- **Search anchor:** the placeholder `<Section title="Drop Shadow" sectionKey="effects_shadow" defaultOpen={false}>` from F14.7.
- **Replace** the placeholder content with:
  ```jsx
  <Section title="Drop Shadow" sectionKey="effects_shadow" defaultOpen={false}>
    <div className="toggle-row">
      <label>Enable shadow</label>
      <button
        className={`toggle ${settings.effects.shadow.enabled ? "on" : ""}`}
        onClick={() => setEffect("shadow", "enabled", !settings.effects.shadow.enabled)}
        aria-pressed={settings.effects.shadow.enabled}
        aria-label="Toggle drop shadow"
      />
    </div>
    {settings.effects.shadow.enabled && (
      <>
        <SliderRow label="Offset X" value={settings.effects.shadow.dx}
          min={-50} max={50} step={1}
          onChange={v => setEffect("shadow", "dx", v)} unit="px" />
        <SliderRow label="Offset Y" value={settings.effects.shadow.dy}
          min={-50} max={50} step={1}
          onChange={v => setEffect("shadow", "dy", v)} unit="px" />
        <SliderRow label="Blur" value={settings.effects.shadow.blur}
          min={0} max={40} step={1}
          onChange={v => setEffect("shadow", "blur", v)} unit="px" />
        <SliderRow label="Opacity" value={settings.effects.shadow.opacity}
          min={0} max={1} step={0.05}
          onChange={v => setEffect("shadow", "opacity", v)} />
        <ColorPickerRow label="Shadow color" value={settings.effects.shadow.color}
          onChange={v => setEffect("shadow", "color", v)} />
      </>
    )}
  </Section>
  ```
- **Acceptance:** Drop shadow controls work end-to-end. Toggle on/off; sliders update live; color picker updates live. PNG and SVG exports match the canvas preview.

---

### F8b — Randomizer tab full toggle UI

**Goal:** Replace the placeholder with per-category lock toggles. Each toggle, when on, freezes that category during randomize.

**Categories** (each toggleable):
- `content` — `bigLetter`, `name`
- `font` — `fontFamily`
- `shape` — `shape`
- `texture` — `texture`
- `bgColor` — `bgTop`, `bgBot`, `bgGradientType`, `bgGradientAngle`, `bgGradientStop`, `bgMode`
- `letterColor` — `kTop`, `kBot`, `kGradientType`, `kGradientAngle`, `kGradientStop`, `kMode`
- `nameColor` — `nameTop`, `nameBot`, `nameGradientType`, `nameGradientAngle`, `nameGradientStop`, `nameMode`
- `outline` — `outlineColor`, `outlineWidth`
- `rings` — `showRing1`, `showRing2`, `ring1`, `ring2`, `ring1Width`, `ring2Width`, `ring1Gap`
- `typography` — `letterSize`, `nameSize`, `letterSpacing`, `letterOffsetX`, `letterOffsetY`, `nameOffsetY`
- `effects` — `effects.glow`, `effects.shadow`

#### F8b.1 — Define the category → keys map
- **File:** `index.html`
- **Location:** Top of the Babel script, near other constants.
- **Insert:**
  ```js
  const RANDOMIZER_CATEGORIES = {
    content:     ["bigLetter", "name"],
    font:        ["fontFamily"],
    shape:       ["shape"],
    texture:     ["texture"],
    bgColor:     ["bgTop", "bgBot", "bgGradientType", "bgGradientAngle", "bgGradientStop", "bgMode"],
    letterColor: ["kTop", "kBot", "kGradientType", "kGradientAngle", "kGradientStop", "kMode"],
    nameColor:   ["nameTop", "nameBot", "nameGradientType", "nameGradientAngle", "nameGradientStop", "nameMode"],
    outline:     ["outlineColor", "outlineWidth"],
    rings:       ["showRing1", "showRing2", "ring1", "ring2", "ring1Width", "ring2Width", "ring1Gap"],
    typography:  ["letterSize", "nameSize", "letterSpacing", "letterOffsetX", "letterOffsetY", "nameOffsetY"],
    effects:     ["__effects__"]  // sentinel — handled separately in handleRandomize
  };
  const RANDOMIZER_CATEGORY_LABELS = {
    content: "Content (letter & name)",
    font: "Font",
    shape: "Shape",
    texture: "Texture",
    bgColor: "Background colors",
    letterColor: "Big letter colors",
    nameColor: "Name colors",
    outline: "Outline",
    rings: "Rings",
    typography: "Typography",
    effects: "Effects (glow / shadow)"
  };
  ```
- **Acceptance:** Map present.

#### F8b.2 — Update `handleRandomize` to respect locks
- **File:** `index.html`
- **Search anchor:** `const handleRandomize = () => {`
- **At the top of the function**, read the locks:
  ```js
  const handleRandomize = () => {
    const uiState = loadUIState();
    const locks = uiState.randomizerLocks || {};
    // ... existing logic builds randomizedMatrix ...
  ```
- **At the merge step** (before `setSettings(current => ...)`), strip out any keys belonging to locked categories:
  ```js
  // Strip locked categories
  const finalMatrix = { ...randomizedMatrix };
  for (const [cat, keys] of Object.entries(RANDOMIZER_CATEGORIES)) {
    if (!locks[cat]) continue;
    for (const k of keys) {
      if (k === "__effects__") continue;
      delete finalMatrix[k];
    }
  }
  // Effects: handle separately
  if (!locks.effects) {
    // Roll new effects values
    finalMatrix.effects = {
      glow: {
        enabled: Math.random() > 0.4,
        color: genAccentColor(70),
        intensity: parseFloat((Math.random() * 0.9 + 0.6).toFixed(2))
      },
      shadow: {
        enabled: Math.random() > 0.7,
        dx: Math.floor(Math.random() * 16) - 8,
        dy: Math.floor(Math.random() * 16) - 8,
        blur: Math.floor(Math.random() * 16) + 4,
        color: "#000000",
        opacity: parseFloat((Math.random() * 0.5 + 0.3).toFixed(2))
      }
    };
  }
  // Use finalMatrix instead of randomizedMatrix in the setSettings block below
  setSettings(current => {
    const nextSettings = { ...current, ...finalMatrix };
    // ... rest unchanged ...
  });
  ```
- **Remove** the existing flat-glow keys from the `randomizedMatrix` (the old `glow`, `glowColor`, `glowIntensity` lines) — they're now inside `effects`. Migration mirrors them back via `setEffect`-equivalent logic in the next render cycle. (Actually since this is `setSettings` directly, also write the legacy keys for backcompat: add `glow: finalMatrix.effects.glow.enabled, glowColor: finalMatrix.effects.glow.color, glowIntensity: finalMatrix.effects.glow.intensity` to the merge.)
- **Acceptance:** Randomize works exactly as before (when all locks are off). With a lock on, the locked category's keys are preserved.

#### F8b.3 — Replace the Randomizer tab placeholder with toggle UI
- **File:** `index.html`
- **Search anchor:** `{tab === "randomizer" && (`
- **Replace** the entire tab body with:
  ```jsx
  {tab === "randomizer" && (
    <div className="randomizer-tab">
      <p className="export-desc">
        Lock the categories you want to keep, then hit Randomize. Locked categories stay; everything else gets new values.
      </p>
      <div className="randomizer-locks">
        {Object.keys(RANDOMIZER_CATEGORIES).map(cat => {
          const uiState = loadUIState();
          const isLocked = !!(uiState.randomizerLocks?.[cat]);
          return (
            <div key={cat} className="toggle-row">
              <label>{RANDOMIZER_CATEGORY_LABELS[cat]}</label>
              <button
                type="button"
                className={`toggle ${isLocked ? "on" : ""}`}
                onClick={() => {
                  const next = loadUIState();
                  next.randomizerLocks = { ...(next.randomizerLocks || {}), [cat]: !isLocked };
                  try { localStorage.setItem(UI_STATE_KEY, JSON.stringify(next)); } catch {}
                  // Force a re-render by toggling a dummy state — or use the useUIState hook directly here
                  setRandomizerTick(t => t + 1);
                }}
                aria-pressed={isLocked}
                aria-label={`Lock ${RANDOMIZER_CATEGORY_LABELS[cat]}`}
              />
            </div>
          );
        })}
      </div>

      <button
        className="randomize-global-btn"
        onClick={handleRandomize}
        style={{
          width: '100%', padding: '14px', marginTop: '14px',
          background: 'rgba(255,200,40,0.06)',
          border: '1px solid rgba(255,200,40,0.2)',
          borderRadius: '8px', color: '#ffc828',
          fontFamily: "'Poppins', sans-serif", fontSize: '14px', fontWeight: '600',
          cursor: 'pointer', display: 'flex', alignItems: 'center',
          justifyContent: 'center', gap: '8px'
        }}
      >🎲 Randomize Design</button>
    </div>
  )}
  ```
- **Add** above the return statement in App:
  ```js
  const [randomizerTick, setRandomizerTick] = useState(0);
  ```
- **Acceptance:** Each category has a toggle. Toggling persists across reloads. Randomize respects the locks: locked categories stay identical, unlocked ones change.

#### F8b.4 — Verify F9 (lock colors) is satisfied
- **Action:** Toggle on `bgColor`, `letterColor`, `nameColor`, `outline`, `rings` (all color-touching categories). Randomize. Verify no hex values changed.
- **Acceptance:** F9 closed — lock-colors behavior is implemented as a subset of the per-category toggles.

> **Implementation notes (shipped 2026-06-02):**
> - F8b.2 lands in `handleRandomize` as a "strip locked categories" pass (section `3b`) that clones the rolled matrix into `finalMatrix`, deletes the keys for each locked category, treats the `effects` sentinel specially (drops the rolled `effects` object **and** the legacy `glow` / `glowColor` / `glowIntensity` mirrors), and only re-mirrors the legacy glow keys when effects were actually rerolled. `setSettings` merges `finalMatrix` (not `randomizedMatrix`).
> - The rolled `effects` object also carries `nameGlow` / `nameShadow` sub-objects (richer than the original F8b.2 sketch); these are governed by the same `effects` lock.
> - `content` (`bigLetter`, `name`) is listed as a lockable category but is **not** currently rolled by `handleRandomize`, so locking it is a no-op today. Revisit if/when randomize starts changing letter/name.

#### F8c — "Reset Locks" button (added 2026-06-02, post-F8b request)
- **File:** `index.html`
- **Location:** Randomizer tab, between the `.randomizer-locks` list and the `🎲 Randomize Design` button.
- **Behavior:** Clears all category locks in one click — sets `uiState.randomizerLocks = {}`, persists to `localStorage`, and bumps `setRandomizerTick` so every toggle visibly flips off.
- **Disabled state:** Computed inline via `Object.keys(RANDOMIZER_CATEGORIES).some(cat => loadUIState().randomizerLocks?.[cat])`; the button is `disabled` and greyed when no locks are active.
- **Label:** `🔓 Reset Locks`. Styled as a neutral secondary button (white-tint border) to sit visually below the toggles and above the primary Randomize button.
- **Acceptance:** With one or more locks on, clicking Reset Locks turns them all off and persists across reload; with no locks on, the button is disabled.

---

## 5b. Wave 7 — Bonus features (detailed subtasks)

> **Status (2026-06-02):** Waves 1–6 shipped in a single day (planned for 3–4). Wave 7 — previously parked in §6 "Deferred" — is now **promoted to in-scope** with full subtask breakdowns below. Implement in the order F19 → F18 → F16 → F17 (ascending render-path risk; the layer stack is last because the first three feed its model).
>
> **Status (2026-06-02, updated):** Wave 7 fully implemented. All four features (F19, F18, F16, F17) shipped. Migrations M3–M6 wired.
>
> **Shared groundwork for all four:** every new key must be added to `DEFAULT_SETTINGS` (anchor: `const DEFAULT_SETTINGS = {` ~line 490), defaulted defensively in `migrateSettings` (anchor: `function migrateSettings(s) {` ~line 1658) so old URLs/JSON keep loading, rendered in **both** `drawMonogram` (canvas, ~line 653) **and** the SVG builder inside `downloadSVG` (~line 904), and — unless explicitly excluded — wired into `RANDOMIZER_CATEGORIES` and `handleRandomize`. Add UI to the relevant tab using the existing `<Section>` / `toggle-row` / `<SliderRow>` / `<ColorPickerRow>` components.

---

### F19 — Letter transformations (big letter)

**Goal:** Give the big letter independent geometric control: non-uniform scale (X/Y), skew (X/Y), and rotation, applied around the letter's draw anchor without disturbing rings, background, or name.

**New settings keys** (flat, under root):
```js
letterScaleX: 1.0,   // 0.5 – 2.0
letterScaleY: 1.0,   // 0.5 – 2.0
letterSkewX: 0,      // -45 – 45 (degrees)
letterSkewY: 0,      // -45 – 45 (degrees)
letterRotate: 0      // -180 – 180 (degrees)
```

#### F19.1 — Schema + defaults + migration
- **File:** `index.html`
- Add the five keys to `DEFAULT_SETTINGS`.
- In `migrateSettings`, default each with `s.letterScaleX ??= 1.0` etc. (mutate-and-return pattern already used there).
- **Acceptance:** Old JSON/URL loads with identity transform; no NaN.

#### F19.2 — Canvas render
- **File:** `index.html` — `drawMonogram`, **Big letter** block (anchor: `// Big letter` ~line 695).
- Wrap **all** big-letter passes (glow, drop shadow, fill) in a single `ctx.save()` / transform / `ctx.restore()` so effects move with the glyph. Translate to `(kX, kY)`, apply `rotate(letterRotate°)`, `scale(letterScaleX, letterScaleY)`, `transform(1, tan(skewY), tan(skewX), 1, 0, 0)`, then draw text at the origin (subtract the anchor from each `fillText`, i.e. draw at `(0,0)` after translate).
- **Note:** Because shadows are drawn via `ctx.shadow*`, they already follow the transformed geometry — verify offsets still read as screen-space px (they will; shadow offsets are applied post-transform by the canvas spec, so document the result rather than fighting it).
- **Acceptance:** Scale/skew/rotate visibly transform the letter; rings/name unaffected; glow + shadow track the letter.

#### F19.3 — SVG export
- **File:** `index.html` — SVG builder, big-letter group (anchor: `const bigPath = ` ~line 1268).
- Wrap `bigLetterSVG` in a `<g transform="translate(kX kY) rotate(letterRotate) scale(sx sy) skewX(..) skewY(..) translate(-kX -kY)">…</g>`, or recompute the path around origin. Simplest: emit a wrapping `<g transform="…">` around the existing `bigLetterSVG` string using the **same** anchor `(kX, kY)`.
- **Acceptance:** SVG export matches canvas at several transform combos (SP-18).

#### F19.4 — UI (Design → Typography section)
- **File:** `index.html` — Design tab, `<Section title="Typography" …>` (anchor: `sectionKey="typography"`).
- Add five `<SliderRow>`s: Scale X (0.5–2, step 0.05), Scale Y (same), Skew X (−45–45, 1, "°"), Skew Y (same), Rotate (−180–180, 1, "°"). Wire each to a flat setter (existing `setSettings`-style updater used by other typography sliders).
- **Acceptance:** Sliders update live; reset-to-default returns to identity.

#### F19.5 — Randomizer wiring
- Extend `RANDOMIZER_CATEGORIES.typography` to include the five new keys; have `handleRandomize` roll mild values (e.g. scale 0.85–1.15, skew −12–12, rotate −8–8) so randomize stays tasteful.
- **Acceptance:** Locking `typography` freezes transforms; unlocked randomize varies them subtly.

---

### F18 — Emboss / bevel effect (big letter)

**Goal:** A toggleable emboss effect on the big letter — a light edge and a dark edge offset in opposite directions to fake a raised/engraved look. Lives in the Effects tab alongside glow/shadow.

**New settings** (under `effects`):
```js
effects.emboss: {
  enabled: false,
  depth: 3,            // 1 – 10 px offset
  angle: 135,          // 0 – 360 light direction (degrees)
  highlight: "#ffffff",
  shadow: "#000000",
  opacity: 0.6         // 0 – 1
}
```

#### F18.1 — Schema + defaults + migration
- Add the `emboss` sub-object to `DEFAULT_SETTINGS.effects`.
- In `migrateSettings`, default with `s.effects.emboss ??= { enabled:false, depth:3, angle:135, highlight:"#ffffff", shadow:"#000000", opacity:0.6 }` (guard `s.effects` first).
- **Acceptance:** Old data loads with emboss off.

#### F18.2 — Canvas render
- **File:** `drawMonogram`, big-letter block, **before** the main fill (after drop shadow, before `ctx.fillStyle = kMode …`).
- Compute `dx = depth * cos(angle°)`, `dy = depth * sin(angle°)`. Draw the glyph twice: highlight color at `(kX - dx, kY - dy)` and shadow color at `(kX + dx, kY + dy)`, both at `globalAlpha = opacity`, then the normal fill on top. Respect the F19 transform wrapper (emboss passes go inside it).
- **Acceptance:** Toggling emboss produces a raised look; angle rotates the light source.

#### F18.3 — SVG export
- Add an SVG `<filter id="bigEmboss">` using `feConvolveMatrix` **or** the two-offset-copy approach (two `<use>`/`<path>` copies with the computed offsets and `fill-opacity`). The offset-copy approach is closer to the canvas and avoids filter-region clipping; prefer it.
- Insert the two offset paths into `bigLetterSVG` immediately before the main `bigPath`.
- **Acceptance:** SVG emboss matches canvas (SP-19).

#### F18.4 — UI (Effects tab)
- Add a `<Section title="Emboss" sectionKey="effects_emboss" defaultOpen={false}>` modeled on the Drop Shadow section. Toggle (via `setEffect("emboss","enabled",…)`), then SliderRows for Depth (1–10,1,"px"), Angle (0–360,1,"°"), Opacity (0–1,0.05), and two `<ColorPickerRow>`s for highlight/shadow.
- **Acceptance:** All controls live; gated behind the enable toggle.

#### F18.5 — Randomizer wiring
- `effects` is already a randomizer category. In `handleRandomize`'s rolled `effects` object, add `emboss: { enabled: Math.random() > 0.75, depth: …, angle: …, … }`. Locking `effects` preserves it (already handled by F8b.2's `__effects__` strip).
- **Acceptance:** Randomize occasionally enables emboss; effects lock freezes it.

---

### F16 — Blend mode on big letter

**Goal:** Let the big letter composite against the background/rings with a chosen blend mode (e.g. `multiply`, `screen`, `overlay`) plus an opacity. Cheap, self-contained, and a natural precursor to per-layer blending in F17.

**New settings keys** (flat, under root):
```js
letterBlendMode: "normal",   // one of BLEND_MODES
letterOpacity: 1.0           // 0 – 1
```
Add a constant near the other catalogues:
```js
const BLEND_MODES = ["normal","multiply","screen","overlay","darken","lighten","color-dodge","color-burn","hard-light","soft-light","difference","exclusion","hue","saturation","color","luminosity"];
```

#### F16.1 — Schema + defaults + migration
- Add both keys to `DEFAULT_SETTINGS`; default in `migrateSettings` (`letterBlendMode ??= "normal"`, `letterOpacity ??= 1.0`).
- **Acceptance:** Old data loads as `normal` / fully opaque.

#### F16.2 — Canvas render
- In `drawMonogram`, set `ctx.globalCompositeOperation = letterBlendMode` and `ctx.globalAlpha = letterOpacity` inside the big-letter `save()/restore()` for the **fill pass only** (glow/shadow/emboss should keep `normal`/their own alpha unless we decide otherwise — document the choice). Reset to `"source-over"` / `1` on restore.
- **Caveat:** canvas blend modes composite against everything already drawn (bg + rings). That matches expectation. Note it.
- **Acceptance:** Each mode changes compositing against the background; opacity fades the letter.

#### F16.3 — SVG export
- Map to CSS `mix-blend-mode` via a `style` attr on the big-letter `<g>`: `style="mix-blend-mode:${letterBlendMode}"` plus `opacity="${letterOpacity}"`. SVG blend names match canvas names 1:1 except `normal` (omit when normal).
- **Acceptance:** Browser SVG render matches canvas for the common modes (SP-20). Note: rasterizers other than browsers may ignore `mix-blend-mode` — acceptable, documented.

#### F16.4 — UI (Colors → Big Letter section, or new "Compositing" subsection)
- Add a `<select>` (styled like the font/resolution selects) for blend mode and a `<SliderRow>` for opacity, inside the existing `<Section sectionKey="bigLetterColor">`.
- **Acceptance:** Dropdown + slider update live.

#### F16.5 — Randomizer wiring
- Add `letterBlendMode`, `letterOpacity` to `RANDOMIZER_CATEGORIES.letterColor`. In `handleRandomize`, roll a blend mode from a *curated safe subset* (`["normal","multiply","screen","overlay","soft-light"]`) and opacity 0.85–1.0 so results stay legible.
- **Acceptance:** Locking `letterColor` freezes blend+opacity.

---

### F17 — Layer stack panel (visibility toggles + reorder)

**Goal:** A panel listing the composited layers — Background, Rings, Big Letter, Name — with per-layer visibility toggles and drag-to-reorder of the **draw order** of the foreground layers (Big Letter / Name; Background stays bottom, but its toggle can hide it). XL effort; touches the render path, so it ships last and behind defensive defaults.

> **Architectural decision required before coding (record an ADR via the `engineering:architecture` skill):** whether to (a) keep the four layers as a fixed enum with a reorderable `layerOrder` array + `layerVisible` map (lower risk, recommended), or (b) refactor to a generic layer array (enables F16/F18 per-layer later but is the risky XL path). The subtasks below assume **(a)**.

**New settings keys** (root):
```js
layerOrder: ["background","rings","bigLetter","name"],   // reorderable; background pinned to index 0 in UI
layerVisible: { background: true, rings: true, bigLetter: true, name: true }
```

#### F17.1 — ADR + schema + defaults + migration
- Write the ADR (option a vs b). Add the two keys to `DEFAULT_SETTINGS`; in `migrateSettings`, default both, and **validate** `layerOrder` (drop unknown ids, append any missing so the array is always the full set) to protect the render loop.
- **Acceptance:** Malformed/old data normalizes to the canonical 4-layer order, all visible.

#### F17.2 — Refactor `drawMonogram` into ordered layer draws
- Extract the existing background-block, rings, big-letter, and name code into four local closures (`drawBackground(ctx)`, `drawRings(ctx)`, `drawBigLetter(ctx)`, `drawName(ctx)`) **without changing their internals**. Then iterate `layerOrder`, skipping any whose `layerVisible[id] === false`. Keep background clipping semantics correct (the shape clip currently wraps bg+texture; ensure rings still draw after bg regardless of order, or constrain reordering to bigLetter/name only — decide in the ADR and document).
- **Acceptance:** With default order + all visible, output is **pixel-identical** to pre-refactor (SP-21 regression). Hiding a layer removes exactly that layer.

#### F17.3 — Mirror ordering in SVG export
- The SVG builder assembles a fixed string. Refactor it to build the four layer fragments (`bgSVG`, `ringsSVG`, `bigLetterSVG`, `nameSVG`) — most already exist as locals — then concatenate them in `layerOrder`, skipping hidden ones.
- **Acceptance:** SVG order + visibility match canvas (SP-22).

#### F17.4 — UI: layer panel (new tab or Design subsection)
- Add a `<Section title="Layers">` (or a dedicated tab) listing layers in current `layerOrder`. Each row: a visibility toggle (eye) bound to `layerVisible`, the label, and up/down reorder buttons (drag-reorder is a stretch; **ship up/down arrows first** to de-risk — drag can be a follow-up). Background row's up/down disabled (pinned bottom) per the ADR.
- Persist changes through `setSettings` so they participate in undo/redo and export.
- **Acceptance:** Toggling/reordering updates canvas + SVG live and survives undo/redo + JSON roundtrip.

#### F17.5 — Randomizer + history
- **Exclude** `layerOrder`/`layerVisible` from `handleRandomize` (randomizing layer visibility would produce confusing empty canvases). Add a short code comment noting the deliberate exclusion. Confirm both keys flow through the existing history snapshot logic (they will, since it snapshots whole `settings`).
- **Acceptance:** Randomize never hides/reorders layers; undo/redo includes layer changes.

---

## 6. Deferred to next sprint

Spec'd but **not built this sprint**. These need their own sprint plan when ready.

| #  | Feature | Why deferred |
|----|---|---|
| F20 | Optional: `fonts/manifest.json` for drop-in font workflow | Nice-to-have; doesn't block sprint goals |
| F21 | Multi-line `name` text + leading control | Schema reservation only; build deferred |
| R1 | Glow/shadow parity hardening (canvas vs SVG) | Cosmetic-only; matches at normal settings. Unify shadow-blur convention (#2), shared glow pass table (#3), consistent intensity scaling (#4). Full detail in `Architecture.md` §14 "Remaining gaps / roadmap". |

> **Promoted out of this list (2026-06-02):** F16, F17, F18, F19 — now fully spec'd as **Wave 7** in §5b after Waves 1–6 finished a day in.
>
> **Resolved during Wave 7 (2026-06-02):** glow/shadow/emboss vs blend-mode interaction. The big-letter blend mode now composites the letter against the **background only** (canvas: offscreen background-buffer + glyph mask; SVG: isolated glyph-clipped group with `mix-blend-mode`), and stacking order is unified to shadow → glow → emboss → fill in both renderers. Closed parity audit items #1 and #5; #2/#3/#4 carried forward as roadmap **R1**.

---

## 7. Aggregated testing criteria

Run this end-to-end after each wave. All must pass before declaring the wave done.

| ID | Test | Pass criteria |
|---|---|---|
| SP-1 | Fresh page load (empty localStorage) | Default design renders. Tab list reads Design / Colors / Effects / Randomizer / Export-Import (full sprint complete). Floating action panel visible on canvas. |
| SP-2 | Old shared URL (pre-sprint format) loads | No errors. Design renders identically to pre-sprint. New keys default to gradient mode and effects.glow mirrors top-level glow. |
| SP-3 | Old JSON import | Same as SP-2. |
| SP-4 | New JSON roundtrip (export → import) | Identical design. All new keys preserved. |
| SP-5 | Share link roundtrip | Same as SP-4 via URL hash. |
| SP-6 | Each color slot: switch to Solid → only top color used | Canvas + SVG both flat-color. |
| SP-7 | Each color slot: switch back to Gradient → previous values restored | No data loss. |
| SP-8 | Save 3 user presets (via Export/Import → Save as preset section), reload | All three present in the Presets section, applyable, deletable. |
| SP-8b | Try to save a 5th preset | In Export/Import → Save as preset: button hidden once 4 presets exist; yellow-tinted "limit reached" note shown referencing "Export JSON below". After deleting one in the Presets section, the Save button reappears here. |
| SP-8c | Reset-to-default with 4 user presets saved | Settings reset; all 4 user presets still present in localStorage and in the Presets section. |
| SP-8d | Mobile: tap canvas with overlay visible | Overlay fades out. Tap again → fades back in. Tap PNG/SVG/Share button → action fires, overlay does NOT hide. |
| SP-8e | UI separation | Confirm: Presets section has list + delete only (no Save button). Export/Import has Save as preset section (no list, no delete). |
| SP-9 | Collapse all sections, reload | Stay collapsed. |
| SP-10 | Mobile @ 375px end-to-end | Floating overlay visible at top-right of preview. All tabs render. Color picker popover repositions to bottom of screen. Font dropdown opens. |
| SP-11 | Reset-to-default | Settings reset to defaults; UI state (collapsed sections, randomizer locks) NOT reset; user presets NOT deleted. |
| SP-12 | Undo / redo through 10+ ops including effect changes | All new keys participate in history; no missing-key errors. |
| SP-13 | Toggle glow off via Effects tab, undo | Glow comes back on. |
| SP-14 | Each new font: select → preview correct → SVG export correct | All 24 fonts work (each woff2 covers full basic latin). |
| SP-15 | Drop shadow toggle on, every slider responsive, PNG + SVG match | Visual parity. |
| SP-16 | Randomizer: lock `font`, randomize, font unchanged | Per-category lock works. Other categories rerolled. |
| SP-17 | Randomizer: lock several categories, click "Reset Locks" | All toggles flip off; persists across reload. Button is disabled/greyed when no locks are active. |
| SP-18 | F19: set letter scale/skew/rotate, export PNG + SVG | Both match; rings/name/background unaffected; glow + shadow track the transformed letter. |
| SP-19 | F18: enable emboss, vary angle/depth, export PNG + SVG | Raised/engraved look in both; angle rotates the light source; off by default on old data. |
| SP-20 | F16: cycle big-letter blend modes + opacity, export PNG + SVG | Browser SVG render matches canvas for common modes; `normal`/opaque on old data. |
| SP-21 | F17: default layer order + all visible, after refactor | Output pixel-identical to pre-refactor (regression guard). |
| SP-22 | F17: hide a layer / reorder bigLetter↔name, export PNG + SVG | Canvas + SVG reflect the same order/visibility; survives undo/redo + JSON roundtrip. |
| SP-23 | Wave 7 old-data load (pre-Wave-7 URL + JSON) | Loads clean: identity transform, emboss off, blend normal, canonical 4-layer order all visible. |

---

## 8. Risks & open questions

### Risks

| Risk | Likelihood | Impact | Mitigation |
|---|---|---|---|
| `<ColorPickerRow>` swap breaks something subtle (existing color flow expected `ColorRow`) | Medium | Medium | Swap is purely visual — same `value` / `onChange` contract. Smoke-test every color in Colors tab. |
| Color wheel canvas drawing buggy | Medium | Low | If wheel has issues, fall back to current `<input type="color">` inside the popover until fixed. |
| New fonts hosted via Google Webfonts Helper produce slightly different filenames per version (`v18` vs `v22`) | High | Low | Use exactly the filename the helper gives; update `FONT_FILES` map to match. |
| Mobile color popover overlaps the canvas | Medium | Low | Repositioned via F11.6 to fixed-bottom. |
| Migration introduces a key the old code doesn't know about, breaking old reads | Low | High | Migration is additive only — old code keeps reading legacy keys, which are still maintained via mirror writes (F14.3). |
| `display: contents` on `.canvas-sidebar` (mobile) breaks when `.canvas-actions` is removed | Low | Medium | After F5.4 the sidebar contains only presets. Mobile rule `.canvas-sidebar { display: contents }` becomes redundant but harmless. Remove if convenient. |
| `setRandomizerTick` re-render trick is ugly | Low | Low | Acceptable for placeholder; refactor into proper `useUIState`-driven render after F8b ships. |

### Open questions for the architect / user

All resolved as of 2026-06 review. Resolutions baked into the relevant sections; recorded here for traceability.

| # | Question | Resolution | Affected sections |
|---|---|---|---|
| 1 | Should resetting the design also clear user presets? | **No.** User presets survive any settings-level reset. | §3.6, F10.x, SP-8c |
| 2 | Max user presets count? | **4.** Hard cap. Save UI lives in Export/Import tab (not Presets section). At cap, Save button is replaced with a note pointing the user at Export JSON as a portable alternative. | §3.6, F10.2, F10.3a, F10.3b, SP-8b, SP-8e |
| 3 | Should the floating overlay hide while the user is touching the canvas (mobile)? | **Yes** — tap-to-toggle. Tap canvas → overlay fades out; tap again → fades back in. Taps on overlay buttons themselves still fire normally without hiding. Desktop unaffected. | §3.2, F5.6, SP-8d |
| 4 | Wave 6 (F8b) — randomization of `texture` as a toggleable category? | **Yes**, include `texture` in the per-category locks. Placeholder (F8a) just moves the existing button — full toggle list comes in F8b. | F8b.1, F8b.3 |
| 5 | F13b font choices acceptable? | **Yes** — keep the curated 8 (Cinzel Decorative, UnifrakturCook, Pirata One, Audiowide, Black Ops One, Russo One, Caveat, Special Elite). | F13b table |

---

## 9. Doc-update checklist after implementation

Mark `[x]` as each lands:

- [x] `Schemas.md` §1 — add `bgMode`, `kMode`, `nameMode`, `effects` sub-object to Settings table
- [x] `Schemas.md` §6 (migrations) — record M1 (color modes), M2 (effects mirror)
- [x] `Schemas.md` — new "User presets" subsection covering the localStorage format
- [x] `Architecture.md` §3 — describe the `migrateSettings` framework
- [x] `Architecture.md` §4 — drawMonogram now branches on `*Mode`; effects pipeline (Glow + Drop Shadow)
- [x] `Architecture.md` — new "Floating action overlay" subsection covering F5
- [x] `Architecture.md` — new "Collapsible sections" subsection covering F6 + `useUIState`
- [x] `Architecture.md` — new "Font dropdown" subsection
- [x] `Plan.md` §1 — tick every F# in the Sprint as it ships *(through F8b/F8c/F9; Wave 7 still open)*
- [x] `Plan.md` §2 — randomizer-lock acceptance tests added (G4, G5)
- [x] `Plan.md` §6 — record migrations
- [x] `Agents.md` §3.x — new recipes ("add an effect", "add a Section", "add a font" updated for 16 entries)
- [x] `readme.md` — features list (font count 8 → 16, color wheel, drop shadow, user presets, randomizer locks, etc.)

---

## 10. Definition of done

Sprint is complete when:
- All Wave 1–6 features (`F1`–`F15`, `F8a`, `F8b`, `F8c`, `F9`, `F10`–`F13b`) are `[x]` in §1. **(Done 2026-06-02.)**
- Wave 7 features (`F16`, `F17`, `F18`, `F19`) are `[x]` in §1 per the §5b subtasks. **(Done 2026-06-02.)**
- All `SP-*` acceptance tests in §7 pass (including SP-18 – SP-23 for Wave 7).
- Doc-update checklist in §9 is complete.
- No regressions in `Agents.md` §6 smoke test.
- Open questions in §8 are resolved or explicitly noted as accepted.
