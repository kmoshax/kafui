# Switch

**M3 category:** Selection — a toggle control for a single binary setting. Represents an **immediate on/off action** (unlike a Checkbox which is form state committed on submit). Common for settings screens.

M3 2024 details:
- The thumb **morphs size on press**: grows to 28dp while held, returns to resting size on release.
- An optional **icon** occupies the thumb — a check mark when selected, an optional X/cross when unselected.
- M3 Expressive (2025) adds an optional spring easing for the thumb translate (`emphasized-decelerate` on select, `emphasized-accelerate` on deselect).
- Icons are **separate slots** (`checkedIcon`, `uncheckedIcon`), not a single toggling icon, matching the M3 anatomy.

---

## Anatomy / Parts

```
.switch                          — root <label>; flex row; wraps track + optional label
├── .switch__track               — 52×32dp pill track
│   ├── .switch__track-bg        — background fill + border; color-transitions on/off
│   └── .switch__thumb-container — moves via inset-inline-start; 40dp touch zone
│       ├── .switch__state-layer — 40dp circle; hover/focus/pressed overlay
│       ├── .switch__handle      — thumb circle; size-morphs
│       │   ├── .switch__icon--checked    — shown when selected (check SVG)
│       │   └── .switch__icon--unchecked  — shown when unselected (cross SVG); only with iconVariant="both"
└── .switch__label               — (optional) adjacent text; body-large
```

| BEM element | Notes |
|---|---|
| `.switch` | Root `<label>`; `display: inline-flex; align-items: center` |
| `.switch__track` | 52×32dp; `border-radius: var(--corner-full)`; `position: relative` |
| `.switch__track-bg` | `position: absolute; inset: 0`; transitions fill + border |
| `.switch__thumb-container` | 40×40dp; centers handle; translates via `inset-inline-start` |
| `.switch__state-layer` | 40dp circle; `pointer-events: none` |
| `.switch__handle` | Thumb circle; size-morphs 16dp → 24dp → 28dp |
| `.switch__icon--checked` | Check SVG; `opacity: 0` when unselected |
| `.switch__icon--unchecked` | Cross SVG; `opacity: 0` when selected; only rendered when `iconVariant="both"` |
| `.switch__label` | Text label; `body-large` |

**Handle size transitions (M3 2024):**

| State | Size |
|---|---|
| Unselected resting | 16dp |
| Selected resting | 24dp |
| Pressed (either state) | 28dp |
| Dragging | 28dp |

All size transitions use `--duration-short3` (~150 ms) + `--easing-standard`.

---

## Variants

| Variant | `iconVariant` prop | Description |
|---|---|---|
| **No icon** | `"none"` (default) | Plain thumb |
| **Check only** | `"check"` | Check icon visible when selected; no icon unselected |
| **Check + cross** | `"both"` | Check on selected; X on unselected |

No `variant` prop for visual style — M3 Switch has a single visual form.

---

## States

| State | Track | Handle size | Handle color | Icon |
|---|---|---|---|---|
| **Unselected resting** | `--surface-container-highest` bg; 2dp `--outline` border | 16dp | `--outline` | hidden (or cross if `both`) |
| **Selected resting** | `--primary` fill; no border | 24dp | `--on-primary` | check (if `check`/`both`) |
| **Unselected hover** | + `--state-hover` (8%) tint `--on-surface` | 16dp | `--on-surface-variant` | — |
| **Selected hover** | + `--state-hover` tint `--primary` | 24dp | `--on-primary-container` | — |
| **Unselected focus** | + `--state-focus` (10%) tint `--on-surface` | 16dp | unchanged | — |
| **Selected focus** | + `--state-focus` tint `--primary` | 24dp | unchanged | — |
| **Unselected pressed** | + `--state-pressed` (10%) tint `--on-surface` | **28dp** | `--on-surface-variant` | — |
| **Selected pressed** | + `--state-pressed` tint `--primary` | **28dp** | `--on-primary-container` | — |
| **Disabled unselected** | `--on-surface` @ 12% bg; `--on-surface` @ 12% border | 16dp | `--on-surface` @ 38% | @ 38% opacity |
| **Disabled selected** | `--on-surface` @ 12% bg; no border | 24dp | `--surface` | @ 38% opacity |

