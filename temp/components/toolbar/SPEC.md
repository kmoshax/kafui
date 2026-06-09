# Toolbar (M3 Expressive)

The M3 Expressive Toolbar is a **floating or docked row of contextual action buttons** — icon buttons plus an optional FAB — that anchors to the bottom of the screen. It is the Expressive successor to the M3 standard Bottom App Bar, emphasising bold shape, colour variety, and flexible docking. M3 category: **Navigation → Toolbar (Expressive)**.

> **Not MUI Toolbar.** MUI's `Toolbar` is a zero-semantic horizontal flex container inside `AppBar`. This component lives at the **bottom** of the screen, is pill-shaped or full-width, carries `role="toolbar"` with full roving-focus semantics, and is a standalone M3 Expressive pattern.

---

## Anatomy / parts

| BEM element | Description |
|---|---|
| `.toolbar` | Root container; pill-shaped (floating) or rectangular (docked) surface |
| `.toolbar__actions` | Inner RAC `Toolbar` (`role="toolbar"`) — holds icon buttons; roving focus lives here |
| `.toolbar__fab` | Optional FAB slot; leading or trailing relative to `__actions` |
| `.toolbar__divider` | Optional 1 dp vertical rule between FAB and `__actions`; rendered automatically when both present |

Icon buttons within `__actions` are `<IconButton>` components (BEM: `.icon-button`). The Toolbar owns no button styles.

---

## Variants

| Variant | Container shape | Positioning | Elevation | Background | `variant` prop |
|---|---|---|---|---|---|
| `floating` | All-corner `--corner-extra-large` (28 dp) or `--corner-full` at narrow widths | Centered or end-aligned, with gap from screen edge; consumer applies `position: fixed` | `--elevation-2` | `--surface-container-high` | `"floating"` (default) |
| `docked` | Top corners only: `--corner-extra-large`; bottom corners `--corner-none`; flush with bottom edge | Full-width, no gap; consumer applies `position: fixed; bottom: 0; inset-inline: 0` | `--elevation-0` | `--surface-container` | `"docked"` |

**Vibrant / expressive color variants (M3 Expressive):** The Toolbar container can optionally adopt `--primary-container` / `--on-primary-container` for a vibrant look. Exposed as a `color` prop:

| `color` prop | Background | Icon color |
|---|---|---|
| `"surface"` (default) | `--surface-container-high` (floating) / `--surface-container` (docked) | `--on-surface-variant` |
| `"primary"` | `--primary-container` | `--on-primary-container` |
| `"secondary"` | `--secondary-container` | `--on-secondary-container` |
| `"tertiary"` | `--tertiary-container` | `--on-tertiary-container` |

---

## States

### Container
The toolbar container is not interactive. No state layer on container.

### FAB slot
The FAB follows its own state model (level 3 → 4 elevation on hover), independent of the toolbar container elevation.

### Visibility

| Modifier | Meaning |
|---|---|
| `.toolbar--hidden` | Off-screen; `translateY(100% + gap)` for floating, `translateY(100%)` for docked |
| (no modifier) | Resting / visible |

Show/hide driven by consumer (`isVisible` prop or `useScrollSentinel` hook). Not internal to the component.

---

## Design tokens

All tokens are **unprefixed system roles** via `@layer kafui`.

### Color
Component-internal vars set per variant and `color` prop:
```css
.toolbar {
  --toolbar-bg:   var(--surface-container-high); /* floating default */
  --toolbar-icon: var(--on-surface-variant);
}
.toolbar--docked        { --toolbar-bg: var(--surface-container); }
.toolbar--primary       { --toolbar-bg: var(--primary-container);   --toolbar-icon: var(--on-primary-container); }
.toolbar--secondary     { --toolbar-bg: var(--secondary-container);  --toolbar-icon: var(--on-secondary-container); }
.toolbar--tertiary      { --toolbar-bg: var(--tertiary-container);   --toolbar-icon: var(--on-tertiary-container); }
```

### Shape
- Floating: `border-radius: var(--corner-extra-large)` (all corners)
- Docked: `border-start-start-radius: var(--corner-extra-large); border-start-end-radius: var(--corner-extra-large); border-end-start-radius: var(--corner-none); border-end-end-radius: var(--corner-none);`

