# Radio Button

**M3 category:** Selection — lets users select exactly one option from a set. Always used inside a `RadioGroup`; a lone `Radio` without a group is semantically invalid.

---

## Anatomy / Parts

```
RadioGroup
└── Radio (× N)
    ├── .radio__container          — 40×40dp touch target; positions state-layer + circles
    │   ├── .radio__state-layer    — 40×40dp ripple / hover / focus overlay
    │   ├── .radio__outer-circle   — 20dp ring; stroke only when unselected; ring+border when selected
    │   └── .radio__inner-dot      — 10dp filled dot; scale(0) unselected → scale(1) selected
    └── .radio__label              — adjacent text label (optional)
```

| BEM element | Role |
|---|---|
| `.radio-group` | `<div role="radiogroup">` |
| `.radio-group__label` | Visible group label |
| `.radio-group__items` | Layout container for radio items |
| `.radio-group__description` | Helper text below group |
| `.radio-group__error` | Error message; visible when `isInvalid` |
| `.radio` | Per-item root `<label>` element |
| `.radio__container` | 40dp hit-area; flex-center |
| `.radio__state-layer` | Absolute inset-0; opacity transitions |
| `.radio__outer-circle` | 20dp ring; color transitions |
| `.radio__inner-dot` | 10dp filled circle; scale + opacity transition |
| `.radio__label` | body-large text; inline with container |

---

## Variants

No `variant` prop — the single visual form is canonical M3. Differentiated only by state:

| Configuration | Notes |
|---|---|
| Unselected | Default; stroke ring only |
| Selected | Ring + inner dot |
| With label | Natural child content; `labelPlacement="start|end"` |
| Error | `isInvalid` on the **group** only; ring + dot + state-layer switch to `--error` token |
| Disabled | 38% opacity; no interaction |

---

## States

| State | Visual |
|---|---|
| **Unselected** | Outer circle: `--on-surface-variant` border; no inner dot |
| **Selected** | Outer circle: `--primary` filled ring + border; inner dot: `--primary` |
| **Hover** | State-layer at `--state-hover` (8%) tinted `--on-surface` (unselected) / `--primary` (selected) |
| **Focus visible** | State-layer at `--state-focus` (10%); system focus ring on container |
| **Pressed** | State-layer at `--state-pressed` (10%) + spring ripple; outer circle scales slightly inward |
| **Disabled** | `--on-surface` at 38% opacity on circle, dot, and label; no interaction |
| **Error (group)** | Outer circle + inner dot use `--error`; state-layer tint = `--error` |

State-layer color is always on a separate element; the circle geometry is never tinted by interaction.

**Focus opacity correction:** M3 spec lists focus opacity as 0.10 for radio button (same as pressed). The SPEC previously listed 0.12 — corrected here to match the authoritative M3 state-layer table (which uses 0.10 for both focus and pressed on selection controls). Use `--state-focus` token which resolves to 0.10.

---

## Design Tokens

All references are unprefixed system roles per `_TOKENS.md`.

| Token | Usage |
|---|---|
| `--primary` | Selected ring + dot |
| `--on-surface-variant` | Unselected ring border |
| `--on-surface` | Disabled tint; unselected hover/focus state-layer color |
| `--error` | Error ring, error dot, error state-layer tint |
| `--corner-full` | `border-radius: 50%` on circles |
| `--body-large-*` | Label text typescale |
| `--duration-short2` | Inner-dot scale transition (~100 ms) |
| `--easing-standard` | Inner-dot easing |
| `--state-hover` | 0.08 |
| `--state-focus` | 0.10 |
| `--state-pressed` | 0.10 |

---

## Interaction & Accessibility

### ARIA roles
- `RadioGroup` root → `role="radiogroup"`; requires `aria-label` or `aria-labelledby` (RAC enforces via `Label`).
- Each `Radio` → `role="radio"`, `aria-checked="true|false"`, `aria-disabled` when disabled.
- `aria-invalid` on each radio **input** when the group `isInvalid` (RAC propagates via context) — not just on the group wrapper.

### Keyboard
| Key | Action |
|---|---|
| `Tab` | Focus enters/leaves the group as a single tab stop (roving tabindex via RAC) |
| `Arrow Down` / `Arrow Right` | Move selection to next option (wrap) |
| `Arrow Up` / `Arrow Left` | Move selection to previous option (wrap) |
| `Space` | Select focused option if not already selected |