The state-layer color is controlled by a `--sl-color` custom property on `.switch__thumb-container`, toggled between `--on-surface` (unselected) and `--primary` (selected).

---

## Design Tokens

All references are unprefixed system roles per `_TOKENS.md`.

| Token | Usage |
|---|---|
| `--primary` | Selected track fill; selected state-layer tint |
| `--on-primary` | Selected handle; check icon color |
| `--on-primary-container` | Selected hover/pressed handle |
| `--surface-container-highest` | Unselected track fill |
| `--outline` | Unselected track border; unselected handle color |
| `--on-surface-variant` | Unselected hover/pressed handle |
| `--on-surface` | Unselected state-layer tint; disabled tint |
| `--surface` | Disabled selected handle |
| `--corner-full` | Track pill radius; handle circle |
| `--body-large-*` | Label text typescale |
| `--duration-short3` | Thumb size + translate transition (~150 ms) |
| `--easing-standard` | Thumb easing (resting transitions) |
| `--easing-emphasized-decelerate` | Thumb translate on select (M3 Expressive) |
| `--easing-emphasized-accelerate` | Thumb translate on deselect (M3 Expressive) |
| `--state-hover` | 0.08 |
| `--state-focus` | 0.10 |
| `--state-pressed` | 0.10 |

---

## Interaction & Accessibility

### ARIA roles
- RAC `Switch` renders a `<label>` wrapping a visually-hidden `<input type="checkbox" role="switch">`.
- `role="switch"` with `aria-checked="true|false"` (not `aria-pressed` — `switch` role uses `aria-checked`).
- `aria-disabled` when disabled.
- `aria-label` or `aria-labelledby` required when no visible label child is provided.
- `aria-readonly` when `isReadOnly`.

### Keyboard
| Key | Action |
|---|---|
| `Tab` | Focus switch |
| `Space` | Toggle on/off |
| `Enter` | Toggle on/off (additional; some platforms expect this) |

### Touch / pointer
- Full 52×32dp track is the hit target (exceeds 48dp minimum in the wider dimension).
- `.switch__thumb-container` provides an additional 40dp centered touch zone on the handle.
- `onPress` (React Aria); not `onClick`.
- **Optional drag:** pointer-down on track, drag left/right to toggle. Progressive enhancement — requires a custom `useMove` hook layered over RAC `Switch`'s press handling. Implemented as a separate `useSwitchDrag` hook; opt-in via `enableDrag` prop.

### Reduced motion
```css
@layer kafui {
  @media (prefers-reduced-motion: reduce) {
    .switch__handle         { transition: width var(--duration-short3), height var(--duration-short3); /* keep size only */ }
    .switch__thumb-container { transition: none; } /* no translate animation */
    .switch__track-bg       { transition: background var(--duration-short3); } /* keep color only */
  }
}
```

Note: color transitions are non-spatial, so they are preserved under reduced motion. Only translate and size morph are removed.

### RTL
- `dir="rtl"`: OFF position of thumb is at inline-end. `inset-inline-start` on `.switch__thumb-container` flips automatically.
- No asset flip needed; check and cross icons are direction-neutral.

---

## CSS Architecture

