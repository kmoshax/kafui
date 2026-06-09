# Extended FAB

An Extended FAB is a wider FAB variant that always shows a text label and optionally a leading icon. It communicates the primary action more explicitly and is suited to larger screens or when labeling the action adds critical clarity. M3 category: **Actions → FAB → Extended FAB**.

---

## Anatomy / parts

| BEM element | Description |
|---|---|
| `.extended-fab` | Root `<button>` element; the visible container |
| `.extended-fab__state-layer` | Hover/focus/pressed tint overlay — absolutely positioned, `border-radius: inherit` |
| `.extended-fab__icon` | Optional leading icon — `<Icon>` sprite wrapper; always `aria-hidden="true"` |
| `.extended-fab__label` | Required text label — renders `label-large` typescale role; always visible in expanded state |

When no icon is present, the label is horizontally centered. When an icon is present, icon + label are row-flexed with a 12 dp gap; icon is at `inline-start`.

---

## Variants (color)

Same four color variants as FAB. Prop is `color` for the same reasons (see FAB spec — only color roles differ, not anatomy).

| `color` | Container | Icon + label color | State-layer color |
|---|---|---|---|
| `surface` (default) | `surface-container-high` | `primary` | `primary` |
| `primary` | `primary-container` | `on-primary-container` | `on-primary-container` |
| `secondary` | `secondary-container` | `on-secondary-container` | `on-secondary-container` |
| `tertiary` | `tertiary-container` | `on-tertiary-container` | `on-tertiary-container` |

### Lowered variant

`lowered` boolean: resting elevation 1 (not 3), hover elevation 2 (not 4). BEM modifier: `.extended-fab--lowered`.

---

## Collapse / Extend behavior

The Extended FAB may **collapse** (icon-only, like a regular FAB) when the user scrolls down, and **extend** (icon + label) when scrolling up or idle. The component supports two visual states:

- **Extended** (default): full width with label visible. `min-inline-size: 80 dp`.
- **Collapsed**: icon-only, dimensions = 56×56 dp, corner radius = `--corner-large` (same as regular medium FAB).

Transition between states:
- Container width: `--easing-emphasized`, `--duration-medium2` (~300 ms).
- Label opacity: fades to 0 simultaneously; uses `--duration-short2` (~100 ms) with a slight delay (label starts fading immediately but width finishes later — staggered for perceived smoothness).
- Under `prefers-reduced-motion: reduce`: instant transition — `transition: none`.

The `collapsed` prop is **controlled** — the component does not own scroll state. Consumers drive it from a scroll hook. A `useExtendedFabCollapse(threshold?)` utility hook is provided in `@kafui/hooks`.

BEM modifier: `.extended-fab--collapsed`.

> **Important:** when `collapsed={true}` and no `icon` was provided, collapse is a no-op — an icon-less Extended FAB cannot collapse into a meaningful icon-only state. The component should log a dev-mode warning and remain extended.

---

## Sizing

There is exactly **one size** for Extended FAB. No `size` prop.

| Dimension | Value |
|---|---|
| Height | 56 dp |
| Min-width (extended) | 80 dp |
| Padding-inline (with icon) | 16 dp start, 20 dp end |
| Padding-inline (no icon) | 20 dp both sides |
| Icon size | 24 dp |
| Icon-to-label gap | 12 dp |
| Corner radius (extended) | `--corner-large` (16 dp) |
| Corner radius (collapsed) | `--corner-large` (16 dp — same, no change) |

---

## States

| State | State-layer opacity | Elevation (normal / lowered) |
|---|---|---|
| Enabled | 0% | 3 / 1 |
| Hovered | `--state-hover` (0.08) | 4 / 2 |
| Focused | `--state-focus` (0.10) | 3 / 1 |
| Pressed | `--state-pressed` (0.10) | 3 / 1 |
| Disabled | — | 0; container `on-surface` 12%; content `on-surface` 38% |

---

## Design tokens

All tokens are unprefixed system role names resolved as CSS custom properties.

