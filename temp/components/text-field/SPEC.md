# Text Field

**Purpose:** Single-line and multiline text input that captures user-entered values. The floating label provides context at all times; supporting text, character counters, and error messages communicate validity. M3 category: **Text inputs → Text field**.

---

## Anatomy / Parts → BEM Elements

```
.text-field                       Root container (div). Holds all parts; carries variant modifier.
.text-field__container            Inner box: the filled surface or the outlined border box.
.state-layer                      Hover/focus tint overlay (filled variant only, covers container).
.text-field__leading-icon         Optional icon before the input area.
.text-field__label                Floating label (<label>). Transitions between resting and floating positions.
.text-field__prefix               Static text before input content (e.g. "$"). Visible only when focused or filled.
.text-field__input                The <input> or <textarea> element.
.text-field__suffix               Static text after input content (e.g. ".com"). Visible only when focused or filled.
.text-field__trailing-icon        Optional icon after the input/suffix area.
.text-field__active-indicator     Bottom border line (filled variant only). Thickens on focus.
.text-field__outline              Container element for the three-segment outlined border.
.text-field__notch-leading        Inline-start segment of the three-part outlined border.
.text-field__notch                Middle segment of the outlined border that the label floats into.
.text-field__notch-trailing       Inline-end segment of the three-part outlined border.
.text-field__supporting-text-row  Row below the container: holds supporting/error text + counter.
.text-field__supporting-text      Helper or error message.
.text-field__character-counter    Character-count indicator (inline-end aligned).
```

**Multiline:** `.text-field--multiline` replaces `.text-field__input` (`<input>`) with a `<textarea>`. The container grows with content (auto-resize via `field-sizing: content` or a `ResizeObserver` JS fallback).

---

## Variants

| Variant | `variant` prop | Container surface | Bottom border | Outer border |
|---|---|---|---|---|
| Filled | `"filled"` | `surface-container-highest` | `active-indicator` (1 dp resting, 2 dp focused) | Top corners rounded (`corner-extra-small`), bottom corners 0 |
| Outlined | `"outlined"` | transparent | none | Full rect border (`outline` color, 1 dp resting, 2 dp focused), all corners rounded (`corner-extra-small`) |

M3 dropped the legacy "standard" (underline-only, no container) as a primary variant — kafUI does not implement it.

---

## States

| State | Label position | Active-indicator / outline | Container / state-layer |
|---|---|---|---|
| **Enabled (empty)** | Resting: vertically centered in container, `body-large` type, `on-surface-variant` color | 1 dp, `on-surface-variant` | Filled: `surface-container-highest`; outlined: transparent |
| **Hovered (empty)** | Resting | 1 dp, `on-surface` | Filled: + 8% `on-surface` state layer; outlined: border → `on-surface` |
| **Focused (empty)** | Floating (top of container / notch), `body-small` type, `primary` color | 2 dp / 2 dp border, `primary` | Filled: + 10% `primary` state layer on container |
| **Filled / has value** | Floating always, `body-small`, `on-surface-variant` (not focused), `primary` (focused) | Same as enabled/focused | |
| **Error (any)** | Floating (if has value or focused), `body-small`, `error` color | `error` color throughout; trailing icon auto-swaps to error icon | Outlined: `error` border |
| **Disabled** | Resting (empty) / floating (has value); 38% `on-surface` | 1 dp dashed, 12% `on-surface` | Filled: 4% `on-surface` container; no state layer; `pointer-events: none` |

### Label float behavior

- **Resting:** `transform: translateY(0) scale(1)`. Centered vertically in the input area.
- **Floating:** `transform: translateY(var(--label-dy)) scale(0.75)`. Filled: label lifts to the top interior edge. Outlined: label lifts into the `__notch` gap.
- Transition: `transform` + `color` wrapped in `@media (prefers-reduced-motion: no-preference)` using `--duration-short2` and `--easing-standard`.
- Float is triggered when: input has `data-focused` OR `data-has-value` OR `placeholder` is set.
- `@media (prefers-reduced-motion: reduce)`: instant jump, no transition.