```css
@layer kafui {
  .switch {
    /* ── component-internal vars ── */
    --track-w:     52px;
    --track-h:     32px;
    --handle-rest: 16px;   /* unselected resting */
    --handle-sel:  24px;   /* selected resting */
    --handle-press: 28px;  /* pressed/dragged */
    --sl:          40px;   /* state-layer diameter */
    --border-w:    2px;

    /* current state colors — toggled via modifier classes */
    --sl-color:    var(--on-surface);
    --handle-color: var(--outline);
    --handle-size: var(--handle-rest);

    display: inline-flex;
    align-items: center;
    gap: 16px;
    cursor: pointer;
    -webkit-tap-highlight-color: transparent;
  }

  .switch[data-label-placement="start"] { flex-direction: row-reverse; }

  /* ── track ── */
  .switch__track {
    width: var(--track-w);
    height: var(--track-h);
    border-radius: var(--corner-full);
    position: relative;
    flex-shrink: 0;
  }

  .switch__track-bg {
    position: absolute;
    inset: 0;
    border-radius: inherit;
    background: var(--surface-container-highest);
    border: var(--border-w) solid var(--outline);
    transition:
      background var(--duration-short3) var(--easing-standard),
      border-color var(--duration-short3) var(--easing-standard);
  }

  /* ── thumb container ── */
  .switch__thumb-container {
    position: absolute;
    width: var(--sl);
    height: var(--sl);
    top: calc((var(--track-h) - var(--sl)) / 2);
    inset-inline-start: calc(var(--border-w) - (var(--sl) - var(--handle-size)) / 2);
    display: flex;
    align-items: center;
    justify-content: center;
    border-radius: 50%;
    transition: inset-inline-start var(--duration-short3) var(--easing-standard);
  }

  /* ── state-layer ── */
  .switch__state-layer {
    position: absolute;
    inset: 0;
    border-radius: 50%;
    background: var(--sl-color);
    opacity: 0;
    pointer-events: none;
    transition: opacity 150ms var(--easing-standard);
  }

  /* ── handle ── */
  .switch__handle {
    width: var(--handle-size);
    height: var(--handle-size);
    border-radius: 50%;
    background: var(--handle-color);
    display: flex;
    align-items: center;
    justify-content: center;
    transition:
      width var(--duration-short3) var(--easing-standard),
      height var(--duration-short3) var(--easing-standard),
      background var(--duration-short3) var(--easing-standard);
    position: relative;
    z-index: 1;
  }

  /* ── icons ── */
  .switch__icon--checked,
  .switch__icon--unchecked {
    position: absolute;
    width: 16px;
    height: 16px;
    opacity: 0;
    transition: opacity var(--duration-short3) var(--easing-standard);
    pointer-events: none;
  }
  .switch__icon--checked   { color: var(--on-primary-container); }
  .switch__icon--unchecked { color: var(--on-surface-variant); }

  /* ── selected state ── */
  .switch--selected {
    --sl-color:     var(--primary);
    --handle-color: var(--on-primary);
    --handle-size:  var(--handle-sel);
  }
  .switch--selected .switch__track-bg {
    background: var(--primary);
    border-color: transparent;
  }
  .switch--selected .switch__thumb-container {
    inset-inline-start: calc(
      var(--track-w) - var(--border-w) - var(--handle-sel) - (var(--sl) - var(--handle-sel)) / 2
    );
  }
  .switch--selected .switch__icon--checked { opacity: 1; }

  /* ── hover / focus / pressed handle color overrides ── */
  .switch[data-hovered]:not(.switch--selected) { --handle-color: var(--on-surface-variant); }
  .switch[data-hovered].switch--selected        { --handle-color: var(--on-primary-container); }
  .switch[data-pressed]                         { --handle-size: var(--handle-press); }
  .switch[data-pressed]:not(.switch--selected)  { --handle-color: var(--on-surface-variant); }
  .switch[data-pressed].switch--selected        { --handle-color: var(--on-primary-container); }

  /* ── interaction state-layer opacity ── */
  .switch[data-hovered]      .switch__state-layer { opacity: var(--state-hover); }
  .switch[data-focus-visible] .switch__state-layer { opacity: var(--state-focus); }
  .switch[data-pressed]      .switch__state-layer { opacity: var(--state-pressed); }

  /* ── icon both variant ── */
  .switch--icon-both:not(.switch--selected) .switch__icon--unchecked { opacity: 1; }

  /* ── disabled ── */
  .switch--disabled {
    cursor: not-allowed;
    pointer-events: none;
  }
  .switch--disabled .switch__track-bg {
    background: color-mix(in srgb, var(--on-surface) 12%, transparent);
    border-color: color-mix(in srgb, var(--on-surface) 12%, transparent);
  }
  .switch--disabled:not(.switch--selected) .switch__handle {
    background: color-mix(in srgb, var(--on-surface) 38%, transparent);
  }
  .switch--disabled.switch--selected .switch__handle {
    background: var(--surface);
  }
  .switch--disabled .switch__icon--checked,
  .switch--disabled .switch__icon--unchecked {
    opacity: 0.38;
  }

  /* ── label ── */
  .switch__label {
    font: var(--body-large-font);
    color: var(--on-surface);
    user-select: none;
  }

  /* ── reduced motion ── */
  @media (prefers-reduced-motion: reduce) {
    .switch__thumb-container { transition: none; }
    .switch__handle {
      transition:
        width var(--duration-short3),
        height var(--duration-short3);
    }
  }
}
```

