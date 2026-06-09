# Tabs

Tabs organise content into multiple sections and let users switch between them. Only one section is visible at a time. M3 category: **Navigation → Tabs**.

---

## Anatomy / parts

| BEM element | Description |
|---|---|
| `.tabs` | Root container; wraps tab row and panels |
| `.tabs__list` | Horizontal row of tab items (`role="tablist"`) |
| `.tabs__indicator` | Animated active indicator bar that slides under the active tab; absolutely-positioned child of `.tabs__list` |
| `.tabs__scroller` | Scroll container (scrollable variant); clips the tab row and enables `overflow-x: auto` |
| `.tabs__scroll-button` | Leading/trailing scroll arrow button (scrollable variant) |
| `.tabs__tab` | Individual tab item (`role="tab"`) |
| `.tabs__tab-icon` | Optional icon slot — `<Icon>` sprite wrapper, 24 dp |
| `.tabs__tab-label` | Tab label text — `--label-large-*` (primary) or `--title-small-*` (secondary) |
| `.tabs__tab-badge` | Optional badge element overlaid on icon |
| `.tabs__panel` | Content panel (`role="tabpanel"`); only the active panel is visible |

State layer is handled via `::after` pseudo-element on `.tabs__tab` (not a separate child element): `content: ""; position: absolute; inset: 0; background: currentColor; opacity: 0; transition: opacity 0.15s`.

---

## Variants

### Type (`variant` prop)

| Variant | Description |
|---|---|
| `primary` | Default. Full-width active indicator (3 dp high, `--corner-full` top corners) at bottom of tab bar. Label uses `--label-large-*`. Active tab color: `--primary`. Inactive: `--on-surface-variant`. |
| `secondary` | Subtler. Narrow pill indicator under content only (2 dp, `--corner-full`). Label uses `--title-small-*`. Active: `--on-surface`. Inactive: `--on-surface-variant`. |

### Layout (`layout` prop)

| Layout | Description |
|---|---|
| `fixed` | All tabs share equal width, filling the container (`flex: 1` per tab). Best for 2–5 tabs. |
| `scrollable` | Tabs take natural content width; row scrolls horizontally when overflowing. Best for 5+ tabs. Optional leading/trailing scroll buttons. |

### Content type

Tabs may contain label only (most common), icon + label, or icon only (rare; requires `aria-label` on each tab).

BEM modifiers on `.tabs__tab`: `.tabs__tab--icon-only`, `.tabs__tab--icon-top` (icon above label), `.tabs__tab--icon-inline` (icon inline-start of label).

---

## States

| State | Tab item visual |
|---|---|
| Active (selected) | Indicator fully visible; label/icon in active color (`--primary` or `--on-surface` per variant); `aria-selected="true"` |
| Inactive (unselected) | Indicator hidden; label/icon in `--on-surface-variant`; `aria-selected="false"` |
| Hovered | State-layer `::after` opacity `var(--state-hover)` (0.08) |
| Focus-visible | State-layer `::after` opacity `var(--state-focus)` (0.10); `outline` on `.tabs__tab` |
| Pressed | State-layer `::after` opacity `var(--state-pressed)` (0.10) |
| Disabled | Color `color-mix(in oklch, var(--on-surface) 38%, transparent)`; no state layer; `aria-disabled="true"` |

The indicator slides between active tab positions when selection changes — animated via `transform: translateX()` + `width` transition.

---

## Design tokens

### Color
- Primary active label/icon: `--primary`
- Primary indicator: `--primary`
- Secondary active label/icon: `--on-surface`
- Secondary indicator: `--on-surface`
- Inactive label/icon (both variants): `--on-surface-variant`
- Tab bar surface: inherits from parent; no own background
- State-layer color: `currentColor` (inherits active or inactive label color)

### Shape
- Primary indicator: rectangle, full-width of tab, `3px` high, `border-start-start-radius: var(--corner-full); border-start-end-radius: var(--corner-full)` (pill on top corners only)
- Secondary indicator: pill shape `2px` high, `border-radius: var(--corner-full)`, width matches inner content (icon + label)

### Typography
- Primary tab label: `font: var(--label-large-font); font-size: var(--label-large-size); font-weight: var(--label-large-weight); line-height: var(--label-large-line-height)`
- Secondary tab label: `font: var(--title-small-font); font-size: var(--title-small-size); font-weight: var(--title-small-weight); line-height: var(--title-small-line-height)`

