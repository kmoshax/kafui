# FAB (Floating Action Button)

A FAB represents the primary or most-common action on a screen, floating above content at a fixed position. M3 category: **Actions → FAB**.

---

## Anatomy / parts

| BEM element | Description |
|---|---|
| `.fab` | Root `<button>` element; the visible container |
| `.fab__state-layer` | Hover/focus/pressed tint overlay — absolutely positioned, `border-radius: inherit` |
| `.fab__icon` | Required icon — `<Icon>` sprite wrapper; always `aria-hidden="true"` |

FABs always contain an icon and no visible text label. Accessible name is always via `aria-label`.

---

## Variants (color)

M3 defines four color variants. The prop is called `color` (not `variant`) because the structural anatomy is identical across all four — only the color roles differ.

| `color` | Container | Icon color | State-layer color |
|---|---|---|---|
| `surface` (default) | `surface-container-high` | `primary` | `primary` |
| `primary` | `primary-container` | `on-primary-container` | `on-primary-container` |
| `secondary` | `secondary-container` | `on-secondary-container` | `on-secondary-container` |
| `tertiary` | `tertiary-container` | `on-tertiary-container` | `on-tertiary-container` |

### Lowered variant

The **lowered** FAB rests at elevation 1 (instead of 3) and hovers at elevation 2 (instead of 4). It signals reduced visual prominence without removing the FAB from the hierarchy. Activated via a `lowered` boolean prop. BEM modifier: `.fab--lowered`.

---

## Sizes

| Size | Container | Icon size | Corner radius | Touch target note |
|---|---|---|---|---|
| `small` | 40×40 dp | 24 dp | `--corner-medium` (12 dp) | Needs `::before` pad to 48 dp |
| `medium` (default) | 56×56 dp | 24 dp | `--corner-large` (16 dp) | Meets 48 dp minimum |
| `large` | 96×96 dp | 36 dp | `--corner-extra-large` (28 dp) | Far exceeds minimum |

BEM modifiers: `.fab--small`, `.fab--large` (no modifier = `medium`).

---

## States

| State | State-layer opacity | Elevation (normal / lowered) |
|---|---|---|
| Enabled | 0% | 3 / 1 |
| Hovered | `--state-hover` (0.08) | 4 / 2 |
| Focused | `--state-focus` (0.10) | 3 / 1 |
| Pressed | `--state-pressed` (0.10) | 3 / 1 |
| Disabled | — | 0; container `on-surface` 12%; icon `on-surface` 38% |

> Note: M3 spec does not define a "disabled" FAB as a common case — the guidance is to hide or replace a FAB rather than disable it. This is documented but disabled support is provided for completeness.

---

## Design tokens

All tokens are unprefixed system role names resolved as CSS custom properties.

### Color
- `surface` container: `--surface-container-high`; icon: `--primary`
- `primary` container: `--primary-container`; icon: `--on-primary-container`
- `secondary` container: `--secondary-container`; icon: `--on-secondary-container`
- `tertiary` container: `--tertiary-container`; icon: `--on-tertiary-container`
- Disabled container: `color-mix(in srgb, var(--on-surface) 12%, transparent)`
- Disabled icon: `color-mix(in srgb, var(--on-surface) 38%, transparent)`

### Shape
- Small: `--corner-medium` (12 dp)
- Medium: `--corner-large` (16 dp)
- Large: `--corner-extra-large` (28 dp)

### Elevation
- Level 1: `--elevation-1`
- Level 2: `--elevation-2`
- Level 3: `--elevation-3`
- Level 4: `--elevation-4`

### Motion
- Scroll hide/show (optional, app-level): `--easing-emphasized`, `--duration-medium2`
- Reduced motion: skip enter/exit animation entirely (`transition: none`)

### State-layer opacities
- Hover: `--state-hover` (0.08)
- Focus/pressed: `--state-focus` / `--state-pressed` (0.10)

### CSS private-variable pattern