---

## Proposed kafUI React API

```tsx
interface SwitchProps {
  // Value
  isSelected?: boolean;           // controlled
  defaultSelected?: boolean;      // uncontrolled; default false
  onChange?: (isSelected: boolean) => void;

  // State
  isDisabled?: boolean;
  isReadOnly?: boolean;

  // Icons
  /**
   * Controls which icons appear in the thumb.
   * "none"  — plain thumb (default)
   * "check" — checkmark when selected; nothing when unselected
   * "both"  — checkmark when selected; X when unselected
   */
  iconVariant?: "none" | "check" | "both";
  /** Replace the default checkmark SVG with custom content */
  checkedIcon?: React.ReactNode;
  /** Replace the default cross SVG (only relevant when iconVariant="both") */
  uncheckedIcon?: React.ReactNode;

  // Label
  /**
   * Visible label text or element.
   * When omitted, aria-label is required.
   */
  children?: React.ReactNode;
  /** @default "end" */
  labelPlacement?: "start" | "end";

  // Form
  name?: string;
  /** Form submission value when selected. Default: "on" (HTML default). */
  value?: string;

  // ARIA
  "aria-label"?: string;
  "aria-labelledby"?: string;
  "aria-describedby"?: string;
}
```

**BEM classes emitted:**

```
.switch                         ← root <label>
  .switch--selected
  .switch--disabled
  .switch--icon-check            ← iconVariant="check"
  .switch--icon-both             ← iconVariant="both"

  .switch__track
    .switch__track-bg
    .switch__thumb-container     ← data-selected, data-pressed, data-hovered, data-focus-visible
      .switch__state-layer
      .switch__handle
        .switch__icon--checked
        .switch__icon--unchecked

  .switch__label
```

**React Aria primitives used:**
- `Switch` from `react-aria-components` — `<label>` + hidden `<input type="checkbox" role="switch">`; provides `isSelected`, `isDisabled`, `isHovered`, `isFocusVisible`, `isPressed` via render props.

**Design decisions / deviations:**

- Handle size morph is fully CSS-driven via `--handle-size` custom property toggled by `[data-pressed]` and `.switch--selected`. React only sets data attributes and class modifiers — no JS size computation.
- `inset-inline-start` for thumb positioning means RTL works automatically; no transform-based positioning that would need manual RTL math.
- `iconVariant` is kafUI-specific; RAC `Switch` has no icon concept — kafUI renders icon `<span>` elements conditionally and controls opacity via CSS state.
- `labelPlacement` uses `data-label-placement` attribute (not BEM modifier) for pure CSS handling.
- Drag-to-toggle is opt-in via `enableDrag` prop (not in initial release). Implementation uses a `useMove` hook from RAC.
- `isReadOnly` is passed through to RAC; renders `aria-readonly="true"` and prevents interaction while keeping the control focusable and announcing its state.
- The `label` prop from the original draft is removed in favor of `children` — consistent with all other kafUI form controls. If the consumer wants a visible label, they pass it as children.