### Elevation
- Floating resting: `--elevation-2`
- Floating drag (repositionable, rare): `--elevation-4`
- Docked: `--elevation-0`

### Motion
| Action | Easing token | Duration token |
|---|---|---|
| Show (enter) | `--easing-emphasized-decelerate` | `--duration-medium2` |
| Hide (exit) | `--easing-emphasized-accelerate` | `--duration-short4` |
| Reduced motion | `opacity` only; no `transform` | `--duration-short2` |

### Divider
- `--outline-variant` (1 dp wide, 32 dp tall)

### State layer (icon buttons within)
- Hover: `--state-hover` (0.08)
- Focus: `--state-focus` (0.10)
- Pressed: `--state-pressed` (0.10)

### Component-internal vars
```css
.toolbar {
  --toolbar-bg:    var(--surface-container-high);
  --toolbar-icon:  var(--on-surface-variant);
  --toolbar-gap:   16px; /* floating: gap from screen edge */
  --toolbar-pad-x: 16px;
  --toolbar-pad-y: 12px;
}
```

---

## CSS structure (in `@layer kafui`)

```css
@layer kafui {
  .toolbar {
    --toolbar-bg:    var(--surface-container-high);
    --toolbar-icon:  var(--on-surface-variant);
    --toolbar-gap:   16px;
    --toolbar-pad-x: 16px;
    --toolbar-pad-y: 12px;

    display: inline-flex;
    align-items: center;
    background: var(--toolbar-bg);
    color: var(--toolbar-icon);
    padding: var(--toolbar-pad-y) var(--toolbar-pad-x);
    gap: 0;
    border-radius: var(--corner-extra-large);
    box-shadow: var(--elevation-2);
    /* Positioning: consumer applies position: fixed via wrapper or className */
  }

  .toolbar--docked {
    --toolbar-bg:    var(--surface-container);
    --toolbar-pad-x: 8px;
    inline-size: 100%;
    justify-content: space-around;
    border-start-start-radius: var(--corner-extra-large);
    border-start-end-radius:   var(--corner-extra-large);
    border-end-start-radius:   var(--corner-none);
    border-end-end-radius:     var(--corner-none);
    box-shadow: var(--elevation-0);
  }

  /* Vibrant color variants */
  .toolbar--primary   { --toolbar-bg: var(--primary-container);   --toolbar-icon: var(--on-primary-container); }
  .toolbar--secondary { --toolbar-bg: var(--secondary-container);  --toolbar-icon: var(--on-secondary-container); }
  .toolbar--tertiary  { --toolbar-bg: var(--tertiary-container);   --toolbar-icon: var(--on-tertiary-container); }

  .toolbar__actions {
    display: flex;
    align-items: center;
    gap: 4px;
  }

  .toolbar__divider {
    inline-size: 1px;
    block-size: 32px;
    background: var(--outline-variant);
    margin-inline: 4px;
    flex-shrink: 0;
  }

  .toolbar__fab {
    /* Leading FAB */
    margin-inline-end: 0; /* gap provided by __divider */
  }
  .toolbar--fab-end .toolbar__fab {
    order: 1; /* after __actions */
  }

  /* Hide/show animations */
  .toolbar--hidden {
    transform: translateY(calc(100% + var(--toolbar-gap)));
    transition:
      transform var(--duration-short4) var(--easing-emphasized-accelerate);
  }
  .toolbar:not(.toolbar--hidden) {
    transform: translateY(0);
    transition:
      transform var(--duration-medium2) var(--easing-emphasized-decelerate);
  }

  .toolbar--docked.toolbar--hidden {
    transform: translateY(100%);
  }

  @media (prefers-reduced-motion: reduce) {
    .toolbar,
    .toolbar--hidden {
      transform: none;
      transition: opacity var(--duration-short2);
    }
    .toolbar--hidden {
      opacity: 0;
      pointer-events: none;
    }
  }
}
```

---

## Interaction & accessibility

**ARIA role:**
- `.toolbar__actions` renders as RAC `Toolbar` → emits `role="toolbar"` with `aria-label` forwarded.
- The outer `.toolbar` container is a plain `<div>` (no role); it is not interactive.
- Each icon button: `role="button"` (implicit via `<button>`), `aria-label` required.
- FAB: `aria-label` required.

