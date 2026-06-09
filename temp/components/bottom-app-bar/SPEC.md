# Bottom App Bar

**Purpose:** A mobile-only surface fixed to the bottom of the screen, providing quick access to 3–4 icon actions and optionally a FAB (Floating Action Button). It anchors the primary interaction affordances for a given screen on small viewports. M3 category: **Navigation → Bottom App Bar**.

> **Expressive note:** The M3 Expressive spec supersedes the Bottom App Bar with the **Docked Toolbar** — a more flexible surface that supports labels on icon buttons, variable action counts, and richer layout control. New designs should prefer the Docked Toolbar (`toolbar` component). This component is documented for M3 baseline conformance and migration reference.

---

## Anatomy / parts

| BEM element | Description |
|---|---|
| `.bottom-app-bar` | Root container; `<div role="toolbar">` fixed to bottom of viewport |
| `.bottom-app-bar__content` | Inner flex row; sets height (80 dp per M3 spec) and padding |
| `.bottom-app-bar__actions` | `inline-start`-aligned cluster of up to 4 icon buttons |
| `.bottom-app-bar__action` | Individual icon-button wrapper; `<Button>` from RAC |
| `.bottom-app-bar__fab` | `inline-end`-aligned slot for a FAB sibling component |

The `__fab` slot is optional; its presence switches the container to `--with-fab` layout. The Bottom App Bar does not render the FAB itself — the consumer passes `<Fab>` or `<ExtendedFab>`.

---

## Variants

| Variant | `variant` prop | Description |
|---|---|---|
| Default | `"default"` | Actions left-aligned; no FAB slot |
| With FAB | `"with-fab"` | FAB at `inline-end`; actions cluster at `inline-start`; FAB overlaps the bar edge (M3: FAB extends above the bar by ~8 dp) |

No other M3 variants. The bar is always full-width, single-row.

---

## States

| State | Behaviour |
|---|---|
| Default | Visible; `--surface-container` background; `--elevation-2` shadow/tonal overlay |
| Scrolled under content | Elevation stays at level 2 (no change — unlike Top App Bar) |
| Hidden (scroll-away) | `transform: translateY(100%)` + transition on downward scroll; revealed on upward scroll; consumer drives via `isHidden` prop or `useScrollSentinel` hook |
| Icon action hover | State layer 8% `--on-surface-variant` |
| Icon action focused | State layer 10%; focus ring |
| Icon action pressed | State layer 10% |
| Icon action disabled | 38% opacity `--on-surface`; no state layer |

---

## Design tokens

All tokens are **unprefixed system roles** consumed via `@layer kafui`.

### Color
| Role | CSS custom property |
|---|---|
| Bar background | `--surface-container` |
| Inactive icon fill | `--on-surface-variant` |
| Active/selected icon fill | `--on-surface` |
| Tonal elevation overlay tint | `--surface-tint` |

### Elevation
- Always: `--elevation-2`

### Shape
- `--corner-none` — full-width edge surface; no radius.

### Motion
| Action | Token |
|---|---|
| Scroll-hide easing | `--easing-emphasized-accelerate` |
| Scroll-reveal easing | `--easing-emphasized-decelerate` |
| Scroll-hide duration | `--duration-short3` |
| Scroll-reveal duration | `--duration-medium2` |

### State layer opacities
- Hover: `--state-hover` (0.08)
- Focus/pressed: `--state-focus` / `--state-pressed` (0.10)

### Component-internal vars (inside `.bottom-app-bar {}`)
```css
.bottom-app-bar {
  --bar-h: 80px;
  --fab-overlap: 8px; /* how far FAB extends above bar top */
}
```

---

## CSS structure (in `@layer kafui`)

```css
@layer kafui {
  .bottom-app-bar {
    --bar-h: 80px;
    --fab-overlap: 8px;

    position: fixed;
    inset-block-end: 0;
    inset-inline: 0;
    background: var(--surface-container);
    box-shadow: var(--elevation-2);
    transition:
      transform var(--duration-short3) var(--easing-emphasized-accelerate);
  }

  .bottom-app-bar__content {
    display: flex;
    align-items: center;
    block-size: var(--bar-h);
    padding-inline: 4px;
  }

  .bottom-app-bar__actions {
    display: flex;
    align-items: center;
    gap: 0;
    color: var(--on-surface-variant);
  }

  .bottom-app-bar__fab {
    margin-inline-start: auto;
    /* FAB overlaps the bar top edge — use negative margin-block-start */
    margin-block-start: calc(-1 * var(--fab-overlap));
    align-self: flex-start;
  }

  /* Hidden state: slide off screen */
  .bottom-app-bar--hidden {
    transform: translateY(100%);
    /* Reveal uses decelerate easing */
    transition-timing-function: var(--easing-emphasized-decelerate);
    transition-duration: var(--duration-medium2);
  }

  /* Reduced motion: use opacity instead of transform */
  @media (prefers-reduced-motion: reduce) {
    .bottom-app-bar {
      transition: opacity var(--duration-short2);
    }
    .bottom-app-bar--hidden {
      opacity: 0;
      pointer-events: none;
      transform: none;
    }
  }

  /* State layer on each action — shared .state-layer pattern */
  .bottom-app-bar__action {
    position: relative;
  }
  .bottom-app-bar__action .state-layer::after {
    content: "";
    position: absolute;
    inset: 0;
    border-radius: inherit;
    background: var(--on-surface-variant);
    opacity: 0;
    transition: opacity var(--duration-short2) var(--easing-standard);
  }
  .bottom-app-bar__action[data-hovered] .state-layer::after  { opacity: var(--state-hover); }
  .bottom-app-bar__action[data-focused] .state-layer::after  { opacity: var(--state-focus); }
  .bottom-app-bar__action[data-pressed] .state-layer::after  { opacity: var(--state-pressed); }
}
```