### Elevation
- Tab bar: `--elevation-0` (flat); inherits parent surface. No shadow.

### Motion
- Indicator slide + width: `transform: translateX()` + `width`; easing `--easing-emphasized`, duration `--duration-medium2` (~300 ms)
- Optional panel cross-fade: `opacity` transition, easing `--easing-standard`, duration `--duration-short4`
- `@media (prefers-reduced-motion: reduce)`: indicator position updates instantly; no panel cross-fade

### State layer
- `--state-hover`: 0.08
- `--state-focus`: 0.10
- `--state-pressed`: 0.10

Component-internal vars (scoped inside `.tabs { … }`):

```css
@layer kafui {
  .tabs {
    --tab-min-h: 48px;
    --tab-px: 16px;
    --indicator-h-primary: 3px;
    --indicator-h-secondary: 2px;
  }
}
```

Indicator position is driven by CSS custom properties set via `useLayoutEffect`:

```css
.tabs__indicator {
  transform: translateX(var(--indicator-x, 0));
  width: var(--indicator-w, 0);
}
```

---

## Interaction & accessibility

**ARIA roles:**
- `.tabs` root: no required role (not a `<nav>`; tabs switch content panels, not routes)
- `.tabs__list`: `role="tablist"`, `aria-label` required (describes the tab set, e.g. "Account sections")
- Each `.tabs__tab`: `role="tab"`, `aria-selected="true|false"`, `aria-controls="<panel-id>"`, unique `id`
- Each `.tabs__panel`: `role="tabpanel"`, `aria-labelledby="<tab-id>"`, unique `id`; `tabindex="0"` (panel is focusable); inactive panels: `hidden` attribute

> **Nav-bar vs Tabs distinction:** Navigation bar/rail/drawer items are links (`<a>` + `aria-current="page"`) because they navigate between routes. Tabs are `role="tab"` + `role="tabpanel"` because they switch in-page content sections. Never conflate these patterns. The tabs root is NOT a `<nav>`.

**Keyboard navigation:**
- `Tab` key enters the tab list; focuses the active tab (the only tab with `tabindex="0"`)
- Further `Tab` moves focus from the tab list to the active panel (`tabindex="0"` on panel)
- `←` / `→` arrow keys move focus between tabs (roving `tabindex` — focused tab = `tabindex="0"`, others = `tabindex="-1"`)
- `Home` / `End` jump to first / last enabled tab
- Selection model:
  - **Automatic** (default): arrow-key focus immediately activates the tab and shows its panel
  - **Manual**: focus and activation are decoupled; `Enter` or `Space` activates the focused tab. Prefer manual for async-loaded panels to avoid triggering data fetches on arrow key navigation.
- Disabled tabs: skipped in roving tabindex; `tabindex="-1"`, `aria-disabled="true"`

**Scroll buttons (scrollable layout):**
- `role="button"`, `aria-label="Scroll tabs left"` / `"Scroll tabs right"` (or localized equivalent via `I18nProvider`)
- Hidden (`aria-hidden="true"`, `visibility: hidden`) when no overflow exists
- RAC `<Button>` with `onPress`

**RTL:**
- Arrow key behavior inverts (`←` → next tab, `→` → previous tab) — RAC handles this automatically.
- CSS `flex-direction: row` on `.tabs__list` + logical inline properties; visual order mirrors under `dir="rtl"`.
- Indicator `translateX` direction reverses: under `[dir="rtl"]`, the offset is measured from the right. The `useTabIndicator` hook must account for direction when computing `--indicator-x`.

**Reduced motion:** Indicator jumps instantly (no `transform` transition); panel swap is instantaneous.

---

## Proposed kafUI React API

