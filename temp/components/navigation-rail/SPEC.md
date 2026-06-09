# Navigation Rail

**Purpose:** A compact vertical navigation surface anchored to the inline-start edge, providing destination switching for medium window classes (tablets, wide phones in landscape, desktop sidebars). Bridges the gap between the Navigation Bar (compact/mobile) and Navigation Drawer (large/desktop). Supports 3–7 destinations with icons and optional labels.  
**M3 category:** Navigation → Navigation Rail.

> **Expressive note:** M3 Expressive introduces an **expandable/collapsible** Navigation Rail that animates between icon-only (collapsed, 80 dp) and icon+label (expanded, 256 dp) layouts. The collapsed state shares the same indicator pill anatomy as the Navigation Bar; the expanded state shares full-row indicator anatomy with the Navigation Drawer. This duality is the key design challenge — both states use the same item component with CSS driving the layout switch.

---

## Anatomy / Parts → BEM Elements

```
.navigation-rail                       root <nav>; fixed to inline-start, full viewport height
  .navigation-rail__header             top area: optional menu-icon button + optional FAB slot
    .navigation-rail__menu-button      hamburger/menu icon <button> (opens drawer)
    .navigation-rail__fab              FAB slot (rendered above destinations)
  .navigation-rail__destinations       <ul role="list">; flex column; vertically centered or top-aligned
    .navigation-rail__item             <li> wrapper
      <a> / RAC <Link>                 interactive element; carries aria-current
        .navigation-rail__item-indicator  pill container; receives state layer; background filled on active
          .navigation-rail__item-icon    <Icon> sprite (24 dp)
        .navigation-rail__item-label   destination label (below indicator in collapsed; inline-end in expanded)
        .navigation-rail__item-badge   optional <Badge> anchored to indicator
```

The `__item-indicator` pill wraps only the icon (56 dp wide) in the default collapsed state. In the expanded state it spans the full item row (like the drawer indicator). The badge is positioned absolutely on `__item-indicator`.

> **Shared indicator pattern:** The active indicator (pill shape, `--secondary-container` fill, animated) is identical across Navigation Bar, Navigation Rail, and Navigation Drawer — only its width changes: 64 dp (bar), 56 dp (rail collapsed), full-row (drawer + rail expanded). These three components share a common CSS mechanism; see Cross-Component Notes.

---

## Variants

| Variant | `variant` prop | Description |
|---------|---------------|-------------|
| Standard | `"standard"` | Icon + label below; header with optional menu + FAB; baseline M3 |
| No header | `"no-header"` | Destinations only; no menu button or FAB slot |
| Expandable | `"expandable"` | Adds `isExpanded` prop; when expanded, labels move inline-end of icon (full-row layout); animated width transition — M3 Expressive |

---

## States

### Rail-level (Expressive expandable)

| State | CSS |
|-------|-----|
| Collapsed | `--rail-w: 80px`; icon + label stacked |
| Expanded | `--rail-w: 256px`; icon + label inline-end |
| Animating | `transition: width var(--duration-medium3) var(--easing-emphasized)` |

Use `var(--easing-emphasized-decelerate)` for the expand-open motion and `var(--easing-emphasized-accelerate)` for collapse-close.

### Item states

| State | Visual |
|-------|--------|
| Inactive | Icon `--on-surface-variant`; label `--on-surface-variant`; no indicator background |
| Active | Indicator filled `--secondary-container`; icon `--on-secondary-container`; label `--on-surface`; `aria-current="page"` |
| Hover (inactive) | State-layer 8% `--on-surface` on `__item-indicator` |
| Hover (active) | State-layer 8% `--on-secondary-container` on `__item-indicator` |
| Focus-visible | State-layer 10%; `outline` on `<a>` |
| Pressed | State-layer 10% |
| Disabled | 38% opacity; `aria-disabled="true"` |

State layers are scoped to `__item-indicator` (not the full item height), per M3 spec.

---

## Design Tokens

