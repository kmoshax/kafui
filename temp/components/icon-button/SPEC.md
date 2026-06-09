# Icon Button

Icon buttons are compact, icon-only interactive elements for common, recognizable actions. They support a **toggle** (selected/unselected) capability. M3 category: **Actions → Icon Button**. M3 Expressive adds explicit sizes (XS–XL) and width modes (narrow, default, wide).

---

## Anatomy / parts

| BEM element | Description |
|---|---|
| `.icon-button` | Root `<button>` element; the interactive container |
| `.icon-button__state-layer` | Hover/focus/pressed tint overlay — absolutely positioned, `border-radius: inherit` |
| `.icon-button__icon` | The icon — `<Icon>` sprite wrapper; always `aria-hidden="true"` |

No visible text label is rendered. Accessible name is always via `aria-label`.

---

## Variants

| Variant | Container (unselected) | Icon (unselected) | Container (selected) | Icon (selected) |
|---|---|---|---|---|
| `standard` (default) | transparent | `on-surface-variant` | transparent | `primary` |
| `filled` | `surface-variant` | `primary` | `primary` | `on-primary` |
| `filled-tonal` | `surface-variant` | `on-surface-variant` | `secondary-container` | `on-secondary-container` |
| `outlined` | transparent + `outline` border | `on-surface-variant` | `inverse-surface` | `inverse-on-surface` (no border) |

**Toggle state detail:**

- `standard`: only the icon color changes (`on-surface-variant` → `primary`). No container or border change.
- `filled`: the visual weight inverts. Unselected = lighter (`surface-variant` background). Selected = heavier (`primary` background). This is intentional M3 behavior — selected state is the "activated" state with higher visual weight.
- `filled-tonal`: unselected = neutral (`surface-variant` + `on-surface-variant`). Selected = brand-toned (`secondary-container` + `on-secondary-container`).
- `outlined`: selected fills with `inverse-surface` and drops the `outline` border entirely.

> Non-toggle icon buttons (`<IconButton>`) have no selected state. Only `<ToggleIconButton>` uses selected/unselected color pairs. The `standard` variant's unselected colors are the non-toggle `standard` icon button's normal state.

---

## States

| State | State-layer opacity |
|---|---|
| Enabled | 0% |
| Hovered | `--state-hover` (0.08) |
| Focused | `--state-focus` (0.10) |
| Pressed | `--state-pressed` (0.10) |
| Disabled | 0%; container `on-surface` 12%; icon `on-surface` 38% |

State-layer color = current icon color (variant + selected state determine this).

---

## Sizes (M3 Expressive)

| Size | Container | Icon size | Corner radius |
|---|---|---|---|
| `xs` | 28×28 dp | 16 dp | `--corner-full` |
| `sm` | 32×32 dp | 18 dp | `--corner-full` |
| `md` (default) | 40×40 dp | 24 dp | `--corner-full` |
| `lg` | 48×48 dp | 24 dp | `--corner-extra-large` |
| `xl` | 56×56 dp | 28 dp | `--corner-extra-large` |

Non-Expressive: only `md` is required. Size modifiers: `.icon-button--size-xs`, `.icon-button--size-sm`, `.icon-button--size-lg`, `.icon-button--size-xl` (no modifier = `md`).

## Width modes (M3 Expressive)

Applied as a modifier on top of size. Changes the container from square to a rectangular pill. The icon remains centered.

| Width mode | Approximate shape at `md` | Use |
|---|---|---|
| `narrow` | 28×40 dp (taller than wide) | Compact vertical layouts |
| `default` | 40×40 dp (square) | Standard |
| `wide` | 56×40 dp (wider than tall) | Toolbar emphasis |

BEM modifiers: `.icon-button--narrow`, `.icon-button--wide` (no modifier = `default`).

Width mode transitions (optional, Expressive): width animates between narrow/default/wide on mode change, using `--easing-standard` and `--duration-short3`.

---

## Design tokens

All tokens are unprefixed system role names.

### Color — non-toggle (`<IconButton>`)
- `standard`: icon = `--on-surface-variant`
- `filled`: container = `--surface-variant`; icon = `--primary`
- `filled-tonal`: container = `--surface-variant`; icon = `--on-surface-variant`
- `outlined`: border = `--outline`; icon = `--on-surface-variant`
- Disabled container: `color-mix(in srgb, var(--on-surface) 12%, transparent)`
- Disabled icon: `color-mix(in srgb, var(--on-surface) 38%, transparent)`

### Color — toggle selected state (`<ToggleIconButton>` with `data-selected`)
- `standard` selected: icon = `--primary` (container stays transparent)
- `filled` selected: container = `--primary`; icon = `--on-primary`
- `filled-tonal` selected: container = `--secondary-container`; icon = `--on-secondary-container`
- `outlined` selected: container = `--inverse-surface`; icon = `--inverse-on-surface`; border: none

### Shape
- `xs` / `sm` / `md`: `--corner-full`
- `lg` / `xl`: `--corner-extra-large`

