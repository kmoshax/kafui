# Button

Common buttons trigger low-to-high-emphasis actions. M3 category: **Actions → Common Buttons**. M3 Expressive adds five explicit sizes (XS–XL) and a shape-morph animation on press.

---

## Anatomy / parts

| BEM element | Description |
|---|---|
| `.kafui-button` | Root `<button>` element; carries the state layer and ripple |
| `.kafui-button__state-layer` | Absolutely-positioned overlay that renders hover/focus/pressed tints |
| `.kafui-button__icon` | Leading (or trailing) icon slot — `<Icon>` sprite wrapper |
| `.kafui-button__label` | Text label — maps to `md.sys.typescale.label-large` |

The icon is optional. When present it sits before the label (leading) by default; a `trailingIcon` prop flips it after the label.

---

## Variants

| Variant | Container | Label color | Use |
|---|---|---|---|
| `filled` | `primary` | `on-primary` | Highest emphasis, single primary action |
| `filled-tonal` | `secondary-container` | `on-secondary-container` | Medium-high emphasis, secondary action |
| `elevated` | `surface-container-low` + elevation 1 | `primary` | Medium emphasis, needs visual separation |
| `outlined` | transparent + `outline` border | `primary` | Medium emphasis, paired with filled |
| `text` | transparent | `primary` | Lowest emphasis, inline or optional actions |

---

## States

All variants share the same state-layer model applied **on top of the container color**.

| State | State-layer opacity | Additional change |
|---|---|---|
| Enabled | 0% | — |
| Hovered | 8% (state-layer color = label color) | Elevation +1 for `elevated` |
| Focused | 10% | Focus ring (`outline` color, 3dp offset) visible |
| Pressed | 10% | Expressive: shape morph (corner radius briefly increases) |
| Disabled | Container 12% `on-surface`, label 38% `on-surface` | No state layer; not interactive |
| Dragged | 16% | Elevation +4 (elevated variant only during drag scenarios) |

### M3 Expressive — shape morph on press
On `pressed` the corner radius animates from the resting shape to a rounder shape and back using `md.sys.motion.easing.standard` + a short duration (~100 ms). The resting shape is size-dependent (see Sizes below).

---

## Sizes (M3 Expressive)

| Size | Height | Min-width | Icon size | Corner radius (resting) |
|---|---|---|---|---|
| `xs` | 32 dp | — | 16 dp | `full` (pill) |
| `sm` | 36 dp | — | 18 dp | `full` |
| `md` (default) | 40 dp | 48 dp | 18 dp | `full` |
| `lg` | 48 dp | — | 20 dp | `extra-large` |
| `xl` | 56 dp | — | 24 dp | `extra-large` |

For non-Expressive deployments, only `md` is required. The CSS class modifier is `.kafui-button--size-xs` etc.

---

## Design tokens

### Color
- Container (filled): `--md-sys-color-primary`
- Label (filled): `--md-sys-color-on-primary`
- Container (filled-tonal): `--md-sys-color-secondary-container`
- Label (filled-tonal): `--md-sys-color-on-secondary-container`
- Container (elevated): `--md-sys-color-surface-container-low`
- Label (elevated/outlined/text): `--md-sys-color-primary`
- Border (outlined): `--md-sys-color-outline`
- Disabled container: `--md-sys-color-on-surface` at 12%
- Disabled label: `--md-sys-color-on-surface` at 38%

### Shape
- Default (md): `--md-sys-shape-corner-full` (fully rounded = pill)
- lg/xl: `--md-sys-shape-corner-extra-large`

### Typography
- `--md-sys-typescale-label-large-font`, `--md-sys-typescale-label-large-size`, `--md-sys-typescale-label-large-weight`, `--md-sys-typescale-label-large-line-height`

### Elevation
- Elevated resting: level 1 (`--md-sys-elevation-level1` = `0 1px 2px ...`)
- Elevated hover: level 2
- Disabled: level 0

### Motion (Expressive shape morph)
- `--md-sys-motion-easing-standard`
- `--md-sys-motion-duration-short2` (~100 ms)

### State layer
- `--md-sys-state-hover-state-layer-opacity`: 0.08
- `--md-sys-state-focus-state-layer-opacity`: 0.10
- `--md-sys-state-pressed-state-layer-opacity`: 0.10

---

## Interaction & accessibility

**Keyboard:** `Space` and `Enter` activate. Native `<button>` handles this natively.

**Focus:** visible focus ring using `outline` offset from the container edge, using `--md-sys-color-outline` (or `primary` on light surfaces). `focus-visible` CSS pseudo-class only.

**ARIA:**
- Root is a native `<button>` — role `button` is implicit.
- `aria-disabled="true"` (not the HTML `disabled` attr) keeps the element focusable and communicates state to AT; this is React Aria's default behaviour.
- Icon-only buttons (no label) must pass `aria-label`; when a label is present the icon is `aria-hidden="true"`.
- If used as a toggle (rare for plain button), add `aria-pressed`.

**Touch target:** minimum 48×48 dp — achieved by `min-height: 48px` on an invisible touch-target pseudo-element or wrapper, while the visual height may be 40 dp.

**Reduced motion:** the Expressive shape morph uses `@media (prefers-reduced-motion: reduce)` to skip the radius transition.

**RTL:** icon order flips via CSS logical properties or `direction: rtl` inheritance; no JS needed.

---

## Proposed kafUI React API

```tsx
// react-aria-components base: <Button>

type ButtonVariant = 'filled' | 'filled-tonal' | 'elevated' | 'outlined' | 'text';
type ButtonSize = 'xs' | 'sm' | 'md' | 'lg' | 'xl'; // 'md' default

interface ButtonProps {
  variant?: ButtonVariant;          // default: 'filled'
  size?: ButtonSize;                // default: 'md'
  icon?: string;                    // Icon sprite name; renders <Icon>
  trailingIcon?: boolean;           // Move icon after label; default false
  isDisabled?: boolean;             // React Aria convention
  onPress?: PressEvent => void;     // React Aria; NOT onClick
  children: React.ReactNode;        // label text (or ReactNode for flexibility)
  className?: string;               // merged after BEM classes
  // ...all standard RAC Button passthrough props
}
```

**BEM classes emitted:**
- `.kafui-button` (always)
- `.kafui-button--{variant}` e.g. `.kafui-button--filled`
- `.kafui-button--size-{size}` e.g. `.kafui-button--size-md`
- `.kafui-button--icon-only` when no children text provided
- `.kafui-button--trailing-icon` when `trailingIcon={true}`
- RAC data attributes (`data-hovered`, `data-focused`, `data-pressed`, `data-disabled`) drive state-layer CSS rules — no className toggling in JS

**React Aria base:** `Button` from `react-aria-components`. It handles keyboard, pointer, focus-visible, and `aria-disabled` automatically.

**Justification for deviations from MUI:**
- No `color` prop (MUI's `color="secondary"` etc.) — variant encodes both shape and color role per M3 spec.
- No `startIcon`/`endIcon` — consolidated into `icon` + `trailingIcon` flag, simpler API.
- `size` prop is Expressive-native; MUI's `small/medium/large` don't map to M3 sizes.
- No `disableElevation`, `disableRipple` — not M3 concepts; ripple is always present (CSS).
