# Navigation Drawer

**Purpose:** A side panel housing the full set of navigation destinations. Supports grouped destinations with section headlines and dividers, an optional header for branding/account, and optional trailing badges. Used at medium-to-large window classes (expanded, some medium). Modal variant is used on compact/medium when triggered from a menu button.  
**M3 category:** Navigation → Navigation Drawer.

---

## Anatomy / Parts → BEM Elements

```
.navigation-drawer                     root; contains header + scrollable content
  .navigation-drawer__header           optional branding / account area (free-form slot)
  .navigation-drawer__content          overflow-y: auto; holds sections + items
    .navigation-drawer__section        logical group of destinations (maps to <section> or <ul>)
      .navigation-drawer__headline     section header text (<h2> or <h3>)
      .navigation-drawer__divider      <hr role="separator"> above section
      .navigation-drawer__item         <li> wrapper for one destination
        <a> / RAC <Link>               interactive element; carries aria-current
          .navigation-drawer__item-indicator  full-row pill (active fill; always in DOM)
          .navigation-drawer__item-icon       leading <Icon> sprite (24 dp, optional)
          .navigation-drawer__item-label      primary destination text
          .navigation-drawer__item-badge      optional <Badge> at inline-end
```

The `__item-indicator` spans the full width of the item row — this is the key visual distinction from Navigation Rail (icon-centered pill) and Navigation Bar (64 dp centered pill). It sits behind the icon + label as an absolutely-positioned background.

> **Shared indicator pattern:** Active indicator (pill shape, `--secondary-container` fill) is shared across Navigation Bar, Rail, and Drawer — only the width differs. See Cross-Component Notes.

---

## Variants

| Variant | `variant` prop | Description |
|---------|---------------|-------------|
| Standard | `"standard"` | Persistent; always visible; content is pushed aside or overlapped by layout; no scrim; no close gesture |
| Modal | `"modal"` | Overlays content from `inline-start`; scrim covers remaining screen; dismissible via Escape, scrim tap, or close button; focus trapped |

A **dismissible** variant (persistent but collapsible to 0 width) is not in the current M3 spec; it may be added as an extension in a future release. Do not implement v1.

---

## States

### Drawer-level

| State | Standard | Modal |
|-------|----------|-------|
| Open | Always visible | `isOpen={true}` — slide in from `inline-start` |
| Closed | N/A | `isOpen={false}` — slide out to `inline-start`; scrim hidden; focus returns to trigger |

### Item states

| State | Visual |
|-------|--------|
| Inactive | Label + icon `--on-surface-variant`; `__item-indicator` background transparent |
| Active | `__item-indicator` filled `--secondary-container`; label + icon `--on-secondary-container`; `aria-current="page"` |
| Hover (inactive) | State-layer 8% `--on-surface` over full item row (`<a>::after` or state-layer element) |
| Hover (active) | State-layer 8% `--on-secondary-container` |
| Focus-visible | State-layer 10%; `outline` on `<a>` |
| Pressed | State-layer 10% |
| Disabled | 38% opacity; `aria-disabled="true"`; no state layer |

State layers on drawer items cover the full row (unlike bar/rail which scope to `__item-indicator` only).

---

## Design Tokens

| Token (unprefixed CSS custom property) | Role |
|----------------------------------------|------|
| `--surface` | drawer background (standard + modal) |
| `--surface-container-low` | alternative drawer background per M3 2024 surface model |
| `--on-surface-variant` | inactive icon + label |
| `--on-surface` | headline text; inactive label |
| `--secondary-container` | active indicator pill fill |
| `--on-secondary-container` | active icon + label color |
| `--scrim` | modal scrim (`rgba(0,0,0,0.5)` or M3 scrim color) |
| `--elevation-1` | modal variant shadow (standard has no elevation) |
| `--corner-full` | active indicator pill fully rounded ends |
| `--corner-large` | drawer container shape (inline-end corners only, modal) |
| `--label-large-size`, `--label-large-weight`, `--label-large-font` | destination label |
| `--title-small-size`, `--title-small-weight`, `--title-small-font` | section headline |
| `--outline-variant` | divider color |
| `--duration-medium2` | modal open/close transition |
| `--easing-emphasized-decelerate` | modal open easing |
| `--easing-emphasized-accelerate` | modal close easing |
| `--state-hover` | 0.08 |
| `--state-focus` | 0.10 |
| `--state-pressed` | 0.10 |