```tsx
// react-aria-components base: Tabs, TabList, Tab, TabPanel

type TabsVariant = 'primary' | 'secondary';
type TabsLayout = 'fixed' | 'scrollable';

interface TabsProps {
  variant?: TabsVariant;              // default: 'primary'
  layout?: TabsLayout;                // default: 'fixed'
  selectedKey?: Key;
  defaultSelectedKey?: Key;
  onSelectionChange?: (key: Key) => void;
  /**
   * 'automatic': arrow key = immediate activation (default).
   * 'manual': Enter/Space activates; use for async panels.
   * Maps to RAC keyboardActivation prop.
   */
  keyboardActivation?: 'automatic' | 'manual';
  className?: string;
  children: React.ReactNode; // Tabs.List + Tabs.Panel children
}

interface TabListProps {
  'aria-label': string; // required — describes the tab set
  children: React.ReactNode;
  className?: string;
}

interface TabProps {
  id: Key;        // required; matched to Tabs.Panel id
  isDisabled?: boolean;
  icon?: string;  // Icon sprite name
  iconPosition?: 'top' | 'inline'; // default: 'top' when icon+label
  badge?: React.ReactNode;
  children: React.ReactNode; // label text (or icon-only with aria-label on Tab)
  className?: string;
}

interface TabPanelProps {
  id: Key; // must match corresponding Tab id
  children: React.ReactNode;
  className?: string;
}

// Usage:
<Tabs variant="primary" layout="fixed" defaultSelectedKey="home">
  <Tabs.List aria-label="Account sections">
    <Tabs.Tab id="home">Home</Tabs.Tab>
    <Tabs.Tab id="activity">Activity</Tabs.Tab>
    <Tabs.Tab id="profile" icon="person" iconPosition="top">Profile</Tabs.Tab>
  </Tabs.List>
  <Tabs.Panel id="home">…</Tabs.Panel>
  <Tabs.Panel id="activity">…</Tabs.Panel>
  <Tabs.Panel id="profile">…</Tabs.Panel>
</Tabs>
```

**BEM classes emitted (unprefixed, inside `@layer kafui`):**
- Root: `.tabs`, `.tabs--primary` / `.tabs--secondary`, `.tabs--fixed` / `.tabs--scrollable`
- Tab list: `.tabs__list`
- Indicator: `.tabs__indicator` (child of `.tabs__list`, absolutely positioned)
- Scroller wrapper: `.tabs__scroller` (scrollable layout only)
- Scroll button: `.tabs__scroll-button`
- Tab item: `.tabs__tab`, `.tabs__tab--icon-only`, `.tabs__tab--icon-top`, `.tabs__tab--icon-inline`
- Tab icon: `.tabs__tab-icon`
- Tab label: `.tabs__tab-label`
- Tab badge: `.tabs__tab-badge`
- Panel: `.tabs__panel`

RAC `data-selected`, `data-hovered`, `data-focused`, `data-pressed`, `data-disabled` drive all CSS state rules. No additional BEM modifier classes for hover/focus/pressed — CSS attribute selectors handle these.

**React Aria base:** `Tabs`, `TabList`, `Tab`, `TabPanel` from `react-aria-components`. Provides `tablist`/`tab`/`tabpanel` roles, `aria-selected`, `aria-controls`/`aria-labelledby` pairing, roving `tabindex`, arrow-key + Home/End navigation, `keyboardActivation` mode.

**Indicator positioning:** `useTabIndicator` hook on `Tabs.List`:
- `useLayoutEffect` reads `getBoundingClientRect()` of the active `.tabs__tab` relative to `.tabs__list`.
- Sets `--indicator-x` and `--indicator-w` CSS custom properties on the `.tabs__list` element.
- For `secondary` variant: measures inner content container (`__tab-label` + `__tab-icon`) width instead of full tab width.
- For `[dir="rtl"]`: computes offset from the right edge; uses `useLocale()` from RAC to detect direction.
- Fires on mount, on `selectedKey` change, and on resize (via `ResizeObserver`).

**API decisions and justifications:**
- No `indicatorColor` / `textColor` props — colors are determined entirely by `variant` and M3 token roles. MUI's prop-based color overrides are replaced by CSS custom property overrides at any scope.
- No `orientation="vertical"` — M3 Tabs are horizontal only. Vertical navigation uses Navigation Rail/Drawer.
- `keyboardActivation` (not `activationMode`) — mirrors RAC's prop name directly.
- `activationMode` alias is NOT provided — keep one name, reduce confusion.
- Compound anatomy: `Tabs.List`, `Tabs.Tab`, `Tabs.Panel` — not `TabList`, `Tab`, `TabPanel` as top-level exports. Consumers import one thing (`Tabs`) and access sub-components via dot notation.
