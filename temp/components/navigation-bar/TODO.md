# Navigation Bar — TODO

## MUI Equivalent

**`BottomNavigation`** + **`BottomNavigationAction`** (`@mui/material`).

MUI's `BottomNavigation` uses a controlled `value`/`onChange` selection model, renders `<button>` elements (not links), and applies `aria-selected` + an internal tab-like pattern — semantically wrong for a navigation landmark. There is no active-indicator pill, no `aria-current`, no `<nav>` element, no badge slot, and no M3 motion tokens. Every gap below is a concrete opportunity to beat MUI.

---

## Beat-MUI Opportunities

### 1. Correct semantics — `<nav>` + `aria-current` vs MUI's button+aria-selected

MUI renders `<div>` + `<button>` with `aria-selected`, implying a tab widget to screen readers. kafUI renders `<nav>` + `<a>` + `aria-current="page"` per WAI-ARIA Navigation pattern.

**Tasks:**
- [ ] Root: `<nav aria-label={ariaLabel ?? "Main navigation"}>` wrapping `<ul role="list">`
- [ ] Each item: `<li>` > RAC `<Link href={href}>` with `aria-current={isCurrent ? "page" : undefined}`
- [ ] Inactive items: omit `aria-current` entirely (do NOT set `"false"`)
- [ ] Screen-reader test: VoiceOver/NVDA announces "navigation" landmark + items as links + active item as "current page"

### 2. M3-faithful active-indicator pill — MUI has none

MUI has no indicator pill. kafUI ships the full spec: 64×32 dp pill, `--secondary-container` fill, animated expand/contract, label size shift on active.

**Tasks:**
- [ ] `__indicator` always in DOM; `transform: scaleX(0)` (or `width: 0`) inactive; `transform: scaleX(1)` / `width: var(--indicator-w)` active
- [ ] Transition: `transform` (or `width`) using `--duration-short2` + `--easing-emphasized`; prefer `transform: scaleX()` over `width` to avoid reflow
- [ ] Indicator fill: `--secondary-container`; shape: `border-radius: var(--corner-full)`
- [ ] Active icon color: `--on-secondary-container`; inactive: `--on-surface-variant`
- [ ] Active label: `--label-large-font` + `--on-surface`; inactive: `--label-medium-font` + `--on-surface-variant`
- [ ] `@media (prefers-reduced-motion: reduce)`: disable transition; instant toggle

### 3. CSS-first state layers — MUI's JS ripple vs kafUI's zero-JS tinting

MUI's `Ripple` component injects JS-driven keyframe animations per interaction. kafUI uses a CSS `::after` pseudo-element on `__icon-container` with `opacity` driven by RAC `data-*` attributes — no JS overhead.

**Tasks:**
- [ ] `__icon-container`: `position: relative; overflow: hidden; border-radius: var(--corner-full)`
- [ ] State layer on `__icon-container::after` (or shared `.state-layer` class): `content: ""; position: absolute; inset: 0; background: currentColor; opacity: 0; transition: opacity 0.15s`
- [ ] `[data-hovered] .navigation-bar__icon-container::after { opacity: var(--state-hover); }` — 0.08 inactive, `--on-surface`; active uses `--on-secondary-container` via `currentColor`
- [ ] `[data-focused] … { opacity: var(--state-focus); }` — 0.10
- [ ] `[data-pressed] … { opacity: var(--state-pressed); }` — 0.10
- [ ] State layer scoped to `__icon-container` only — label does NOT get tinted

### 4. First-class badge slot — MUI requires manual composition

MUI has no `badge` slot on `BottomNavigationAction`; consumers must wrap children manually with no accessible-name integration. kafUI's `badge` prop automatically updates the `<a>` accessible name.

**Tasks:**
- [ ] `badge` prop: render inside `__badge` wrapper (absolutely positioned over `__icon-container`)
- [ ] When `badge` is a `<Badge count={n}>`, append `, ${n} unread` to the `<a aria-label>` automatically
- [ ] `"icon-only"` variant: `aria-label="${label}, ${badgeCount} unread"` on `<a>`; label span visually hidden with `visibility: hidden` + `height: 0` (not `display: none` — preserves layout)

### 5. Single-source theming — MUI requires per-component palette overrides

MUI requires `theme.palette.primary.main` + `MuiBottomNavigationAction.styleOverrides` for any color change. kafUI: setting `--source` at `:root` re-derives all tokens automatically.

