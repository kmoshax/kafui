# Navigation Bar

**Purpose:** Primary navigation surface for mobile screens (compact window class). Provides 3–5 destination items, each with an icon, active-indicator pill, label, and optional badge. Always visible at the bottom of the screen; replaces the bottom tab bar from earlier Material versions.  
**M3 category:** Navigation → Navigation Bar.

---

## Anatomy / Parts → BEM Elements

```
.navigation-bar                    root <nav> fixed to bottom of viewport
  .navigation-bar__items           horizontal flex row (equally-spaced items)
    .navigation-bar__item          <li> wrapper for one destination
      <a> / RAC <Link>             the interactive element (carries aria-current)
        .navigation-bar__icon-container   contains indicator + icon; receives state layer
          .navigation-bar__indicator      active-indicator pill (always in DOM)
          .navigation-bar__icon           <Icon> sprite (24 dp)
        .navigation-bar__badge          optional <Badge> anchor overlay (on icon-container)
        .navigation-bar__label          destination label text
```

The `__indicator` pill is always present in DOM; it has `width: 0` (or `opacity: 0`) when inactive and expands to `64px` on activation via CSS transition. The `__badge` is positioned absolutely relative to `__icon-container`. The state layer (`::after` pseudo-element per the shared `.state-layer` pattern) is applied on `__icon-container`, not on the full item, per M3 spec — the label does not receive state-layer tints.

> **Active-indicator without layout thrash:** The indicator is a child of `__icon-container`, sized absolutely (or via `width` transition on an in-flow pill). No JS measurement needed. The item width never changes when the indicator appears — indicator expansion is purely visual, achieved with `transform: scaleX()` on a centered pill or `width` on an overflow-hidden container, avoiding any reflow.

---

## Variants

| Variant | `variant` prop | Description |
|---------|---------------|-------------|
| Standard | `"standard"` | Icons + labels always visible; M3-spec default |
| Icon-only | `"icon-only"` | Labels hidden visually; `label` prop still required for `aria-label`; use only when meaning is unambiguous |

M3 spec only defines a single layout (icon + label), but the icon-only variant is a common real-world need. It is an explicit degradation from spec; consumers should prefer `"standard"`.

---

## States

### Item states

| State | Visual |
|-------|--------|
| Inactive / default | Icon `--on-surface-variant`; no indicator fill; label `--on-surface-variant` |
| Active | Icon `--on-secondary-container`; indicator pill filled `--secondary-container`; label `--on-surface` (weight/size shift); `aria-current="page"` on `<a>` |
| Hover (inactive) | State-layer `::after` 8% `--on-surface` on `__icon-container` |
| Hover (active) | State-layer 8% `--on-secondary-container` on `__icon-container` |
| Focus-visible (inactive) | State-layer 10% `--on-surface`; `outline` on `<a>` |
| Focus-visible (active) | State-layer 10% `--on-secondary-container`; `outline` on `<a>` |
| Pressed (inactive) | State-layer 10% `--on-surface` |
| Pressed (active) | State-layer 10% `--on-secondary-container` |
| Disabled | 38% opacity; no state layer; `pointer-events: none`; `aria-disabled="true"` |

State layers are scoped to `__icon-container` per M3 spec — the label area does not receive state-layer tints.

---

## Design Tokens

| Token (unprefixed CSS custom property) | Role |
|----------------------------------------|------|
| `--surface-container` | bar background |
| `--on-surface-variant` | inactive icon + label color |
| `--on-surface` | active label color |
| `--secondary-container` | active indicator pill fill |
| `--on-secondary-container` | active icon color (inside pill) |
| `--elevation-2` | bar surface elevation (tonal tint + shadow) |
| `--corner-full` | indicator pill shape (fully rounded) |
| `--label-medium-size`, `--label-medium-weight`, `--label-medium-font` | destination label (inactive) |
| `--label-large-size`, `--label-large-weight`, `--label-large-font` | destination label (active) |
| `--state-hover` | 0.08 |
| `--state-focus` | 0.10 |
| `--state-pressed` | 0.10 |
| `--duration-short2` | indicator pill expand/contract |
| `--easing-emphasized` | indicator pill animation |

Component-internal vars (scoped inside `.navigation-bar { … }`):

```css
@layer kafui {
  .navigation-bar {
    --h: 80px;                  /* bar height */
    --indicator-w: 64px;        /* pill width when active */
    --indicator-h: 32px;        /* pill height */
    --item-min-w: 48px;         /* per M3 touch target */
  }
}
```

---

## Interaction & Accessibility

### Why `<nav>` + links, not Tabs

Navigation bar destinations are **page/route destinations** — semantically links, not tab panels. Using `role="tablist"` / `role="tab"` would imply panel-switching ARIA semantics (requiring `aria-controls`, `aria-selected`, hidden panels), which is incorrect for app-level routing. A `<nav>` landmark with `<a>` elements gives AT users:
- Correct "navigation" landmark announcement.
- Each item announced as a link, not a tab.
- `aria-current="page"` on the active item signals current location without implying content panels.