```css
@layer kafui {
  .fab {
    /* Local aliases */
    --_bg:      var(--surface-container-high);
    --_fg:      var(--primary);
    --_radius:  var(--corner-large);
    --_sz:      56px;
    --_icon-sz: 24px;

    display: inline-flex;
    align-items: center;
    justify-content: center;
    inline-size:  var(--_sz);
    block-size:   var(--_sz);
    border-radius: var(--_radius);
    background: var(--_bg);
    color: var(--_fg);
    box-shadow: var(--elevation-3);
    border: none;
    cursor: pointer;
    position: relative;
    overflow: hidden;
  }

  /* Touch target extension for small FAB */
  .fab--small::before {
    content: '';
    position: absolute;
    inset: -4px; /* (48 - 40) / 2 = 4 dp */
  }

  /* State layer */
  .fab__state-layer {
    position: absolute;
    inset: 0;
    border-radius: inherit;
    background: currentColor;
    opacity: 0;
    pointer-events: none;
    transition: opacity 150ms var(--easing-standard);
  }
  .fab[data-hovered] .fab__state-layer  { opacity: var(--state-hover); }
  .fab[data-focused] .fab__state-layer  { opacity: var(--state-focus); }
  .fab[data-pressed] .fab__state-layer  { opacity: var(--state-pressed); }

  /* Hover elevation */
  .fab[data-hovered]      { box-shadow: var(--elevation-4); }
  .fab--lowered           { box-shadow: var(--elevation-1); }
  .fab--lowered[data-hovered] { box-shadow: var(--elevation-2); }

  /* Color variants */
  .fab--primary   { --_bg: var(--primary-container);   --_fg: var(--on-primary-container); }
  .fab--secondary { --_bg: var(--secondary-container); --_fg: var(--on-secondary-container); }
  .fab--tertiary  { --_bg: var(--tertiary-container);  --_fg: var(--on-tertiary-container); }

  /* Size variants */
  .fab--small { --_sz: 40px; --_radius: var(--corner-medium); }
  .fab--large { --_sz: 96px; --_icon-sz: 36px; --_radius: var(--corner-extra-large); }

  /* Icon */
  .fab__icon { inline-size: var(--_icon-sz); block-size: var(--_icon-sz); }

  /* Disabled */
  .fab[data-disabled] {
    --_bg: color-mix(in srgb, var(--on-surface) 12%, transparent);
    --_fg: color-mix(in srgb, var(--on-surface) 38%, transparent);
    box-shadow: none;
    pointer-events: none;
  }

  /* Scroll-hide utility — applied externally by consumer/hook */
  .fab--hidden {
    transform: scale(0);
    opacity: 0;
  }
  @media (prefers-reduced-motion: no-preference) {
    .fab {
      transition:
        transform var(--duration-medium2) var(--easing-emphasized),
        opacity   var(--duration-medium2) var(--easing-emphasized),
        box-shadow 150ms var(--easing-standard);
    }
  }
}
```

---

## Interaction & accessibility

**Keyboard:** `Space` and `Enter` activate. Native `<button>` handles this.

**Focus:** focus ring at 3 dp offset, color `--outline`, via `outline` CSS property on `:focus-visible` only.

**ARIA:**
- Root is `<button>` — `role="button"` implicit.
- `aria-label` is **required** in the TypeScript type (not optional). FABs are icon-only — omitting `aria-label` is always an a11y bug. A compile-time type error is better than a runtime audit finding.
- `aria-disabled="true"` (not HTML `disabled`) preserves tab order.
- If FAB visibility changes on scroll, announce to `aria-live` region at the app level (out of scope for the component).

**Touch target:** small FAB (40 dp) needs the `::before` pseudo-element to reach 48 dp. Medium (56 dp) and large (96 dp) already exceed the minimum.

**Reduced motion:** scroll-hide/show animation is disabled under `prefers-reduced-motion: reduce` (transition collapses to instant).

**RTL:** FABs contain only an icon — no directional layout concern. Directional icons (e.g. back/forward arrows) should use the `<Icon>` component's `:dir(rtl)` flip mechanism, not component-level CSS.

**Scroll behavior (optional):** FAB may hide on scroll-down and show on scroll-up. This is a consumer concern — the component documents a `.fab--hidden` CSS class contract (`transform: scale(0)` + `opacity: 0` transition) so integration is consistent. A `useFabScrollHide()` utility hook will be provided in `@kafui/hooks`.

---

## Proposed kafUI React API

```tsx
// react-aria-components base: <Button>
import { Button as RACButton, type PressEvent } from 'react-aria-components';

type FabColor = 'surface' | 'primary' | 'secondary' | 'tertiary';
type FabSize  = 'small' | 'medium' | 'large';

interface FabProps {
  /** Icon sprite name. FABs are always icon-only. Required. */
  icon: string;
  /** M3 color variant. Default: 'surface'. */
  color?: FabColor;
  /** Size. Default: 'medium'. */
  size?: FabSize;
  /** Resting elevation 1 instead of 3. Default: false. */
  lowered?: boolean;
  /** React Aria disabled — aria-disabled, not HTML disabled. */
  isDisabled?: boolean;
  /** React Aria press handler. */
  onPress?: (e: PressEvent) => void;
  /** Required. FABs have no visible text — accessible name must be explicit. */
  'aria-label': string;
  className?: string;
  // All remaining RAC <Button> props spread through.
}
```

**BEM classes emitted:**
- `.fab` (always)
- `.fab--{color}` e.g. `.fab--primary` (no modifier when `color="surface"` — that is the base style)
- `.fab--small` or `.fab--large` (no modifier = `medium`)
- `.fab--lowered` when `lowered={true}`
- `.fab--hidden` — a CSS utility class consumers add externally for scroll-hide; not managed by this component

**React Aria base:** `Button` from `react-aria-components`.

**Design rationale — why `color` not `variant`:**
M3 uses the word "variant" for FABs in its docs but the four options differ only in color roles — anatomy and dimensions are identical. Using `color` instead of `variant` aligns with how the M3 spec actually describes the choices ("color scheme variant") and avoids confusion with the Button `variant` prop which also controls structure. This distinction is consistent across FAB and ExtendedFab.

> **Layer safety:** all styles live in `@layer kafui { … }`. The bare `.fab` class is safe — unlayered author styles win automatically.