**Tasks:**
- [ ] Bar background: `background: var(--surface-container)` (the `surface-container` role)
- [ ] Elevation: apply `--elevation-2` (tonal tint + shadow) to bar surface
- [ ] Verify dark mode: `color-scheme: dark` on `:root` flips all `light-dark()` roles with zero component changes
- [ ] RTL: all properties use `padding-inline`, `margin-inline`, `inset-inline-*` — `dir="rtl"` needs no extra styles

### 6. `"icon-only"` variant accessibility — MUI ignores this entirely

MUI does not have an icon-only mode; consumers just omit label text. kafUI's `"icon-only"` variant hides the label visually but preserves it as the accessible name of the link.

**Tasks:**
- [ ] `.navigation-bar--icon-only .navigation-bar__label`: visually hidden (not `display: none`) so it still contributes to layout but is off-screen
- [ ] `<a aria-label={label}>` when `variant="icon-only"` (RAC `Link`'s accessible name falls back to `aria-label`)
- [ ] Adjust `__icon-container` vertical centering when label is hidden

### 7. Item count enforcement (3–5) — M3 constraint, MUI ignores

M3 specifies 3–5 destinations. kafUI should warn in dev when constraint is violated.

**Tasks:**
- [ ] In `DEV`, `console.warn` if `children` count is < 3 or > 5
- [ ] CSS: `__items` uses `flex: 1` on each `__item` — auto-distributes equal widths for 3, 4, or 5 items; `min-width: var(--item-min-w)` (48px per M3 touch target)

---

## Styles (`@kafui/styles`)

- [ ] Create `src/components/navigation-bar/_navigation-bar.css` (plain CSS inside `@layer kafui { }`)
- [ ] `.navigation-bar`: `position: fixed; inset-inline: 0; inset-block-end: 0; height: var(--h); background: var(--surface-container); display: flex; align-items: stretch`
- [ ] `.navigation-bar__items`: `display: flex; flex-direction: row; width: 100%; list-style: none; padding: 0; margin: 0`
- [ ] `.navigation-bar__item`: `flex: 1; min-width: var(--item-min-w); display: flex` (positions `<a>` to fill)
- [ ] `<a>` child of `__item`: `display: flex; flex-direction: column; align-items: center; justify-content: center; width: 100%; text-decoration: none`
- [ ] Indicator transition via `transform: scaleX()` (not `width`) to avoid reflow

## React (`@kafui/react`)

- [ ] Create `src/components/NavigationBar/NavigationBar.tsx`
- [ ] `NavigationBar.Item`: `<li className="navigation-bar__item ...">` > RAC `<Link href={href} aria-current={isCurrent ? "page" : undefined} aria-label={iconOnly ? label : undefined}>`
- [ ] `isCurrent` → `.navigation-bar__item--active` on `<li>` + `aria-current="page"` on `<a>`
- [ ] Wire RAC `RouterProvider` compatibility
- [ ] `isDisabled` → `aria-disabled="true"` + `.navigation-bar__item--disabled`; prevent press
- [ ] Export `NavigationBar`, `NavigationBar.Item`

## Tests

- [ ] Renders `<nav>` landmark with `aria-label`
- [ ] Active item `<a>` has `aria-current="page"`; inactive items have no `aria-current` attribute
- [ ] Clicking inactive item fires `onPress`; does NOT set internal active state
- [ ] `badge` prop renders inside `__badge`; accessible name includes count
- [ ] `"icon-only"`: label visually hidden; `aria-label` on link equals `label` prop
- [ ] `isDisabled`: `aria-disabled="true"`, no press events
- [ ] Tab order: all 3–5 items reachable via Tab (no roving focus)
- [ ] Dev warning fires when < 3 or > 5 items

## QA

- [ ] Indicator pill animation: 60 fps, no layout shift; `transform: scaleX()` preferred over `width`
- [ ] Label font shift: `--label-medium-*` → `--label-large-*` on active; no layout jump
- [ ] Badge overflow: count 999+ renders `999+` within indicator
- [ ] RTL: `dir="rtl"` — logical properties correct; no direction-specific CSS needed
- [ ] 3, 4, 5 items: equal-width distribution verified
- [ ] Screen reader: "navigation" landmark; items as links; active item "current page"
- [ ] Dark mode: `color-scheme: dark` — all colors flip correctly via `light-dark()`
