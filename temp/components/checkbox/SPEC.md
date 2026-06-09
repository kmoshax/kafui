# Checkbox

M3 category: **Selection controls** (alongside Radio Button and Switch). A checkbox lets users select one or more items from a list, or turn an option on/off. Supports three states: selected, unselected, and indeterminate (parent of a mixed child group).

---

## Anatomy / Parts

```
[container]
  └─ [state-layer]        — hover/focus/pressed ripple surface (40×40dp circle)
  └─ [checkmark]          — animated SVG path (check ✓ or dash –)
[label]                   — optional, positioned end (LTR) or start (RTL)
[error-indicator]         — visible when isInvalid (outline color → --error token)
```

| Part | BEM element |
|---|---|
| Root wrapper | `.checkbox` |
| Outer box | `.checkbox__container` |
| State-layer circle | `.checkbox__state-layer` |
| SVG check/dash | `.checkbox__checkmark` |
| Text label | `.checkbox__label` |

BEM modifiers on `.checkbox`:
- `--selected`, `--indeterminate`, `--unselected` (default)
- `--error`
- `--disabled`

Label placement is driven by a `data-label-placement` attribute on root; no extra BEM modifier needed.

---

## Variants

Checkbox has no distinct named variants in M3 (unlike Button). State drives appearance:

| State | Visual |
|---|---|
| Unselected | Square outline only (`--outline-variant` token; spec uses `--on-surface-variant` for the 2dp border) |
| Selected | Filled box (`--primary`), white checkmark (`--on-primary`) |
| Indeterminate | Filled box (`--primary`), dash (`--on-primary`) |
| Error (unselected) | Outline color → `--error` |
| Error (selected) | Fill → `--error`, checkmark → `--on-error` |
| Disabled | `--on-surface` at 38% opacity; no interaction |

A `CheckboxGroup` wraps multiple checkboxes with a shared label and manages the group's form context. **Parent "select-all" indeterminate state is consumer-managed** — M3 and RAC both leave this wiring to app code.

---

## States

| State | Token applied |
|---|---|
| Hover | State-layer at `--state-hover` opacity (8%) tinted `--on-surface` (unselected) / `--primary` (selected) |
| Focus visible | State-layer at `--state-focus` opacity (10%); system focus ring |
| Pressed | State-layer at `--state-pressed` opacity (10%); container scales 1 → 0.9 |
| Dragged | State-layer at `--state-dragged` opacity (16%) tinted `--primary` |
| Disabled unselected | Border: `--on-surface` at 38% |
| Disabled selected | Fill: `--on-surface` at 38%; checkmark: `--surface` |
| Error | `--error` for border/fill; `--on-error` checkmark; `--error-container` state-layer tint |

State layer is always 40×40dp circle centered on the 18×18dp container box.

---

## Design Tokens

All references are unprefixed system roles as defined in `_TOKENS.md`.

### Color
| Role | Token |
|---|---|
| Selected fill | `--primary` |
| Checkmark | `--on-primary` |
| Unselected outline | `--on-surface-variant` |
| Error fill / outline | `--error` |
| Error checkmark | `--on-error` |
| Error state-layer tint | `--error-container` |
| State-layer (unselected) | `--on-surface` |
| State-layer (selected) | `--primary` |
| Disabled | `--on-surface` @ 38% |
| Disabled selected fill | `--on-surface` @ 38% |
| Disabled checkmark | `--surface` |

### Shape
| Role | Value |
|---|---|
| Container corner | `--corner-extra-small` (2dp) |

### Typography
| Role | Value |
|---|---|
| Label | `--label-large-*` (label-large typescale bundle) |

### Motion
| Token | Usage |
|---|---|
| `--duration-short3` (200 ms) | Check/uncheck fill + checkmark transition |
| `--easing-standard` | Container fill, checkmark SVG path draw |

Reduced motion: disable SVG `stroke-dashoffset` animation and container fill transition; preserve instant opacity toggle.

### Elevation
No elevation on checkbox.

---

## Interaction & Accessibility

**ARIA roles**
- Individual: `role="checkbox"` with `aria-checked="true|false|mixed"` — RAC `Checkbox` injects from `isIndeterminate` automatically.
- Group: `role="group"` on the wrapper, `aria-labelledby` → group label. RAC `CheckboxGroup` provides this.

**Keyboard**
| Key | Action |
|---|---|
| `Space` | Toggle checked ↔ unchecked; indeterminate → checked |
| `Tab` | Next focusable |
| `Shift+Tab` | Previous focusable |

**React Aria primitives**
- `Checkbox` from `react-aria-components` — `aria-checked`, `aria-required`, `aria-disabled`, `aria-invalid`, `isIndeterminate`, `isSelected`, keyboard interaction.
- `CheckboxGroup` — group `role`, `aria-labelledby`, shared `isDisabled`/`isRequired`, `isInvalid` context, `FieldError` wiring.
- `Label` / `FieldError` / `Text` (slot="description") from RAC for group label, error, and helper text.