Component-internal vars (scoped inside `.navigation-drawer { … }`):

```css
@layer kafui {
  .navigation-drawer {
    --w: 360px;             /* M3 spec drawer width */
    --item-h: 56px;         /* destination item height */
    --item-px: 12px 24px;   /* padding-inline: start end */
    --indicator-inset: 4px 12px; /* inset within item row */
  }
}
```

---

## Interaction & Accessibility

### Why `<nav>` + links, not Tabs or Listbox

Drawer items are navigation destinations — they navigate between routes, not switch content panels or select options. `role="tablist"` requires `aria-controls` to associated panels; `role="listbox"` implies option selection. `<nav>` + `<a>` + `aria-current="page"` is correct per WAI-ARIA Navigation pattern.

### Landmark & Structure
- Root: The `<nav>` landmark wraps the content area (and optionally the header if the header contains navigation-relevant content). When `header` is free-form brand content, place `<nav>` around `__content` only.
- Sections with headlines: `<section aria-labelledby="headline-id">` + `<h2 id="headline-id">` (or `<h3>` depending on page heading hierarchy).
- Sections without headlines: plain `<ul role="list">`.
- Dividers: `<hr>` (implicit `role="separator"`).
- Items: `<li>` > `<a href="...">` (RAC `<Link>`) with `aria-current="page"` on active item.

### Modal variant — focus management
- On open: focus moves to the first focusable element in the drawer (or the close button if present).
- Focus is **trapped** within the drawer while open: Tab cycles through focusable elements; Shift+Tab wraps backward.
- On close: focus returns to the trigger element that opened the drawer.
- Implementation: RAC `<ModalOverlay>` + `<Modal>` + `<Dialog>` — provides `aria-modal="true"`, focus trap (`FocusScope`), Escape key handling, and scrim/overlay.
- The `<Dialog>` gets `aria-label` matching the `<nav>` accessible label to provide context when announced as a dialog.

### Standard variant — no focus trap
- No focus trap; Tab exits the drawer into main content naturally.
- Consider a skip link before the drawer for keyboard users.

### `aria-current`
Active destination: `aria-current="page"`. Inactive items: omit the attribute entirely.

### Keyboard (items)
- Tab traverses all focusable items in DOM order.
- Enter / Space activates a focused link.
- No arrow-key roving focus — links, not a composite widget.

### Focus ring
Visible `outline` on `<a>` using `focus-visible`. Ring color: `--secondary`.

### RTL slide animation
The modal slide animation must not rely on physical `translateX`. Instead:
- Use `translate: -100% 0` (closed) → `translate: 0 0` (open) combined with `direction: ltr` on the drawer itself, OR
- Use `@starting-style` + `transition` on `inset-inline-start` (CSS 2023 `@starting-style`), OR
- Target `[dir="rtl"] .navigation-drawer--modal` with `translate: 100% 0` for the closed state.

The simplest robust approach: `translate` physical value flipped via `[dir="rtl"]` selector — no logical equivalent for `translate` exists yet.

### Reduced motion
`@media (prefers-reduced-motion: reduce)`: skip `translate` transition; use `visibility` toggle for instant open/close.

---

## Proposed kafUI React API