### Notched outline mechanic

The outlined border is **not** a `<fieldset>`/`<legend>` element. It uses three CSS `border` segments:

```
[ __notch-leading ][ __notch (gap for label) ][ __notch-trailing ]
```

The `__notch` segment has:
- `border-top: none` while label is floating (gap appears).
- `border-top: 1dp solid` while label is resting (gap closes).
- `width` driven by `--notch-w` (component-scoped custom property), set once via `ResizeObserver` measuring `label.scrollWidth × 0.75 + 8 dp` per side. SSR fallback: inline style from character-width estimation or explicit `notchWidth` prop.

This approach avoids `<fieldset>`/`<legend>` quirks (browser-default `min-inline-size: min-content`, `<legend>` sizing, reset stylesheet conflicts, dual DOM nodes for one label).

---

## Design Tokens

All system tokens are referenced by their unprefixed role name as CSS custom properties. Component-internal variables live inside the `.text-field {}` block.

### Color

| Role | CSS var | Usage |
|---|---|---|
| primary | `--primary` | Active indicator / outline (focused); floating label (focused); focus state layer |
| on-surface | `--on-surface` | Input text value; resting label (hover); active indicator (hover) |
| on-surface-variant | `--on-surface-variant` | Resting label (enabled); resting indicator/outline (enabled); supporting text; prefix/suffix |
| surface-container-highest | `--surface-container-highest` | Filled container background |
| outline | `--outline` | Outlined border (enabled/hover) |
| outline-variant | `--outline-variant` | Disabled outlined border |
| error | `--error` | Error label; active indicator; outline; error supporting text; error trailing icon |

Opacity tints for disabled and state layers use `color-mix(in srgb, var(--on-surface) <pct>%, transparent)` — never hard-coded alpha hex.

### Shape

| Token | CSS var | Usage |
|---|---|---|
| corner-extra-small | `--corner-extra-small` | All corners outlined; top corners only filled (bottom-start and bottom-end = 0) |

```css
@layer kafui {
  .text-field {
    --h: 56px;                  /* container height */
    --label-dy: -22px;          /* translateY for floating label — filled */
    --label-dy-outlined: -28px; /* translateY for outlined float */
    --notch-w: 0px;             /* set via ResizeObserver on label */
    --indicator-h: 1px;         /* resting active-indicator height */
  }
  .text-field[data-focused] {
    --indicator-h: 2px;
  }
}
```

### Typography

| Token | CSS var | Usage |
|---|---|---|
| body-large | `--body-large-size`, `--body-large-weight`, `--body-large-line-height` | Input text; resting label |
| body-small | `--body-small-size`, `--body-small-weight`, `--body-small-line-height` | Floating label; supporting text; character counter |

### Motion

| Token | CSS var | Usage |
|---|---|---|
| duration-short2 | `--duration-short2` | Label float transition (~200 ms) |
| easing-standard | `--easing-standard` | Label translate + scale easing |

### State layers

| Token | CSS var | Value |
|---|---|---|
| state-hover | `--state-hover` | 0.08 |
| state-focus | `--state-focus` | 0.10 |

---

## Interaction & Accessibility

### Label association

The `<label>` MUST be associated with `<input>` / `<textarea>` via `htmlFor` / `id`. React Aria's `TextField` + `Label` + `Input`/`TextArea` wires this automatically — the label id is never managed by hand.

### ARIA

- `aria-describedby`: the `<input>` references the supporting-text id AND the character-counter id (space-separated). On error, `FieldError` appends or replaces. React Aria `TextField` handles this automatically.
- `aria-invalid="true"` set on `<input>` in error state via RAC `isInvalid` prop.
- `aria-required="true"` (or native `required`) via RAC `isRequired`.
- `aria-multiline="true"` is implicit on `<textarea>` — no extra attribute.
- All decorative icons (leading, trailing, error indicator) are `aria-hidden="true"`.