### Landmark & Structure
- Root: `<nav aria-label="Main navigation">` (label customizable via prop).
- Items list: `<ul role="list">` inside `<nav>` — `<ul>` adds list semantics; `role="list"` preserves them in Safari when `list-style` is stripped via CSS.
- Each item: `<li>` > `<a href="...">` (RAC `<Link>`) for SPA routing.

### `aria-current`
- Active item: `aria-current="page"` on the `<a>`.
- Inactive items: omit `aria-current` entirely — do not set `"false"`.

### Keyboard
- Tab moves focus in DOM order across items.
- No roving focus — each item is independently tabbable because they are links, not toolbar buttons.
- Enter / Space activates the focused link (browser native on `<a>`).
- No arrow-key navigation — arrow keys belong to `tablist` / `toolbar` patterns; links use Tab.

### Focus ring
Visible `outline` on the `<a>` element using `focus-visible`. Ring color: `--secondary` (or follow system default; do not suppress). Focus ring sits outside `__icon-container` so it is clearly visible and not clipped.

### Badge integration
`<Badge>` is anchored to `__icon-container` via absolute positioning. The item's accessible name must incorporate the badge count: set `aria-label="Inbox, 5 unread"` on the `<a>` when a badge is present. The `badge` prop triggers this concatenation automatically.

### Reduced motion
`@media (prefers-reduced-motion: reduce)`: disable indicator pill `width`/`transform` transition; indicator appears/disappears instantly.

### RTL
- Logical properties throughout (`padding-inline`, `margin-inline`).
- Item order is not reversed for RTL — destination order is language-independent in M3.
- Directional icons (back/forward) handled by `<Icon rtlMirror>`.

---

## Proposed kafUI React API

```tsx
// React Aria base: RAC <Link> from react-aria-components for each item.
// Root <nav> is a plain semantic element. No RAC composite for the bar itself.

interface NavigationBarProps {
  /** Accessible label for the <nav> landmark. Default: "Main navigation" */
  "aria-label"?: string;
  /** "standard" (default) | "icon-only" */
  variant?: "standard" | "icon-only";
  children: React.ReactNode; // <NavigationBar.Item> elements (3–5)
  className?: string;
}

interface NavigationBarItemProps {
  /** Route href — required to enforce link semantics */
  href: string;
  /** Icon sprite name */
  icon: string;
  /** Destination label — always required; used as aria-label in icon-only mode */
  label: string;
  /** Mark as current destination. Sets aria-current="page". Router-driven; no internal state. */
  isCurrent?: boolean;
  /** Optional badge; pass a <Badge> element. Triggers accessible name update. */
  badge?: React.ReactNode;
  isDisabled?: boolean;
  onPress?: (e: PressEvent) => void;
  className?: string;
}

// Compound usage:
<NavigationBar aria-label="Main navigation">
  <NavigationBar.Item href="/" icon="home" label="Home" isCurrent />
  <NavigationBar.Item href="/explore" icon="search" label="Explore" badge={<Badge count={3} />} />
  <NavigationBar.Item href="/library" icon="library_music" label="Library" />
  <NavigationBar.Item href="/profile" icon="person" label="Profile" />
</NavigationBar>
```

**BEM classes emitted (unprefixed, inside `@layer kafui`):**
- Root `<nav>`: `.navigation-bar`
- `<ul>`: `.navigation-bar__items`
- `<li>`: `.navigation-bar__item`
- `<a>` / RAC `<Link>`: no dedicated BEM class — receives RAC `data-*` attributes
- Icon container `<span>`: `.navigation-bar__icon-container`
- Indicator `<span>`: `.navigation-bar__indicator`
- Icon `<span>`: `.navigation-bar__icon`
- Badge wrapper `<span>`: `.navigation-bar__badge`
- Label `<span>`: `.navigation-bar__label`
- Active modifier on `<li>`: `.navigation-bar__item--active` (when `isCurrent`)
- Variant modifier on root: `.navigation-bar--icon-only`

RAC `data-hovered`, `data-focused`, `data-pressed`, `data-disabled` from `<Link>` drive CSS state-layer rules on `__icon-container`.

**React Aria base:** `Link` from `react-aria-components` for each item. Provides pointer/keyboard press handling, `data-focused`, `data-hovered`, `data-pressed`, `data-disabled`, focus-visible management, and SPA router integration via `RouterProvider`. Do NOT use RAC `Tabs`.

**API decisions and justifications:**
- `isCurrent` (not `active` or `isSelected`) matches React Aria naming and maps directly to `aria-current="page"`.
- `href` is required (not optional) to enforce link semantics. For SPA routers, consumers pass the RAC `RouterProvider`; no proprietary `component` prop needed.
- No `value`/`onChange` — active state is router-driven, not internal controlled state.
- `label` is always required even in `"icon-only"` variant: used for `aria-label` on the `<a>` when label text is visually hidden.
- `badge` prop is `React.ReactNode` (not a count number) to allow `<Badge>` customization; the item automatically builds an accessible name string when a badge with a `count` prop is detected.