```tsx
// React Aria:
//   Modal variant: RAC <ModalOverlay> + <Modal> + <Dialog> (focus trap, Escape, aria-modal)
//   Standard variant: plain <nav> — no RAC composite

interface NavigationDrawerProps {
  /** "standard" (default) | "modal" */
  variant?: "standard" | "modal";
  /** Controls open state. Modal: required. Standard: always open, prop ignored. */
  isOpen?: boolean;
  /** Called when modal drawer requests close (Escape, scrim tap, close button) */
  onClose?: () => void;
  /** Accessible label for the <nav> landmark (and <dialog> in modal). Default: "Application navigation" */
  "aria-label"?: string;
  /**
   * Optional header slot (account card, branding, etc.).
   * Rendered above __content, outside the scroll area.
   * Not inside the <nav> — caller's responsibility to add nav-relevant content to children instead.
   */
  header?: React.ReactNode;
  children: React.ReactNode; // <NavigationDrawer.Section> or <NavigationDrawer.Item>
  className?: string;
}

interface NavigationDrawerSectionProps {
  /** Optional section headline text; renders as <h2> */
  headline?: string;
  /** Render a <hr> divider above this section */
  divider?: boolean;
  children: React.ReactNode; // <NavigationDrawer.Item> elements
}

interface NavigationDrawerItemProps {
  /** Route href — required */
  href: string;
  /** Optional leading icon sprite name */
  icon?: string;
  /** Destination label — required */
  label: string;
  /** Current destination → aria-current="page" + active styles */
  isCurrent?: boolean;
  /** Optional trailing badge element */
  badge?: React.ReactNode;
  isDisabled?: boolean;
  onPress?: (e: PressEvent) => void;
  className?: string;
}

// Modal usage:
<NavigationDrawer variant="modal" isOpen={drawerOpen} onClose={() => setDrawerOpen(false)}>
  <NavigationDrawer.Item href="/" icon="home" label="Home" isCurrent />
  <NavigationDrawer.Section headline="Library" divider>
    <NavigationDrawer.Item href="/playlists" icon="queue_music" label="Playlists" />
    <NavigationDrawer.Item href="/albums" icon="album" label="Albums" />
  </NavigationDrawer.Section>
</NavigationDrawer>

// Standard usage:
<NavigationDrawer header={<AccountCard />}>
  <NavigationDrawer.Item href="/" icon="home" label="Home" isCurrent />
</NavigationDrawer>
```

**BEM classes emitted (unprefixed, inside `@layer kafui`):**
- Root wrapper: `.navigation-drawer`, `.navigation-drawer--standard` / `--modal`, `.navigation-drawer--open`
- Header `<div>`: `.navigation-drawer__header`
- Content `<div>` (scrollable): `.navigation-drawer__content`
- Section `<section>` or `<div>`: `.navigation-drawer__section`
- Headline `<h2>`: `.navigation-drawer__headline`
- Divider `<hr>`: `.navigation-drawer__divider`
- Item `<li>`: `.navigation-drawer__item`, `.navigation-drawer__item--active`
- Item indicator `<span>` (behind icon+label): `.navigation-drawer__item-indicator`
- Item icon `<span>`: `.navigation-drawer__item-icon`
- Item label `<span>`: `.navigation-drawer__item-label`
- Item badge `<span>`: `.navigation-drawer__item-badge`
- Scrim (modal only, rendered inside `<ModalOverlay>`): `.navigation-drawer__scrim`

**React Aria base:**
- Modal variant: RAC `<ModalOverlay>` + `<Modal>` + `<Dialog>` — provides `aria-modal`, `FocusScope`, Escape key, `onOpenChange`.
- Standard variant: plain `<nav>` + `<ul>` + RAC `<Link>` per item.
- Each item: RAC `<Link>` for pointer/keyboard/focus-visible handling.

**API decisions and justifications:**
- Modal uses RAC `<Dialog>` (not custom focus trap) — battle-tested WAI-ARIA Dialog; avoids reinventing focus management.
- `isOpen` is always caller-controlled — routing state drives standard; trigger state drives modal. No internal state.
- `header` is free-form slot — M3 does not prescribe a fixed header structure.
- No `width` prop — drawer width is `var(--w)`, overridable via CSS at any scope.
- `onClose` (not `onOpenChange(false)`) for ergonomics — callers only need to handle close requests; open is controlled by them.
- Section `headline` renders as `<h2>` by default; consumers can override heading level via a future `headingLevel` prop if page hierarchy demands it.
