# Button

Common buttons trigger low-to-high-emphasis actions. M3 category: **Actions → Common Buttons**. M3 Expressive adds five explicit sizes (XS–XL) and a shape-morph animation on press.

---

## Anatomy / parts

| BEM element | Description |
|---|---|
| `.button` | Root `<button>` element; the interactive container |
| `.button__state-layer` | Absolutely-positioned overlay that renders hover/focus/pressed tints |
| `.button__icon` | Leading icon slot — `<Icon>` sprite wrapper; `aria-hidden` |
| `.button__label` | Text label — maps to `label-large` typescale role |

The icon is optional. When present it sits before the label (leading) by default; `trailingIcon` renders it after the label via flex `order`. When there is no label (`children` is absent), the `.button--icon-only` modifier is applied and the component renders at square icon-button dimensions with `aria-label` required.

---

## Variants

| Variant | Container | Label color | Use |
|---|---|---|---|
| `filled` (default) | `primary` | `on-primary` | Highest emphasis, single primary action |
| `filled-tonal` | `secondary-container` | `on-secondary-container` | Medium-high emphasis, alternative action |
| `elevated` | `surface-container-low` + elevation 1 | `primary` | Medium emphasis, separation without a filled surface |
| `outlined` | transparent + `outline` border | `primary` | Medium emphasis, paired with `filled` |
| `text` | transparent | `primary` | Lowest emphasis, inline or optional actions |

---

## States

All variants share the same state-layer model applied on top of the container color. The state-layer color always equals the **label color** for that variant.

| State | State-layer opacity | Additional change |
|---|---|---|
| Enabled | 0% | — |
| Hovered | `--state-hover` (0.08) | `elevated` gains +1 elevation level |
| Focused | `--state-focus` (0.10) | Focus ring visible (`outline` color, 3 dp offset) |
| Pressed | `--state-pressed` (0.10) | Expressive: shape morph (see below) |
| Disabled | — | Container: `on-surface` at 12%; label: `on-surface` at 38%; no state layer |
| Dragged | `--state-dragged` (0.16) | `elevated` only; elevation +4 |

### M3 Expressive — shape morph on press

On `data-pressed` the corner radius transitions from resting shape to a slightly rounder shape and back, using `--easing-standard` and `--duration-short2` (~100 ms). The resting shape is size-dependent (see Sizes). Wrap in `@media (prefers-reduced-motion: no-preference)`.

---

## Sizes (M3 Expressive)

| Size | Height | Min-width | Icon size | Corner radius (resting) |
|---|---|---|---|---|
| `xs` | 32 dp | — | 16 dp | `--corner-full` (pill) |
| `sm` | 36 dp | — | 18 dp | `--corner-full` |
| `md` (default) | 40 dp | 48 dp | 18 dp | `--corner-full` |
| `lg` | 48 dp | — | 20 dp | `--corner-extra-large` |
| `xl` | 56 dp | — | 24 dp | `--corner-extra-large` |

For non-Expressive deployments only `md` is required. Size modifiers: `.button--size-xs`, `.button--size-sm`, `.button--size-lg`, `.button--size-xl` (no modifier = `md`).

---

## Design tokens

All tokens below are **unprefixed system role names** resolved to CSS custom properties. Components declare private local aliases inside their BEM block; see CSS usage example.

### Color
- Container (filled): `--primary`
- Label (filled): `--on-primary`
- Container (filled-tonal): `--secondary-container`
- Label (filled-tonal): `--on-secondary-container`
- Container (elevated): `--surface-container-low`
- Label (elevated / outlined / text): `--primary`
- Border (outlined): `--outline`
- Disabled container: `color-mix(in srgb, var(--on-surface) 12%, transparent)`
- Disabled label: `color-mix(in srgb, var(--on-surface) 38%, transparent)`

### Shape
- xs / sm / md: `--corner-full`
- lg / xl: `--corner-extra-large`
- Shape-morph target (pressed): `--corner-full` (always round — the morph is a subtle bounce at xs/sm/md and a larger jump at lg/xl from `--corner-extra-large` toward `--corner-full`)