**RTL**
Label ordering flips automatically via `flex-direction` + logical margin (`margin-inline-start`). The checkmark SVG is purely geometric — no mirroring needed.

**Error**
`aria-invalid="true"` set when `isInvalid` prop is passed. Error message surfaced via RAC `<FieldError>` linked through `aria-describedby` (RAC manages the ID wiring).

**Reduced motion**
```css
@layer kafui {
  @media (prefers-reduced-motion: reduce) {
    .checkbox__checkmark { transition: none; }
    .checkbox__container { transition: none; }
  }
}
```

---

## CSS Architecture

All rules live in `@layer kafui { … }` for collision safety. Token references use bare custom-property names per `_TOKENS.md` (strip `--md-sys-color-` prefix). Component-internal sizing vars are declared on the block class itself.

```css
@layer kafui {
  .checkbox {
    /* ── component-internal vars ── */
    --box:         18px;      /* container size */
    --sl:          40px;      /* state-layer diameter */
    --sl-offset:   calc((var(--sl) - var(--box)) / -2);  /* centers state-layer */
    --radius:      var(--corner-extra-small);
    --dur:         var(--duration-short3);
    --ease:        var(--easing-standard);

    display: inline-flex;
    align-items: center;
    gap: 4px;
    cursor: pointer;
    -webkit-tap-highlight-color: transparent;
  }

  /* label placement */
  .checkbox[data-label-placement="start"] {
    flex-direction: row-reverse;
  }

  .checkbox__container {
    position: relative;
    width: var(--box);
    height: var(--box);
    flex-shrink: 0;
    border-radius: var(--radius);
    border: 2px solid var(--on-surface-variant);
    background: transparent;
    transition: background var(--dur) var(--ease),
                border-color var(--dur) var(--ease);
  }

  /* state-layer */
  .checkbox__state-layer {
    position: absolute;
    inset: var(--sl-offset);
    border-radius: 50%;
    background: var(--on-surface);        /* overridden to --primary when selected */
    opacity: 0;
    pointer-events: none;
    transition: opacity 150ms var(--ease);
  }

  /* ── selected ── */
  .checkbox--selected .checkbox__container,
  .checkbox--indeterminate .checkbox__container {
    background: var(--primary);
    border-color: var(--primary);
  }
  .checkbox--selected .checkbox__state-layer,
  .checkbox--indeterminate .checkbox__state-layer {
    background: var(--primary);
  }

  /* ── error ── */
  .checkbox--error .checkbox__container {
    border-color: var(--error);
  }
  .checkbox--error.checkbox--selected .checkbox__container,
  .checkbox--error.checkbox--indeterminate .checkbox__container {
    background: var(--error);
    border-color: var(--error);
  }
  .checkbox--error .checkbox__state-layer {
    background: var(--error-container);
  }

  /* ── disabled ── */
  .checkbox--disabled {
    cursor: not-allowed;
    pointer-events: none;
  }
  .checkbox--disabled .checkbox__container {
    border-color: color-mix(in srgb, var(--on-surface) 38%, transparent);
  }
  .checkbox--disabled.checkbox--selected .checkbox__container,
  .checkbox--disabled.checkbox--indeterminate .checkbox__container {
    background: color-mix(in srgb, var(--on-surface) 38%, transparent);
    border-color: transparent;
  }

  /* ── interaction states (driven by RAC data-* attrs) ── */
  .checkbox[data-hovered] .checkbox__state-layer   { opacity: var(--state-hover); }
  .checkbox[data-focus-visible] .checkbox__state-layer { opacity: var(--state-focus); }
  .checkbox[data-pressed] .checkbox__state-layer   { opacity: var(--state-pressed); }
  .checkbox[data-pressed] .checkbox__container     { transform: scale(0.9); }

  /* ── checkmark SVG ── */
  .checkbox__checkmark {
    position: absolute;
    inset: 0;
    stroke: var(--on-primary);
    stroke-width: 2;
    fill: none;
    stroke-dasharray: 24;
    stroke-dashoffset: 24;  /* fully hidden; animates to 0 when selected */
    transition: stroke-dashoffset var(--dur) var(--ease);
  }
  .checkbox--selected .checkbox__checkmark  { stroke-dashoffset: 0; }
  .checkbox--error.checkbox--selected .checkbox__checkmark { stroke: var(--on-error); }
  .checkbox--disabled.checkbox--selected .checkbox__checkmark { stroke: var(--surface); }

  /* ── label ── */
  .checkbox__label {
    font: var(--label-large-font);  /* from typescale bundle */
    color: var(--on-surface);
    user-select: none;
  }
  .checkbox--disabled .checkbox__label {
    color: color-mix(in srgb, var(--on-surface) 38%, transparent);
  }

  /* ── reduced motion ── */
  @media (prefers-reduced-motion: reduce) {
    .checkbox__checkmark,
    .checkbox__container { transition: none; }
  }
}
```