### ARIA note: `validationState` is deprecated in RAC v1

React Aria Components ≥ 1.0 uses `isInvalid: boolean` (not `validationState="invalid"`). The spec uses `isInvalid` throughout.

### Focus ring

M3 specifies that the active-indicator/outline change IS the primary visual focus affordance. No duplicate `outline` ring is shown during normal use. A system `focus-visible` outline (2 dp, offset 2 dp, `primary` color) is rendered only when `data-focus-visible` is present, for high-contrast and AT users.

### Touch target

Minimum 48 dp hit area on leading/trailing icons via `min-width: 48px; min-height: 48px; padding` combination. The field container (56 dp tall) meets this naturally.

### Reduced motion

```css
@layer kafui {
  @media (prefers-reduced-motion: no-preference) {
    .text-field__label {
      transition: transform var(--duration-short2) var(--easing-standard),
                  color var(--duration-short2) var(--easing-standard);
    }
  }
}
```

Under `reduce`: transform and color change instantly.

### RTL

- All spacing via logical properties: `padding-inline-start/end`, `margin-inline-*`, `inset-inline-*`.
- Leading icon at `inline-start`, trailing icon at `inline-end` — no JS directional logic.
- Three-segment notch layout mirrors under `direction: rtl` automatically.
- `text-align: end` on character counter (auto-flips in RTL).

### Character counter localization

Format with `Intl.NumberFormat`: `{count.toLocaleString(locale)} / {max.toLocaleString(locale)}`. Counter span is `aria-hidden="true"` — screen reader gets field context from `aria-describedby`.

---

## Proposed kafUI React API

```tsx
// React Aria primitives:
//   TextField, Label, Input, TextArea, FieldError, Text
// from 'react-aria-components'

type TextFieldVariant = 'filled' | 'outlined';

interface TextFieldProps {
  /** Visual variant. Default: 'filled' */
  variant?: TextFieldVariant;
  /** Floating label text. Always rendered as <label>; never omit. */
  label: string;
  /** Helper text shown below the field in the non-error state. */
  description?: string;
  /**
   * Error message. Rendered via RAC FieldError — auto-wires aria-describedby
   * and aria-invalid. Pass a string or a render function (value) => string.
   */
  errorMessage?: string | ((validation: ValidationResult) => string);
  /** Whether the field is in an invalid state (triggers error styling). */
  isInvalid?: boolean;
  /** Whether a value is required. Sets aria-required on the input. */
  isRequired?: boolean;
  /** Marks field as disabled. */
  isDisabled?: boolean;
  /** Marks field as read-only. */
  isReadOnly?: boolean;
  /** Converts <input> to <textarea>. Default: false */
  multiline?: boolean;
  /** Rows hint for textarea initial height (grows beyond). Default: 3 */
  rows?: number;
  /** Static text before the input value (e.g. "$"). Visible when focused or filled. */
  prefix?: string;
  /** Static text after the input value (e.g. "kg"). Visible when focused or filled. */
  suffix?: string;
  /** Icon name for leading icon slot. */
  leadingIcon?: string;
  /**
   * Icon name for trailing icon slot.
   * Auto-replaced by the error icon in isInvalid state unless suppressErrorIcon is set.
   */
  trailingIcon?: string;
  /** Suppress automatic error icon swap. Default: false */
  suppressErrorIcon?: boolean;
  /** Max character length. Enables character counter. */
  maxLength?: number;
  /** Controlled value. */
  value?: string;
  /** Default value (uncontrolled). */
  defaultValue?: string;
  /** Change handler. */
  onChange?: (value: string) => void;
  onFocus?: (e: FocusEvent) => void;
  onBlur?: (e: FocusEvent) => void;
  /** Placeholder text. When set, label floats even when unfocused/empty. */
  placeholder?: string;
  /** HTML input type. Default: 'text'. Ignored for multiline. */
  type?: 'text' | 'email' | 'search' | 'url' | 'tel' | 'password' | 'number';
  /** Locale for character counter number formatting. Default: navigator.language. */
  locale?: string;
  /** Extra className merged after BEM root class. */
  className?: string;
  // All RAC TextField passthrough props (name, autoComplete, autoFocus, id, etc.)
}

// Usage examples:
<TextField
  variant="outlined"
  label="Email address"
  type="email"
  leadingIcon="mail"
  description="We'll never share your email."
  isRequired
/>

<TextField
  variant="filled"
  label="Bio"
  multiline
  rows={4}
  maxLength={280}
  description="Brief description of yourself."
/>

<TextField
  variant="outlined"
  label="Amount"
  prefix="$"
  suffix="USD"
  trailingIcon="attach_money"
  isInvalid
  errorMessage="Amount must be positive."
/>
```