### Color
- Same mapping as FAB (four variants × container + content). See FAB spec.
- Disabled container: `color-mix(in srgb, var(--on-surface) 12%, transparent)`
- Disabled content: `color-mix(in srgb, var(--on-surface) 38%, transparent)`

### Shape
- Extended and collapsed: `--corner-large` (16 dp) — no shape change on collapse

### Typography
- `--label-large-font`, `--label-large-size`, `--label-large-weight`, `--label-large-line-height`

### Elevation
- `--elevation-1` through `--elevation-4` (same as FAB)

### Motion
- Collapse/extend: `--easing-emphasized`, `--duration-medium2` (~300 ms)
- Label fade: `--duration-short2` (~100 ms)
- Reduced motion: all transitions instant

### State-layer opacities
- Hover: `--state-hover` (0.08); Focus/pressed: `--state-focus` / `--state-pressed` (0.10)

### CSS private-variable pattern

```css
@layer kafui {
  .extended-fab {
    /* Local aliases */
    --_bg:      var(--surface-container-high);
    --_fg:      var(--primary);
    --_radius:  var(--corner-large);
    --_h:       56px;
    --_ps:      16px; /* padding-inline-start (with icon) */
    --_pe:      20px; /* padding-inline-end */
    --_gap:     12px;
    --_icon-sz: 24px;

    display: inline-flex;
    align-items: center;
    gap: var(--_gap);
    block-size: var(--_h);
    min-inline-size: 80px;
    padding-inline: var(--_ps) var(--_pe);
    border-radius: var(--_radius);
    background: var(--_bg);
    color: var(--_fg);
    box-shadow: var(--elevation-3);
    border: none;
    cursor: pointer;
    position: relative;
    overflow: hidden;
    font: var(--label-large-weight) var(--label-large-size) / var(--label-large-line-height) var(--label-large-font);
  }

  /* No icon: center label, symmetric padding */
  .extended-fab:not(:has(.extended-fab__icon)) {
    --_ps: 20px;
    justify-content: center;
  }

  /* State layer */
  .extended-fab__state-layer {
    position: absolute;
    inset: 0;
    border-radius: inherit;
    background: currentColor;
    opacity: 0;
    pointer-events: none;
    transition: opacity 150ms var(--easing-standard);
  }
  .extended-fab[data-hovered] .extended-fab__state-layer { opacity: var(--state-hover); }
  .extended-fab[data-focused] .extended-fab__state-layer { opacity: var(--state-focus); }
  .extended-fab[data-pressed] .extended-fab__state-layer { opacity: var(--state-pressed); }

  /* Elevation */
  .extended-fab[data-hovered]           { box-shadow: var(--elevation-4); }
  .extended-fab--lowered                { box-shadow: var(--elevation-1); }
  .extended-fab--lowered[data-hovered]  { box-shadow: var(--elevation-2); }

  /* Color variants */
  .extended-fab--primary   { --_bg: var(--primary-container);   --_fg: var(--on-primary-container); }
  .extended-fab--secondary { --_bg: var(--secondary-container); --_fg: var(--on-secondary-container); }
  .extended-fab--tertiary  { --_bg: var(--tertiary-container);  --_fg: var(--on-tertiary-container); }

  /* Icon sizing */
  .extended-fab__icon { inline-size: var(--_icon-sz); block-size: var(--_icon-sz); flex-shrink: 0; }

  /* Label */
  .extended-fab__label { white-space: nowrap; overflow: hidden; }

  /* Disabled */
  .extended-fab[data-disabled] {
    --_bg: color-mix(in srgb, var(--on-surface) 12%, transparent);
    --_fg: color-mix(in srgb, var(--on-surface) 38%, transparent);
    box-shadow: none;
    pointer-events: none;
  }

  /* Collapsed state */
  .extended-fab--collapsed {
    inline-size: 56px;
    min-inline-size: unset;
    --_ps: 0;
    --_pe: 0;
    justify-content: center;
  }
  .extended-fab--collapsed .extended-fab__label {
    opacity: 0;
    inline-size: 0;
    overflow: hidden;
  }

  /* Collapse/extend animation */
  @media (prefers-reduced-motion: no-preference) {
    .extended-fab {
      transition:
        inline-size         var(--duration-medium2) var(--easing-emphasized),
        padding-inline      var(--duration-medium2) var(--easing-emphasized),
        box-shadow          150ms var(--easing-standard);
    }
    .extended-fab__label {
      transition:
        opacity    var(--duration-short2) var(--easing-standard),
        inline-size var(--duration-medium2) var(--easing-emphasized);
    }
  }
}
```