### Typography
- `--label-large-font`, `--label-large-size`, `--label-large-weight`, `--label-large-line-height`

### Elevation
- `elevated` resting: `--elevation-1`
- `elevated` hover: `--elevation-2`
- `elevated` disabled: `--elevation-0`
- All other variants: `--elevation-0`

### Motion
- Shape morph easing: `--easing-standard`
- Shape morph duration: `--duration-short2` (~100 ms)

### State-layer opacities
- Hover: `--state-hover` (0.08)
- Focus: `--state-focus` (0.10)
- Pressed: `--state-pressed` (0.10)
- Dragged: `--state-dragged` (0.16)

### CSS private-variable pattern

```css
@layer kafui {
  .button {
    /* Local aliases — component-internal knobs pointing at system roles */
    --_bg:      var(--primary);
    --_fg:      var(--on-primary);
    --_radius:  var(--corner-full);
    --_h:       40px;
    --_icon-sz: 18px;
    --_gap:     8px;
    --_px:      24px;

    display: inline-flex;
    align-items: center;
    gap: var(--_gap);
    padding-inline: var(--_px);
    block-size: var(--_h);
    min-inline-size: 48px; /* md default */
    border-radius: var(--_radius);
    background: var(--_bg);
    color: var(--_fg);
    font: var(--label-large-weight) var(--label-large-size) / var(--label-large-line-height) var(--label-large-font);
    position: relative;
    overflow: hidden;
    border: none;
    cursor: pointer;
    /* Touch target via pseudo-element — visual size unchanged */
  }
  .button::before {
    content: '';
    position: absolute;
    inset: min(0px, calc((48px - 100%) / 2));
    /* Expands hit area to 48×48 dp minimum without affecting layout */
  }
  /* State layer */
  .button__state-layer {
    content: '';
    position: absolute;
    inset: 0;
    border-radius: inherit;
    background: currentColor;
    opacity: 0;
    pointer-events: none;
    transition: opacity 150ms var(--easing-standard);
  }
  .button[data-hovered] .button__state-layer  { opacity: var(--state-hover); }
  .button[data-focused] .button__state-layer  { opacity: var(--state-focus); }
  .button[data-pressed] .button__state-layer  { opacity: var(--state-pressed); }
  /* Variants */
  .button--filled-tonal  { --_bg: var(--secondary-container); --_fg: var(--on-secondary-container); }
  .button--elevated      { --_bg: var(--surface-container-low); box-shadow: var(--elevation-1); }
  .button--outlined      { --_bg: transparent; border: 1px solid var(--outline); }
  .button--text          { --_bg: transparent; --_px: 12px; }
  /* Outlined / text share primary label color via default --_fg = --primary reset */
  .button--outlined, .button--text, .button--elevated { --_fg: var(--primary); }
  /* Disabled */
  .button[data-disabled] {
    --_bg: color-mix(in srgb, var(--on-surface) 12%, transparent);
    --_fg: color-mix(in srgb, var(--on-surface) 38%, transparent);
    box-shadow: none;
    pointer-events: none;
  }
  /* Expressive shape morph */
  @media (prefers-reduced-motion: no-preference) {
    .button { transition: border-radius var(--duration-short2) var(--easing-standard); }
    .button[data-pressed] { border-radius: var(--corner-full); }
  }
  /* Expressive sizes */
  .button--size-xs { --_h: 32px; --_icon-sz: 16px; }
  .button--size-sm { --_h: 36px; --_icon-sz: 18px; }
  .button--size-lg { --_h: 48px; --_icon-sz: 20px; --_radius: var(--corner-extra-large); }
  .button--size-xl { --_h: 56px; --_icon-sz: 24px; --_radius: var(--corner-extra-large); }
  /* Trailing icon — flip via CSS order, no JS */
  .button--trailing-icon .button__icon { order: 1; }
  /* Icon sizing */
  .button__icon { inline-size: var(--_icon-sz); block-size: var(--_icon-sz); }
}
```