---

## Interaction & accessibility

### Role & Landmark
- Root element: `<div role="toolbar" aria-label="...">`. `role="toolbar"` is correct — this bar holds **actions** (Attach, Search, More), not navigation destinations. If destinations are included, use `NavigationBar` instead.
- `aria-label` is required at TypeScript level (not optional).

### Keyboard
- `Tab` moves between focusable icon buttons.
- Roving tabindex (`ArrowLeft`/`ArrowRight`) per WAI-ARIA Toolbar pattern: `tabindex="0"` on the focused button, `tabindex="-1"` on others; arrow keys move the roving index.
- `Home`/`End` jump to first/last button.
- FAB in `__fab` participates in roving focus order **after** the last action button.
- Disabled buttons: `aria-disabled="true"`, `tabindex="-1"`, skipped in arrow-key navigation.

### ARIA
- Each action: `aria-label` required.
- Toggle actions: `aria-pressed="true|false"`.
- No `aria-current` — not navigation.
- FAB: manages its own `aria-label`.

### Focus ring
- Each action button: focus ring 3 dp, `--secondary` color (provided by RAC `Button` `data-focus-visible`).

### Touch targets
- Each action: minimum 48×48 dp via `::before` pseudo-element technique on `__action`.

### RTL
- `__actions` at `inline-start`; `__fab` at `inline-end` — logical properties; flips automatically under `dir="rtl"`.

### Reduced motion
- Scroll-hide transition replaced with `opacity` toggle (no `transform`).

---

## Proposed kafUI React API

```tsx
// React Aria: each action uses RAC <Button>
// Roving tabindex via RAC Toolbar is NOT used here — the Bottom App Bar uses
// a plain <div role="toolbar"> with manual roving implemented via useRovingTabIndex
// hook, so that the FAB can participate in the same focus ring.

interface BottomAppBarProps {
  fab?: React.ReactNode;            // slotted FAB; presence triggers --with-fab layout
  'aria-label': string;             // required; forwarded to role="toolbar"
  isHidden?: boolean;               // controlled hide; toggles --hidden modifier
  rovingFocus?: boolean;            // default: true; enables ArrowLeft/Right/Home/End
  children: React.ReactNode;        // <BottomAppBar.Action> elements
  className?: string;
}

interface BottomAppBarActionProps {
  icon: string;                     // sprite name
  'aria-label': string;             // required
  isDisabled?: boolean;
  isSelected?: boolean;             // maps to aria-pressed; for toggle states
  onPress?: (e: PressEvent) => void;
  className?: string;
}

// Scroll visibility helper (same hook used by TopAppBar + Toolbar):
// const { isHidden } = useScrollSentinel(scrollTarget);
// <BottomAppBar isHidden={isHidden} aria-label="Screen actions" ...>

// Compound usage:
<BottomAppBar aria-label="Screen actions" fab={<Fab icon="edit" aria-label="Compose" />}>
  <BottomAppBar.Action icon="search"      aria-label="Search"       onPress={handleSearch} />
  <BottomAppBar.Action icon="attach_file" aria-label="Attach"       onPress={handleAttach} />
  <BottomAppBar.Action icon="more_vert"   aria-label="More actions" onPress={handleMore} />
</BottomAppBar>
```

**BEM classes emitted:**
- Root: `.bottom-app-bar`, `.bottom-app-bar--with-fab` (when `fab` prop present), `.bottom-app-bar--hidden`
- Content: `.bottom-app-bar__content`
- Actions: `.bottom-app-bar__actions`
- Action: `.bottom-app-bar__action` (inner button uses `icon-button` BEM)
- FAB slot: `.bottom-app-bar__fab`

**React Aria base:** RAC `Button` for each action. RAC `Toolbar` is intentionally NOT used as the root because the FAB must participate in the same roving focus ring and RAC Toolbar does not include non-Button children in focus management. Roving tabindex implemented manually with a `useRovingTabIndex` utility that treats all buttons (actions + FAB) as one contiguous group.

**Deviations / justifications:**
- `isHidden` (not `hidden`) — aligns with React Aria naming convention (`isDisabled`, `isSelected`, `isReadOnly`).
- `fab` is a slot prop; consumer owns FAB props. The bar does not prescribe FAB variant or size.
- `rovingFocus={false}` escape hatch for consumers who want plain tab-order only.
- No internal scroll listener — consumer uses `useScrollSentinel` or custom logic.