| Token (unprefixed CSS custom property) | Role |
|----------------------------------------|------|
| `--surface` | rail background (flat, no elevation) |
| `--surface-container-low` | alternative background (subtle variant) |
| `--on-surface-variant` | inactive icon + label |
| `--on-surface` | active label |
| `--secondary-container` | active indicator pill fill |
| `--on-secondary-container` | active icon color |
| `--elevation-0` | rail is flat (no shadow) |
| `--corner-full` | indicator pill (fully rounded) |
| `--label-medium-size`, `--label-medium-weight`, `--label-medium-font` | destination label (inactive) |
| `--label-large-size`, `--label-large-weight`, `--label-large-font` | destination label (active, optional size shift) |
| `--duration-short2` | indicator pill activate transition |
| `--duration-medium3` | expandable rail width transition |
| `--easing-emphasized` | default transition easing |
| `--easing-emphasized-decelerate` | expand-open easing |
| `--easing-emphasized-accelerate` | collapse-close easing |
| `--state-hover` | 0.08 |
| `--state-focus` | 0.10 |
| `--state-pressed` | 0.10 |

Component-internal vars (scoped inside `.navigation-rail { … }`):

```css
@layer kafui {
  .navigation-rail {
    --rail-w: 80px;              /* collapsed width */
    --rail-w-expanded: 256px;   /* expanded width */
    --indicator-w: 56px;        /* pill width (collapsed active) */
    --indicator-h: 32px;
    --item-min-h: 56px;         /* M3 touch target */
  }
}
```

---

## Interaction & Accessibility

### Why `<nav>` + links, not Tabs or Tree

Navigation Rail items are route destinations — they navigate to pages, not switch content panels. `role="tablist"` requires `aria-controls` to panels; `role="tree"` implies expandable hierarchy — neither applies. `<nav>` + `<a>` + `aria-current="page"` is correct per ARIA authoring practices.

### Landmark & Structure
- Root: `<nav aria-label="Application navigation">`.
- Destinations: `<ul role="list">` inside `__destinations`.
- Each item: `<li>` > `<a href="...">` (RAC `<Link>`) with `aria-current="page"` on active.
- Menu button in `__header`: RAC `<Button aria-label="Open navigation menu" aria-expanded={drawerOpen}>`. This is a `<button>`, not a link.
- FAB in `__header`: standard `<Fab>` component, independently focusable.

### `aria-current`
Active item: `aria-current="page"`. Inactive items: omit `aria-current`.

### Keyboard (destinations)
- Tab traverses all focusable elements in DOM order: menu button → FAB → destinations → (nothing below).
- No roving focus within destinations — they are links, not a composite widget.
- Enter / Space activates a focused link.

### Expandable rail keyboard
- The expand/collapse toggle is a RAC `<Button>` in `__header` with `aria-expanded={isExpanded}`.
- If `variant="expandable"` and no explicit `menuButton`, a default toggle button is auto-rendered.
- Expanding/collapsing does not change the keyboard model for destinations.

### Focus ring
Visible `outline` on `<a>` using `focus-visible`. Ring color: `--secondary`.

### Badge integration
`__item-badge` anchored to `__item-indicator`. Item `<a>` gets `aria-label="Destinations, 3 unread"` when a badge is present.

### Reduced motion
`@media (prefers-reduced-motion: reduce)`:
- Disable indicator pill transition (instant fill).
- Disable rail width transition (instant expand/collapse).

### RTL
- Rail anchors to `inset-inline-start: 0`. Under `dir="rtl"` this is the right edge automatically — no JS needed.
- Expanded label position uses `margin-inline-start` so it sits inline-end of icon in both LTR and RTL.
- Directional icons mirrored via `<Icon rtlMirror>`.

---

## Proposed kafUI React API