---

## Interaction & accessibility

**Keyboard:** `Space` and `Enter` activate. Native `<button>` handles this natively via the browser.

**Focus:** visible focus ring using CSS `outline` offset 3 dp from the container edge, color `--outline`. Uses `:focus-visible` only — no focus ring on pointer interaction.

**ARIA:**
- Root is native `<button>` — `role="button"` is implicit.
- `aria-disabled="true"` (not `disabled` HTML attr) keeps the element in the tab order and communicates state to AT. React Aria applies this automatically via `isDisabled`.
- When icon-only (no label text): `aria-label` is required; icon is `aria-hidden="true"`.
- When label is present: icon is `aria-hidden="true"`.

**Touch target:** minimum 48×48 dp achieved via `::before` pseudo-element expand trick — visual height may be less (e.g. 32 dp for `xs`).

**Reduced motion:** Expressive shape morph is gated on `@media (prefers-reduced-motion: no-preference)`.

**RTL:** icon order is controlled by CSS logical properties and flex `order`; no JS. `dir="rtl"` inheritance is automatic.

---

## Proposed kafUI React API

```tsx
// react-aria-components base: <Button>
import { Button as RACButton, type PressEvent } from 'react-aria-components';

type ButtonVariant = 'filled' | 'filled-tonal' | 'elevated' | 'outlined' | 'text';
type ButtonSize    = 'xs' | 'sm' | 'md' | 'lg' | 'xl';

interface ButtonProps {
  /** M3 visual variant. Default: 'filled'. */
  variant?: ButtonVariant;
  /** M3 Expressive size. Default: 'md'. */
  size?: ButtonSize;
  /** Icon sprite name. Renders <Icon> before the label (leading). */
  icon?: string;
  /** When true, renders icon after the label (trailing). Default: false. */
  trailingIcon?: boolean;
  /** React Aria disabled — uses aria-disabled, keeps element focusable. */
  isDisabled?: boolean;
  /** React Aria press handler — do NOT use onClick. */
  onPress?: (e: PressEvent) => void;
  /** Label text. Omit for icon-only; then aria-label is required. */
  children?: React.ReactNode;
  /** Extra classes merged after BEM classes. */
  className?: string;
  // All remaining RAC <Button> props are spread through.
}
```

**Design rationale — why this beats MUI:**

1. **No `color` prop.** MUI's `color="secondary"` conflates color role with variant, producing non-M3 combinations. `variant` alone encodes both shape and color role per spec.
2. **`icon` + `trailingIcon` over `startIcon`/`endIcon`.** Two props instead of two separate ReactNode slots. Trailing is a boolean flip — order is a CSS concern (`order: 1`), not a structural one. Consumers don't wrap icons in JSX; they pass a sprite name string.
3. **`children` optional, not required.** Enables icon-only mode without a separate component. The modifier `.button--icon-only` is applied automatically.
4. **No `disableElevation` / `disableRipple`.** These are not M3 concepts and indicate underlying framework leakage in MUI's API. Elevation and ripple are baked into the variant token mapping.
5. **`size` is Expressive-native.** MUI's `small/medium/large` don't match M3 dp values. Ours map 1:1 to M3 spec height/padding/icon-size.

**BEM classes emitted:**
- `.button` (always)
- `.button--{variant}` e.g. `.button--filled-tonal`
- `.button--size-{size}` e.g. `.button--size-lg` (no modifier = `md`)
- `.button--icon-only` when no label children
- `.button--trailing-icon` when `trailingIcon={true}`
- RAC data attributes (`data-hovered`, `data-focused`, `data-pressed`, `data-disabled`) drive state-layer CSS — no className toggling in JS

**React Aria base:** `Button` from `react-aria-components`. Provides keyboard/pointer/focus-visible handling and `aria-disabled` automatically.

> **Layer safety:** all styles live in `@layer kafui { … }`. The bare `.button` class name is safe — unlayered author styles will always win if consumers need to override.