---

## Proposed kafUI React API

```tsx
import {
  Checkbox as AriaCheckbox,
  CheckboxGroup as AriaCheckboxGroup,
  Label,
  FieldError,
  Text,
} from "react-aria-components";

// ── Single Checkbox ──────────────────────────────────────────
interface CheckboxProps {
  /** Controlled checked state */
  isSelected?: boolean;
  defaultSelected?: boolean;
  onChange?: (isSelected: boolean) => void;
  /** Shows dash indicator; marks a parent "select-all" checkbox as partial */
  isIndeterminate?: boolean;
  isDisabled?: boolean;
  isRequired?: boolean;
  /** Marks field invalid; renders --error token colors + aria-invalid */
  isInvalid?: boolean;
  /**
   * Position of visible label text relative to control.
   * Implemented via data-label-placement attribute; pure CSS flex-direction flip.
   * @default "end"
   */
  labelPlacement?: "end" | "start";
  /** Required when no visible label child is provided */
  "aria-label"?: string;
  /** Pass-through for aria linking */
  "aria-describedby"?: string;
  /** form value when used inside CheckboxGroup */
  value?: string;
  children?: React.ReactNode;
}

function Checkbox(props: CheckboxProps): JSX.Element;

// ── Checkbox Group ────────────────────────────────────────────
interface CheckboxGroupProps {
  /** Visible group label (required for a11y unless aria-label provided) */
  label?: React.ReactNode;
  /** Helper text shown below group */
  description?: React.ReactNode;
  /** Error message shown when isInvalid; prefer React.ReactNode for rich content */
  errorMessage?: React.ReactNode;
  isDisabled?: boolean;
  isRequired?: boolean;
  isInvalid?: boolean;
  /** Controlled value — array of selected item values */
  value?: string[];
  defaultValue?: string[];
  onChange?: (value: string[]) => void;
  /** @default "vertical" */
  orientation?: "vertical" | "horizontal";
  children: React.ReactNode;
  "aria-label"?: string;
}

function CheckboxGroup(props: CheckboxGroupProps): JSX.Element;
```

**BEM classes emitted by React:**

| Element | Class |
|---|---|
| Root | `.checkbox` |
| Selected | `.checkbox--selected` |
| Unselected | `.checkbox--unselected` |
| Indeterminate | `.checkbox--indeterminate` |
| Error | `.checkbox--error` |
| Disabled | `.checkbox--disabled` |
| Container box | `.checkbox__container` |
| State-layer | `.checkbox__state-layer` |
| SVG checkmark | `.checkbox__checkmark` |
| Label text | `.checkbox__label` |
| Group root | `.checkbox-group` |
| Group label | `.checkbox-group__label` |
| Group description | `.checkbox-group__description` |
| Group error | `.checkbox-group__error` |
| Horizontal group | `.checkbox-group--horizontal` |

**Usage examples:**

```tsx
// Uncontrolled single:
<Checkbox defaultSelected>Accept terms</Checkbox>

// Controlled with error:
<Checkbox isSelected={checked} onChange={setChecked} isInvalid>
  I agree
</Checkbox>

// Group (controlled):
<CheckboxGroup
  label="Toppings"
  value={selected}
  onChange={setSelected}
  isInvalid={hasError}
  errorMessage="Select at least one topping"
>
  <Checkbox value="cheese">Cheese</Checkbox>
  <Checkbox value="onion">Onion</Checkbox>
  <Checkbox value="peppers">Peppers</Checkbox>
</CheckboxGroup>

// Select-all with indeterminate (consumer-managed):
const all = ["cheese", "onion", "peppers"];
const someSelected = selected.length > 0 && selected.length < all.length;
<Checkbox
  isIndeterminate={someSelected}
  isSelected={selected.length === all.length}
  onChange={(v) => setSelected(v ? all : [])}
>
  All toppings
</Checkbox>
```

**Design decisions / deviations:**

- `labelPlacement` is implemented via `data-label-placement` attribute on root (`<label>`) so CSS handles the visual reorder without any JS layout logic. No BEM modifier needed.
- `isInvalid` follows RAC naming convention (not M3's hypothetical `error` boolean) for consistent API across all kafUI form controls.
- `indeterminate` parent state is consumer-managed. RAC `CheckboxGroup` does not auto-manage a parent checkbox — this is correct per M3 spec; the app owns the "select all" relationship.
- `errorMessage` typed as `React.ReactNode` (not `string`) so consumers can pass localized components, icons, or rich content without reimplementing the container.
- `CheckboxGroup` description slot uses `<Text slot="description">` from RAC which auto-wires `aria-describedby`; no manual ID management.
- RAC `CheckboxGroup` propagates `isInvalid` to children via context, setting `aria-invalid` on each individual checkbox input. This is correct M3 + WCAG behavior — each item in an invalid group should be marked invalid.