### Elevation
- All variants: `--elevation-0` (icon buttons never elevate)

### Motion
- Width mode transition: `--easing-standard`, `--duration-short3`
- Reduced motion: `transition: none`

### State-layer opacities
- Hover: `--state-hover` (0.08); Focus/pressed: `--state-focus` / `--state-pressed` (0.10)

### CSS private-variable pattern

```css
@layer kafui {
  .icon-button {
    /* Local aliases — defaults = standard non-toggle */
    --_bg:      transparent;
    --_fg:      var(--on-surface-variant);
    --_border:  none;
    --_radius:  var(--corner-full);
    --_sz:      40px;
    --_icon-sz: 24px;

    display: inline-flex;
    align-items: center;
    justify-content: center;
    inline-size:  var(--_sz);
    block-size:   var(--_sz);
    border-radius: var(--_radius);
    background: var(--_bg);
    color: var(--_fg);
    border: var(--_border);
    cursor: pointer;
    position: relative;
    overflow: hidden;
    padding: 0;
  }

  /* Touch target extension for xs (28dp) and sm (32dp) */
  .icon-button--size-xs::before,
  .icon-button--size-sm::before {
    content: '';
    position: absolute;
    /* Center a 48dp hit area over a smaller visual container */
    inset: min(0px, calc((48px - 100%) / 2));
  }

  /* State layer */
  .icon-button__state-layer {
    position: absolute;
    inset: 0;
    border-radius: inherit;
    background: currentColor;
    opacity: 0;
    pointer-events: none;
    transition: opacity 150ms var(--easing-standard);
  }
  .icon-button[data-hovered] .icon-button__state-layer { opacity: var(--state-hover); }
  .icon-button[data-focused] .icon-button__state-layer { opacity: var(--state-focus); }
  .icon-button[data-pressed] .icon-button__state-layer { opacity: var(--state-pressed); }

  /* Icon sizing */
  .icon-button__icon { inline-size: var(--_icon-sz); block-size: var(--_icon-sz); }

  /* Filled variant — unselected (default for non-toggle filled) */
  .icon-button--filled {
    --_bg: var(--surface-variant);
    --_fg: var(--primary);
  }

  /* Filled-tonal variant — unselected */
  .icon-button--filled-tonal {
    --_bg: var(--surface-variant);
    --_fg: var(--on-surface-variant);
  }

  /* Outlined variant — unselected */
  .icon-button--outlined {
    --_border: 1px solid var(--outline);
    --_fg: var(--on-surface-variant);
  }

  /* Toggle selected states */
  .icon-button--standard[data-selected]      { --_fg: var(--primary); }
  .icon-button--filled[data-selected]        { --_bg: var(--primary); --_fg: var(--on-primary); }
  .icon-button--filled-tonal[data-selected]  { --_bg: var(--secondary-container); --_fg: var(--on-secondary-container); }
  .icon-button--outlined[data-selected]      { --_bg: var(--inverse-surface); --_fg: var(--inverse-on-surface); --_border: none; }

  /* Expressive sizes */
  .icon-button--size-xs { --_sz: 28px; --_icon-sz: 16px; }
  .icon-button--size-sm { --_sz: 32px; --_icon-sz: 18px; }
  .icon-button--size-lg { --_sz: 48px; --_radius: var(--corner-extra-large); }
  .icon-button--size-xl { --_sz: 56px; --_icon-sz: 28px; --_radius: var(--corner-extra-large); }

  /* Expressive width modes (override inline-size while keeping block-size from --_sz) */
  .icon-button--narrow { inline-size: calc(var(--_sz) * 0.7); }
  .icon-button--wide   { inline-size: calc(var(--_sz) * 1.4); }

  /* Width mode transition (Expressive, opt-in) */
  @media (prefers-reduced-motion: no-preference) {
    .icon-button--narrow,
    .icon-button--wide {
      transition: inline-size var(--duration-short3) var(--easing-standard);
    }
  }

  /* Disabled */
  .icon-button[data-disabled] {
    --_bg: color-mix(in srgb, var(--on-surface) 12%, transparent);
    --_fg: color-mix(in srgb, var(--on-surface) 38%, transparent);
    --_border: none;
    pointer-events: none;
  }
  /* standard disabled — no container, just tinted icon */
  .icon-button--standard[data-disabled] { --_bg: transparent; }
}
```

---

## Interaction & accessibility

**Keyboard:** `Space` and `Enter` activate. For toggle buttons, each activation flips selected state.

**Focus:** focus ring at 3 dp offset, color `--outline`, via CSS `outline` on `:focus-visible` only.

**ARIA:**
- Non-toggle `<IconButton>`: `<button>` with `aria-label` required. No `aria-pressed`.
- Toggle `<ToggleIconButton>`: `<button aria-pressed="true|false">`. React Aria `ToggleButton` sets `aria-pressed` automatically — do NOT set it manually.
- Icon is always `aria-hidden="true"`.
- `aria-label` describes either the **action** ("Favorite") or the **state** ("Favorited") depending on context. Document both patterns; leave the choice to the consumer. M3 guidance: prefer the action name ("Favorite") and let `aria-pressed` communicate state, rather than toggling the label text.

