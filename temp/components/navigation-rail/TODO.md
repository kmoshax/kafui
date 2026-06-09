# Navigation Rail — TODO

## MUI Equivalent

**None in MUI core.** MUI ships no Navigation Rail component. The closest approximation is a permanent `<Drawer variant="permanent">` manually narrowed to ~80 px with `<List>` + `<ListItemButton>` inside — but this:
- Uses modal/portal drawer infrastructure for what is a persistent, non-overlaying surface.
- Has no spec-defined header zone for FAB + menu button.
- Provides no active indicator pill; active state requires manual `selected` prop styling.
- Has no Expressive expandable behavior.
- Uses `anchor="left"` / `anchor="right"` (physical) instead of logical `inset-inline-start`.

This entire component is a kafUI differentiator. Every TODO item is an opportunity to ship something MUI cannot.

---

## Beat-MUI Opportunities

### 1. First Navigation Rail in a major React component library

No popular React UI library ships a spec-faithful Navigation Rail. kafUI ships the first, with indicator pill, header zone, badge, and Expressive expand/collapse out of the box.

**Tasks:**
- [ ] Create `src/components/NavigationRail/NavigationRail.tsx`
- [ ] Root: `<nav aria-label={ariaLabel ?? "Application navigation"}>` with `__header` + `__destinations`
- [ ] Document prominently: "No MUI equivalent — kafUI ships first-class M3 Navigation Rail"
- [ ] Dev warning if < 3 or > 7 items

### 2. Spec-faithful indicator pill at 56 dp — MUI has no pill

MUI's `ListItemButton selected` changes background on the full row with a custom theme. kafUI ships the 56 dp centered pill with `--secondary-container` fill and animated expansion.

**Tasks:**
- [ ] `.navigation-rail__item-indicator`: `width: var(--indicator-w); height: var(--indicator-h); border-radius: var(--corner-full); background: transparent; display: flex; align-items: center; justify-content: center; position: relative; overflow: hidden`
- [ ] Active: `background: var(--secondary-container)` via `[aria-current="page"] .navigation-rail__item-indicator` or `__item--active .navigation-rail__item-indicator`
- [ ] Indicator animate: `transition: background var(--duration-short2) var(--easing-emphasized)` (background color fade, not width — the pill is always 56 dp in collapsed mode)
- [ ] Icon inactive: `color: var(--on-surface-variant)`; active: `color: var(--on-secondary-container)`

### 3. Expressive expand/collapse with M3 motion — MUI cannot approximate this

M3 Expressive adds an animated rail that morphs from icon-only (80 dp) to icon+label (256 dp). No MUI primitive supports this.

**Tasks:**
- [ ] `.navigation-rail--expandable`: `width: var(--rail-w); transition: width var(--duration-medium3) var(--easing-emphasized)`
- [ ] `.navigation-rail--expandable.navigation-rail--expanded`: `width: var(--rail-w-expanded)`
- [ ] In collapsed mode: `__item` is `flex-direction: column; align-items: center`; label below icon
- [ ] In expanded mode (`--expanded` modifier): `__item` switches to `flex-direction: row`; `__destinations` left-aligns items; `__item-indicator` expands to full row width (matches drawer indicator pattern)
- [ ] Expand easing: use `--easing-emphasized-decelerate` for open, `--easing-emphasized-accelerate` for close — implement via CSS `transition-timing-function` swapped on modifier class
- [ ] `@media (prefers-reduced-motion: reduce)`: `transition: none`; instant width change

### 4. FAB integration in header zone — MUI requires manual positioning

M3 spec places a FAB above destinations in the rail header. MUI offers no such concept.

**Tasks:**
- [ ] `fab` prop: render provided `<Fab>` node inside `.navigation-rail__fab` slot
- [ ] `__header`: `display: flex; flex-direction: column; align-items: center; padding-block-start: 12px; gap: 4px`
- [ ] FAB slot is below menu button; above destinations
- [ ] When `variant="expandable"` and `isExpanded={true}`, FAB expands to Extended FAB (label appears inline) — document as optional enhancement; mark as future TODO if not in v1

### 5. Logical-property RTL — MUI uses physical `anchor` prop

MUI's `Drawer` requires `anchor="right"` for RTL — a runtime prop change. kafUI uses `inset-inline-start: 0` which auto-flips under `dir="rtl"` with zero code change.