Note: RAC `RadioGroup` implements roving tabindex so only the selected option (or first if none selected) is in the tab order. Arrow keys cycle within the group. This is the correct ARIA radio button pattern.

### Touch / pointer
- Minimum 40×40dp interactive target via `.radio__container`; the `<label>` element extends tap area over the label text achieving well over 48dp total.
- `onPress` (React Aria); not `onClick`.

### Reduced motion
```css
@layer kafui {
  @media (prefers-reduced-motion: reduce) {
    .radio__inner-dot { transition: none; }
    .radio__outer-circle { transition: none; }
    .radio__state-layer { transition: none; }
  }
}
```

### RTL
- `labelPlacement="start"` reverses flex order via `[data-label-placement="start"] { flex-direction: row-reverse }` and logical margins — pure CSS, no JS.
- Circles are symmetric; no asset flip needed.

---

## CSS Architecture

```css
@layer kafui {
  .radio {
    /* ── component-internal vars ── */
    --ring:        20px;   /* outer circle diameter */
    --dot:         10px;   /* inner dot diameter */
    --sl:          40px;   /* state-layer diameter */
    --dur:         var(--duration-short2);
    --ease:        var(--easing-standard);

    /* current state-layer color — toggled by selected/error state */
    --sl-color:    var(--on-surface);

    display: inline-flex;
    align-items: center;
    gap: 12px;
    cursor: pointer;
    -webkit-tap-highlight-color: transparent;
  }

  .radio[data-label-placement="start"] {
    flex-direction: row-reverse;
  }

  .radio__container {
    position: relative;
    width: var(--sl);
    height: var(--sl);
    flex-shrink: 0;
    display: flex;
    align-items: center;
    justify-content: center;
    border-radius: 50%;
  }

  .radio__state-layer {
    position: absolute;
    inset: 0;
    border-radius: 50%;
    background: var(--sl-color);
    opacity: 0;
    pointer-events: none;
    transition: opacity 150ms var(--ease);
  }

  .radio__outer-circle {
    width: var(--ring);
    height: var(--ring);
    border-radius: 50%;
    border: 2px solid var(--on-surface-variant);
    box-sizing: border-box;
    display: flex;
    align-items: center;
    justify-content: center;
    transition: border-color var(--dur) var(--ease),
                background var(--dur) var(--ease);
  }

  .radio__inner-dot {
    width: var(--dot);
    height: var(--dot);
    border-radius: 50%;
    background: var(--primary);
    transform: scale(0);
    transition: transform var(--dur) var(--ease);
  }

  /* ── selected ── */
  .radio--selected {
    --sl-color: var(--primary);
  }
  .radio--selected .radio__outer-circle {
    border-color: var(--primary);
  }
  .radio--selected .radio__inner-dot {
    transform: scale(1);
  }

  /* ── error (set on .radio when group isInvalid) ── */
  .radio--error {
    --sl-color: var(--error);
  }
  .radio--error .radio__outer-circle {
    border-color: var(--error);
  }
  .radio--error .radio__inner-dot {
    background: var(--error);
  }

  /* ── disabled ── */
  .radio--disabled {
    cursor: not-allowed;
    pointer-events: none;
  }
  .radio--disabled .radio__outer-circle {
    border-color: color-mix(in srgb, var(--on-surface) 38%, transparent);
  }
  .radio--disabled .radio__inner-dot {
    background: color-mix(in srgb, var(--on-surface) 38%, transparent);
  }
  .radio--disabled .radio__label {
    color: color-mix(in srgb, var(--on-surface) 38%, transparent);
  }

  /* ── interaction states ── */
  .radio[data-hovered]      .radio__state-layer { opacity: var(--state-hover); }
  .radio[data-focus-visible] .radio__state-layer { opacity: var(--state-focus); }
  .radio[data-pressed]      .radio__state-layer { opacity: var(--state-pressed); }

  /* ── label ── */
  .radio__label {
    font: var(--body-large-font);
    color: var(--on-surface);
    user-select: none;
  }

  /* ── group layout ── */
  .radio-group {
    display: flex;
    flex-direction: column;
    gap: 4px;
  }
  .radio-group--horizontal .radio-group__items {
    flex-direction: row;
    flex-wrap: wrap;
    gap: 16px;
  }
  .radio-group__items {
    display: flex;
    flex-direction: column;
    gap: 4px;
  }
  .radio-group__label {
    font: var(--body-large-font);
    color: var(--on-surface);
    margin-block-end: 8px;
  }
  .radio-group__description {
    font: var(--body-medium-font);
    color: var(--on-surface-variant);
    margin-block-start: 4px;
  }
  .radio-group__error {
    font: var(--body-small-font);
    color: var(--error);
    margin-block-start: 4px;
  }

  /* ── reduced motion ── */
  @media (prefers-reduced-motion: reduce) {
    .radio__inner-dot,
    .radio__outer-circle,
    .radio__state-layer { transition: none; }
  }
}
```