**Touch target:** minimum 48×48 dp. `xs` (28 dp) and `sm` (32 dp) sizes use a `::before` pseudo-element to extend the hit area without affecting layout.

**Reduced motion:** Expressive width-mode transitions are gated on `@media (prefers-reduced-motion: no-preference)`.

**RTL:** directional icons (back/forward arrows, chevrons) must use directional icon names or the `<Icon>` component's `:dir(rtl)` flip. The icon button itself needs no RTL-specific CSS — layout is symmetric.

---

## Proposed kafUI React API

```tsx
// react-aria-components bases:
//   <IconButton>        → RAC <Button>
//   <ToggleIconButton>  → RAC <ToggleButton>

import { Button as RACButton, ToggleButton as RACToggleButton, type PressEvent } from 'react-aria-components';

type IconButtonVariant = 'standard' | 'filled' | 'filled-tonal' | 'outlined';
type IconButtonSize    = 'xs' | 'sm' | 'md' | 'lg' | 'xl';
type IconButtonWidth   = 'narrow' | 'default' | 'wide';

/** Non-toggle icon button */
interface IconButtonProps {
  /** Icon sprite name. Required. */
  icon: string;
  /** M3 visual variant. Default: 'standard'. */
  variant?: IconButtonVariant;
  /** M3 Expressive size. Default: 'md'. */
  size?: IconButtonSize;
  /** M3 Expressive width mode. Default: 'default'. */
  width?: IconButtonWidth;
  isDisabled?: boolean;
  onPress?: (e: PressEvent) => void;
  /** Required — no visible text label. */
  'aria-label': string;
  className?: string;
  // All remaining RAC <Button> props spread through.
}

/** Toggle icon button — separate component, not a boolean flag on IconButton */
interface ToggleIconButtonProps {
  /** Icon sprite name for unselected state. Required. */
  icon: string;
  /**
   * Optional different icon sprite name when selected.
   * E.g. icon="favorite_outline" selectedIcon="favorite_filled"
   */
  selectedIcon?: string;
  variant?: IconButtonVariant;
  size?: IconButtonSize;
  width?: IconButtonWidth;
  /** Controlled: current selected state. */
  isSelected?: boolean;
  /** Uncontrolled: initial selected state. */
  defaultSelected?: boolean;
  onChange?: (isSelected: boolean) => void;
  isDisabled?: boolean;
  /** Required — no visible text label. */
  'aria-label': string;
  className?: string;
  // All remaining RAC <ToggleButton> props spread through.
}
```

**Design rationale — two separate components, not a `toggle` boolean:**

`<IconButton>` and `<ToggleIconButton>` are distinct exports backed by different RAC primitives (`Button` vs `ToggleButton`). This is an intentional API decision:

1. **Type safety:** toggle needs `isSelected`/`onChange`; non-toggle needs `onPress`. Merging them on one interface requires all toggle props to be optional, which allows consumers to create a toggle button without `onChange` — a common React mistake. Separate interfaces enforce the right contract.
2. **RAC primitive:** `ToggleButton` automatically manages `aria-pressed`. `Button` does not. Switching RAC primitive on a boolean prop would require conditional rendering of different RAC components, which breaks hook rules if done naively.
3. **Tree-shaking:** if you never use toggle icon buttons, `<ToggleButton>` RAC code is never in your bundle.

**`selectedIcon` first-class feature:** when `isSelected` is true and `selectedIcon` is provided, the component renders `selectedIcon` instead of `icon`. This is the standard M3 "filled icon on select" pattern (e.g. `bookmark` → `bookmark_filled`). Without this prop, consumers must manage the icon name in external state — more code, more risk of state drift.

**BEM classes emitted:**
- `.icon-button` (always)
- `.icon-button--{variant}` e.g. `.icon-button--filled-tonal`
- `.icon-button--size-{size}` e.g. `.icon-button--size-xs` (no modifier = `md`)
- `.icon-button--narrow` or `.icon-button--wide` (no modifier = `default`)
- RAC `data-selected` attribute drives selected-state CSS — no className toggling in JS

**React Aria base:**
- `<IconButton>`: `Button` from `react-aria-components`
- `<ToggleIconButton>`: `ToggleButton` from `react-aria-components` — provides `aria-pressed`, `isSelected`, `onChange`, and `defaultSelected` automatically

> **Layer safety:** all styles in `@layer kafui { … }`. The bare `.icon-button` class is safe.

> **Cross-component note for lead:** `surface-variant` is used as the unselected container for `filled` and `filled-tonal` icon buttons. Verify `--surface-variant` is defined in `_TOKENS.md` — it is listed in `md.sys.color` but not currently in the _TOKENS.md unprefixed list. Raise for confirmation.