**Tasks:**
- [ ] Root: `position: fixed; inset-block: 0; inset-inline-start: 0; width: var(--rail-w); background: var(--surface); display: flex; flex-direction: column`
- [ ] All spacing: `padding-inline`, `margin-inline`, `padding-block`; no `left`/`right`/`top`/`bottom`
- [ ] Expanded label: `margin-inline-start: 12px` (not `margin-left`)
- [ ] Test: `dir="rtl"` — rail appears at right edge; no JS needed

### 6. CSS-first state layers on `__item-indicator` — MUI uses `Ripple` JS

**Tasks:**
- [ ] State layer via `__item-indicator::after` pseudo-element: `content: ""; position: absolute; inset: 0; border-radius: inherit; background: currentColor; opacity: 0; transition: opacity 0.15s`
- [ ] RAC `data-hovered` on `<Link>` → `opacity: var(--state-hover)` on `::after`
- [ ] RAC `data-focused` → `opacity: var(--state-focus)`
- [ ] RAC `data-pressed` → `opacity: var(--state-pressed)`
- [ ] `currentColor` inherits `--on-surface-variant` inactive / `--on-secondary-container` active — correct tint automatically

### 7. Menu button with correct `aria-expanded` — MUI never wires this

**Tasks:**
- [ ] `menuButton={true}`: RAC `<Button aria-label="Open navigation menu" aria-expanded={menuIsExpanded} onPress={onMenuPress}>` inside `__menu-button`
- [ ] `menuButton={ReactNode}`: render provided node (consumer controls its own aria props)
- [ ] Expandable rail: auto-render toggle button with `aria-expanded={isExpanded}` + `aria-label={isExpanded ? "Collapse navigation" : "Expand navigation"}`
- [ ] Test: `aria-expanded` updates correctly on open/close

### 8. Badge slot on item — no equivalent in MUI approximation

**Tasks:**
- [ ] `badge` prop: render inside `__item-badge` (absolutely positioned on `__item-indicator`)
- [ ] Accessible name: `<a aria-label="${label}, ${badgeCount} unread">` when badge has count
- [ ] Verify badge does not overflow pill boundary in collapsed mode; clip with `overflow: visible` on `__item` but not on `__item-indicator`

---

## Styles (`@kafui/styles`)

- [ ] Create `src/components/navigation-rail/_navigation-rail.css` inside `@layer kafui { }`
- [ ] Collapsed destinations: `align-items: center`; expanded: `align-items: stretch`
- [ ] `__item`: `display: flex; align-items: center; width: 100%; min-height: var(--item-min-h); position: relative; padding-block: 4px; padding-inline: 12px`
- [ ] `__item-label`: `font: var(--label-medium-font) / var(--label-medium-line-height) var(--label-medium-size)`; `color: var(--on-surface-variant)` inactive; active `--on-surface`

## React (`@kafui/react`)

- [ ] `NavigationRail.Item`: `<li className="navigation-rail__item ...">` > RAC `<Link href={href} aria-current={isCurrent ? "page" : undefined}>`
- [ ] `isCurrent` → `.navigation-rail__item--active` + `aria-current="page"`
- [ ] `variant="expandable"` + `isExpanded` → `.navigation-rail--expanded` on root
- [ ] Export `NavigationRail`, `NavigationRail.Item`

## Tests

- [ ] Renders `<nav>` with correct `aria-label`
- [ ] Active item has `aria-current="page"` + `--active` modifier
- [ ] `menuButton={true}`: renders `<button aria-label="Open navigation menu">` with `aria-expanded`
- [ ] `onMenuPress` fires on press
- [ ] `fab` prop: renders FAB in `__fab` slot inside `__header`
- [ ] Expandable: `isExpanded={true}` → `.navigation-rail--expanded` class on root
- [ ] `isDisabled` item: `aria-disabled="true"`, press suppressed
- [ ] Tab order: menu button → FAB → destinations (DOM order)

## QA

- [ ] Indicator pill: 56 dp centered over icon; matches M3 spec
- [ ] Expanded state: labels inline-end; width 256 dp; indicator full-row
- [ ] Expand/collapse animation at 60 fps; correct easing per direction
- [ ] Test at 600 dp (medium) and 840 dp+ (expanded) breakpoints
- [ ] RTL: rail at inline-start (right edge in RTL); no direction-specific CSS needed
- [ ] Screen reader: "navigation" landmark; active item "current page"; menu button "expanded/collapsed"
- [ ] Badge overlap with indicator pill at dot, 1-digit, 3-digit, 999+ counts