```tsx
// React Aria: each item uses RAC <Link>; menu/expand button uses RAC <Button>.
// No RAC composite for the rail itself — plain <nav> structure.

type NavigationRailVariant = "standard" | "no-header" | "expandable";

interface NavigationRailProps {
  /** "standard" (default) | "no-header" | "expandable" */
  variant?: NavigationRailVariant;
  /** Accessible label for the <nav> landmark. Default: "Application navigation" */
  "aria-label"?: string;
  /**
   * Optional menu icon button in header.
   * true → renders default hamburger icon button.
   * ReactNode → renders custom node in __menu-button slot.
   */
  menuButton?: boolean | React.ReactNode;
  /** Called when menu button is pressed */
  onMenuPress?: (e: PressEvent) => void;
  /** aria-expanded state for menu button */
  menuIsExpanded?: boolean;
  /** Optional FAB rendered in header slot (below menu button) */
  fab?: React.ReactNode;
  /**
   * Expressive: controls expanded state.
   * Only meaningful when variant="expandable".
   * Controlled — caller decides expansion (e.g. based on breakpoint).
   */
  isExpanded?: boolean;
  /** Called when the expand/collapse toggle is pressed */
  onExpandChange?: (expanded: boolean) => void;
  children: React.ReactNode; // <NavigationRail.Item> elements (3–7)
  className?: string;
}

interface NavigationRailItemProps {
  /** Route href — required to enforce link semantics */
  href: string;
  /** Icon sprite name */
  icon: string;
  /** Destination label — always required; used for aria-label when labels are hidden */
  label: string;
  /** Marks as current destination → aria-current="page" */
  isCurrent?: boolean;
  badge?: React.ReactNode;
  isDisabled?: boolean;
  onPress?: (e: PressEvent) => void;
  className?: string;
}

// Standard usage:
<NavigationRail
  menuButton
  menuIsExpanded={drawerOpen}
  onMenuPress={() => setDrawerOpen(true)}
  fab={<Fab icon="edit" aria-label="Compose" />}
>
  <NavigationRail.Item href="/" icon="home" label="Home" isCurrent />
  <NavigationRail.Item href="/explore" icon="search" label="Explore" />
  <NavigationRail.Item href="/library" icon="library_music" label="Library" />
</NavigationRail>

// Expressive expandable (caller-controlled):
<NavigationRail
  variant="expandable"
  isExpanded={isWideLayout}
  onExpandChange={setIsWideLayout}
>
  <NavigationRail.Item href="/" icon="home" label="Home" isCurrent />
  <NavigationRail.Item href="/explore" icon="search" label="Explore" />
</NavigationRail>
```

**BEM classes emitted (unprefixed, inside `@layer kafui`):**
- Root `<nav>`: `.navigation-rail`, `.navigation-rail--standard` / `--no-header` / `--expandable`, `.navigation-rail--expanded`
- Header `<div>`: `.navigation-rail__header`
- Menu button slot `<div>`: `.navigation-rail__menu-button`
- FAB slot `<div>`: `.navigation-rail__fab`
- Destinations `<ul>`: `.navigation-rail__destinations`
- Item `<li>`: `.navigation-rail__item`, `.navigation-rail__item--active`
- Indicator `<span>`: `.navigation-rail__item-indicator`
- Icon `<span>`: `.navigation-rail__item-icon`
- Label `<span>`: `.navigation-rail__item-label`
- Badge `<span>`: `.navigation-rail__item-badge`

RAC `data-hovered`, `data-focused`, `data-pressed`, `data-disabled` from `<Link>` drive state-layer CSS on `__item-indicator`.

**React Aria base:**
- Each destination item: RAC `<Link>` (pointer/keyboard/focus-visible/SPA routing).
- Menu button: RAC `<Button>` with `aria-expanded` + `aria-label`.
- No RAC composite for the rail itself.

**API decisions and justifications:**
- `menuButton` accepts `true` (default hamburger) or a custom `React.ReactNode`; passing `false` / omitting renders no menu button.
- `isExpanded`/`onExpandChange` is always caller-controlled — the parent decides based on breakpoint or user preference. No internal `useState` for this.
- `variant="expandable"` auto-renders a collapse/expand toggle in the header if `menuButton` is not provided.
- No MUI core equivalent: MUI has no Navigation Rail — this is a deliberate kafUI differentiator.
- Label is always required: critical when `isExpanded={false}` hides labels, and for `aria-label` when badge count needs concatenation.