---

## Proposed kafUI React API

```tsx
// ── RadioGroup wrapper ────────────────────────────────────────
interface RadioGroupProps {
  /** Visible label (required for a11y unless aria-label provided) */
  label?: React.ReactNode;
  /** Helper text */
  description?: React.ReactNode;
  /** Error message shown when isInvalid */
  errorMessage?: React.ReactNode;
  /** Controlled selected value */
  value?: string;
  defaultValue?: string;
  onChange?: (value: string) => void;
  isDisabled?: boolean;
  isInvalid?: boolean;
  isRequired?: boolean;
  /** @default "vertical" */
  orientation?: "vertical" | "horizontal";
  children: React.ReactNode;
  "aria-label"?: string;
  "aria-labelledby"?: string;
  "aria-describedby"?: string;
}

// ── Individual radio ──────────────────────────────────────────
interface RadioProps {
  /** Required; must be unique within the group */
  value: string;
  /** Overrides group-level isDisabled for this item only */
  isDisabled?: boolean;
  /**
   * Position of label text relative to circle.
   * Implemented via data-label-placement; pure CSS flip.
   * @default "end"
   */
  labelPlacement?: "start" | "end";
  children?: React.ReactNode;
}
```

**BEM classes emitted:**

```
.radio-group                       ← <div role="radiogroup">
  .radio-group--horizontal
  .radio-group__label
  .radio-group__items
  .radio-group__description
  .radio-group__error               ← visible when isInvalid

.radio                              ← per-item <label>
  .radio--selected
  .radio--disabled
  .radio--error                     ← inherited from group isInvalid
  .radio__container
    .radio__state-layer
    .radio__outer-circle
    .radio__inner-dot
  .radio__label
```

**React Aria primitives used:**
- `RadioGroup` from `react-aria-components` — roving tabindex, `role="radiogroup"`, keyboard navigation, `isInvalid` propagation via context.
- `Radio` from `react-aria-components` — `role="radio"`, `aria-checked`, `aria-disabled`, press/hover/focus state props via render props.
- `Label` / `Text` (slot="description") / `FieldError` from RAC for group label, description, and error message.

**Usage examples:**

```tsx
// Vertical group (default):
<RadioGroup label="Shipping" value={shipping} onChange={setShipping}>
  <Radio value="standard">Standard (5–7 days)</Radio>
  <Radio value="express">Express (2–3 days)</Radio>
  <Radio value="overnight">Overnight</Radio>
</RadioGroup>

// Horizontal:
<RadioGroup label="Size" orientation="horizontal">
  <Radio value="s">S</Radio>
  <Radio value="m">M</Radio>
  <Radio value="l">L</Radio>
</RadioGroup>

// Error state:
<RadioGroup
  label="Agreement"
  isInvalid={!accepted}
  errorMessage="You must select an option"
>
  <Radio value="yes">I agree</Radio>
  <Radio value="no">I disagree</Radio>
</RadioGroup>
```

**Design decisions / deviations:**

- `isInvalid` lives on the **group only** — individual `Radio` items inherit error color via the CSS custom property `--sl-color` scoped to `.radio--error` modifier class. The JS component applies `.radio--error` to each child when the group is invalid. This is both simpler API and more correct semantically.
- `errorMessage` typed as `React.ReactNode` for rich content support.
- `labelPlacement` on `Radio` uses `data-label-placement` attribute (not a BEM modifier) — pure CSS flex reorder, no JS computation.
- The `Radio` `label` prop from the earlier draft is removed — use `children` only. Having both `label` and `children` creates ambiguity. `children` is the canonical React pattern.
- `description` slot on `RadioGroup` uses `<Text slot="description">` from RAC (auto-wires `aria-describedby`).