---

## Interaction & accessibility

**Keyboard:** `Space` and `Enter` activate. Collapsed state does not change keyboard behavior.

**Focus:** focus ring at 3 dp offset, color `--outline`, via CSS `outline` on `:focus-visible` only.

**ARIA:**
- Root is `<button>` — `role="button"` implicit.
- Always set `aria-label` on the root button equal to the `label` prop value. When extended, `aria-label` is redundant but harmless. When collapsed, it is the only accessible name — no visible text is visible to AT.
- The `label` is a required `string` prop (not `children`) specifically so the component can always derive a stable accessible name without parsing ReactNode children.
- Icon is always `aria-hidden="true"`.
- Collapsing while focused: if the label disappears while the button has focus, the accessible name persists via `aria-label` — no announcement gaps.

**Touch target:** height is 56 dp — exceeds the 48 dp minimum. No extra padding needed.

**Reduced motion:** width/label animation disabled; collapse/extend is instant.

**RTL:** icon is at `inline-start` via CSS logical properties and `flex-direction: row`. Label text renders in reading direction automatically.

---

## Proposed kafUI React API

```tsx
// react-aria-components base: <Button>
import { Button as RACButton, type PressEvent } from 'react-aria-components';

type ExtendedFabColor = 'surface' | 'primary' | 'secondary' | 'tertiary';

interface ExtendedFabProps {
  /** Required. Text label rendered visibly; also used as aria-label always. */
  label: string;
  /** Optional leading icon sprite name. */
  icon?: string;
  /** M3 color variant. Default: 'surface'. */
  color?: ExtendedFabColor;
  /** Resting elevation 1 instead of 3. Default: false. */
  lowered?: boolean;
  /** Controlled collapsed state — consumer drives from scroll hook. Default: false. */
  collapsed?: boolean;
  /** React Aria disabled — aria-disabled, not HTML disabled. */
  isDisabled?: boolean;
  /** React Aria press handler. */
  onPress?: (e: PressEvent) => void;
  className?: string;
  // All remaining RAC <Button> props spread through.
}
```

**Design rationale — `label: string` not `children: ReactNode`:**
The `label` prop is intentionally a `string`, not `children`. This is the primary API decision that differs from both MUI and naive React patterns:
1. The component needs the label value as a string to pass as `aria-label` when collapsed — parsing ReactNode for text content is fragile and not SSR-safe.
2. It prevents consumers from placing complex JSX inside the button label, which would break collapse animation.
3. It makes the accessible name a first-class, compile-enforced concern.

If rich label formatting is required (e.g. bold substring), it must be done via CSS on the `.extended-fab__label` element, not via ReactNode children.

**BEM classes emitted:**
- `.extended-fab` (always)
- `.extended-fab--{color}` e.g. `.extended-fab--primary` (no modifier = `surface`)
- `.extended-fab--lowered` when `lowered={true}`
- `.extended-fab--collapsed` when `collapsed={true}`

**React Aria base:** `Button` from `react-aria-components`.

> **Cross-component consistency note:** both `Fab` and `ExtendedFab` use `color` (not `variant`) for the same four M3 color options. The lead should ensure this naming is consistent across the component index.

> **Layer safety:** all styles in `@layer kafui { … }`. The bare `.extended-fab` class is safe.