**Key DX win over MUI:** `description` + `errorMessage` + `isInvalid` replaces MUI's `helperText` + `error={bool}` + `FormHelperText` split across `TextField`/`FormControl`. RAC auto-wires the `aria-describedby` chain — zero manual `id` management.

### BEM classes emitted

```
.text-field                          root always
.text-field--filled                  variant modifier
.text-field--outlined                variant modifier
.text-field--multiline               textarea mode
.text-field--has-leading-icon        shifts label/input padding-inline-start
.text-field--has-trailing-icon       shifts input padding-inline-end
.text-field--has-prefix              prefix visible
.text-field--has-suffix              suffix visible

RAC data attributes on the root and inner <input>/<textarea>
(CSS selectors — no className toggling in JS):
  data-focused           — field or any child has focus
  data-hovered           — pointer inside field
  data-invalid           — isInvalid=true
  data-disabled          — isDisabled=true
  data-required          — isRequired=true
  data-read-only         — isReadOnly=true
  data-focus-visible     — focus arrived by keyboard
  data-has-value         — set by kafUI when value.length > 0 (custom attr)
  data-placeholder-shown — set by kafUI when placeholder prop is provided
```

All CSS state rules use `data-*` attribute selectors. Modifier classes (e.g. `--error`) are **not** needed because the RAC data attributes are present on the root `TextField` wrapper and can be targeted directly.

### React Aria binding details

| kafUI element | RAC component |
|---|---|
| Root wrapper | `TextField` |
| `__label` | `Label` (auto-associated) |
| `__input` | `Input` |
| `__textarea` | `TextArea` |
| `__supporting-text` (helper) | `Text slot="description"` |
| `__supporting-text` (error) | `FieldError` |
| Character counter | `<span aria-hidden="true">` reading from `TextField` render prop value |

### Notched outline — pure CSS / three-div justification

MUI uses `<fieldset><legend>` for the outlined notch gap. kafUI uses three `<div>` segments (`__notch-leading`, `__notch`, `__notch-trailing`) instead because:

1. `<fieldset>` has browser-default `min-inline-size: min-content` which resists `width: 100%` shrinkage and requires explicit `min-inline-size: 0` reset.
2. `<legend>` sizing depends on its content; MUI places a visually-hidden duplicate label inside the `<legend>` to create the gap, and the real visible label is an absolutely-positioned overlay — two DOM nodes for one semantic label.
3. Reset stylesheets (Normalize, Preflight) inconsistently handle `<fieldset>` border rendering across browsers.

kafUI's three-div approach: each segment has its own CSS `border` sides. `__notch` removes `border-block-start` while the label is floating (gap opens); `--notch-w` custom property drives `__notch` width. A `ResizeObserver` sets `--notch-w` once on mount and on label text change; SSR: calculate from an explicit `notchWidth` prop or a character-width approximation.

### CSS layer encapsulation

All component CSS lives in `@layer kafui { … }`. Consumer overrides placed outside a layer win automatically. Brand name surfaces **only** on the layer name — never on classes.

```css
@layer kafui {
  .text-field { … }
  .text-field--outlined { … }
  .text-field__label { … }
  /* etc. */
}
```