**Keyboard navigation (WAI-ARIA Toolbar pattern, via RAC Toolbar):**
- `Tab` enters the toolbar (focuses the roving item); next `Tab` exits to next focusable outside.
- `←` / `→` rove focus between buttons within `role="toolbar"`. `tabindex="0"` on the focused button; `tabindex="-1"` on others.
- `Home` / `End` jump to first / last button.
- Disabled buttons: `aria-disabled="true"`, `tabindex="-1"`, skipped in arrow-key navigation.
- The FAB **participates in the same roving order** as icon buttons. Because RAC `Toolbar` manages focus only over its direct children, the FAB must be rendered inside `__actions` (not in a separate `__fab` wrapper outside the RAC Toolbar element) if roving focus over FAB is required — **or** a custom `useRovingTabIndex` hook spans both `__actions` and `__fab`. Default implementation: FAB is inside the RAC `Toolbar` children, separated visually by `__divider`.

**Toggle buttons:** Icon buttons within the toolbar may be toggle buttons (`aria-pressed="true|false"`). Toolbar does not manage group selection state.

**Focus ring:** Each icon button shows its own `focus-visible` ring. Toolbar container has no focus ring.

**Landmark:** Toolbar is not wrapped in `<nav>` or `<header>`. Consumers may wrap in `<footer aria-label="Toolbar">` for a persistent bottom surface.

**RTL:** RAC `Toolbar` flips arrow-key direction from DOM `dir` attribute automatically. Icon order is reversed via `flex-direction: row-reverse` on `__actions` under `[dir="rtl"]`. Directional icons mirrored in CSS.

**Reduced motion:** Show/hide uses `opacity` only (no `transform`).

**Touch target:** Each icon button ≥ 48×48 dp; spacing via internal padding on `.icon-button`.

---

## Proposed kafUI React API

```tsx
// react-aria-components base: Toolbar (for __actions)

type ToolbarVariant = 'floating' | 'docked';
type ToolbarColor = 'surface' | 'primary' | 'secondary' | 'tertiary';

interface ToolbarProps {
  variant?: ToolbarVariant;           // default: 'floating'
  color?: ToolbarColor;               // default: 'surface'
  'aria-label': string;               // required; forwarded to role="toolbar"
  fab?: React.ReactNode;              // FAB element slot
  fabPosition?: 'start' | 'end';     // default: 'end'
  isVisible?: boolean;                // controlled show/hide; default: true
  onVisibleChange?: (visible: boolean) => void;
  className?: string;
  children: React.ReactNode;          // <IconButton> elements for the action row
}
```

**BEM classes emitted:**
- `.toolbar` (root)
- `.toolbar--floating` / `.toolbar--docked`
- `.toolbar--surface` / `--primary` / `--secondary` / `--tertiary` (color variant)
- `.toolbar--hidden` when `isVisible={false}`
- `.toolbar--fab-end` / `.toolbar--fab-start` (layout modifier for FAB position)
- `.toolbar__actions` (inner RAC Toolbar)
- `.toolbar__fab` (FAB wrapper)
- `.toolbar__divider` (auto-rendered when `fab` prop present)

**React Aria base:** RAC `Toolbar` wraps the `__actions` row. The outer `.toolbar` `<div>` is a plain container. RAC `Toolbar` provides `role="toolbar"`, `aria-label` forwarding, roving tabindex, `←`/`→`/`Home`/`End`, and disabled-item skipping. Orientation is always `horizontal`.

**FAB in roving focus:** FAB is rendered as the first or last child of the RAC `Toolbar` (inside `__actions`) with a `.toolbar__divider` as a non-interactive separator sibling. This keeps all buttons in one RAC `Toolbar` focus group. The `__divider` is `aria-hidden="true"`.

**Scroll visibility:** Consumers use the shared `useScrollSentinel` hook (exported from `@kafui/react`) or manage `isVisible` themselves. The component does not auto-observe scroll.

**Positioning note:** `.toolbar` does not apply `position: fixed` by default — composability first. Consumers apply positioning via `className` or a CSS utility (e.g. `class="toolbar-fixed-bottom"`). Document this clearly.
