# Navigation Drawer — TODO

## MUI Equivalent

**`Drawer`** (`@mui/material`).

MUI's `Drawer` supports `variant="permanent"` (standard), `variant="persistent"`, and `variant="temporary"` (modal). It is a positioning/overlay shell — navigation semantics, `aria-current`, section structure, and item styling are entirely the consumer's responsibility. MUI wraps `<nav>` only if the consumer adds it. Every gap below is an opportunity to beat MUI.

---

## Beat-MUI Opportunities

### 1. Enforced `<nav>` + `aria-current` semantics — MUI defers this entirely to consumers

MUI's `Drawer` renders `<div role="presentation">` (temporary) or a bare `<div>` (permanent). Navigation semantics are a consumer problem. kafUI enforces `<nav>` + `<a>` + `aria-current="page"` through its compound API — correct semantics are the default, not an opt-in.

**Tasks:**
- [ ] Standard: `<nav aria-label={ariaLabel ?? "Application navigation"}>` wrapping `<ul role="list">` and sections
- [ ] Modal: RAC `<Dialog aria-label={ariaLabel}>` wrapping `<nav>` — dialog provides `aria-modal`; nav provides landmark
- [ ] Each item: `<li>` > RAC `<Link href={href} aria-current={isCurrent ? "page" : undefined}>`
- [ ] Inactive items: `aria-current` attribute omitted entirely
- [ ] Screen-reader test: "navigation" landmark; items announced as links; active item as "current page"

### 2. Full-row active indicator pill — MUI uses flat selected color, no pill

MUI's active state on `<ListItemButton selected>` sets a background-color on the full button but doesn't give it rounded pill ends. kafUI ships the M3 `--corner-full` full-row pill with `--secondary-container` fill.

**Tasks:**
- [ ] `.navigation-drawer__item`: `position: relative; height: var(--item-h); display: flex; align-items: center; padding-inline: var(--item-px)`
- [ ] `.navigation-drawer__item-indicator`: `position: absolute; inset: 4px 12px; border-radius: var(--corner-full); background: transparent; transition: background var(--duration-short2) var(--easing-emphasized)`
- [ ] Active: `[aria-current="page"] ~ .navigation-drawer__item-indicator` or `.navigation-drawer__item--active .navigation-drawer__item-indicator { background: var(--secondary-container) }`
- [ ] Active icon: `color: var(--on-secondary-container)`; inactive: `color: var(--on-surface-variant)`
- [ ] Active label: `color: var(--on-secondary-container); font: var(--label-large-font)`

### 3. WAI-ARIA Dialog for modal focus trap — MUI uses its own `FocusTrap`, not ARIA dialog

MUI `Modal` uses a non-dialog focus trap (no `role="dialog"`, no `aria-modal` in the navigation context). RAC `<Dialog>` provides spec-correct `role="dialog" aria-modal="true"`, Escape handling, and proven focus management.

**Tasks:**
- [ ] Modal: wrap in RAC `<ModalOverlay>` + `<Modal>` + `<Dialog aria-label={ariaLabel ?? "Application navigation"}>`
- [ ] `isOpen` → RAC `<ModalOverlay isOpen={isOpen} onOpenChange={(open) => !open && onClose?.()}>`
- [ ] Escape key → `onClose()` via RAC dialog default behavior (no manual keydown handler)
- [ ] Scrim: RAC `<ModalOverlay>` handles scrim; style via `.navigation-drawer__scrim` or `::before` on the overlay
- [ ] Focus on open: moves to first focusable item (or close button if present)
- [ ] Focus on close: returns to trigger via RAC's built-in focus restoration

### 4. `shape.corner.large` on modal drawer corners — MUI has no M3-aligned corner shaping

M3 specifies `shape.corner.large` (16 dp) on the inline-end corners of the modal drawer only. MUI has no first-class concept of asymmetric corner radii per M3 shape tokens.

**Tasks:**
- [ ] Modal: `border-start-end-radius: var(--corner-large); border-end-end-radius: var(--corner-large)` on drawer surface
- [ ] Standard: no explicit corner radius (full-height flush edge)
- [ ] RTL: logical properties auto-flip the rounded corners to inline-start corners in RTL (start-end → end-end becomes correct side)

### 5. Section headlines + dividers as first-class compound API — MUI requires manual `<List>` + `<ListSubheader>` + `<Divider>` assembly

MUI requires consumers to manually compose `<List>`, `<ListSubheader>`, and `<Divider>` with no semantic enforcement. kafUI's `NavigationDrawer.Section` renders `<section aria-labelledby>` + `<h2>` + `<hr>` automatically.

**Tasks:**
- [ ] `NavigationDrawer.Section`: render `<section aria-labelledby={headlineId}>`; if `headline`, render `<h2 id={headlineId} className="navigation-drawer__headline">`; if `divider`, render `<hr className="navigation-drawer__divider">`; render `<ul role="list">` around children
- [ ] `__headline`: `font: var(--title-small-font) / var(--title-small-line-height) var(--title-small-size); color: var(--on-surface-variant); padding-block-start: 16px; padding-block-end: 4px; padding-inline: 24px`
- [ ] `__divider`: uses `--outline-variant` as border color; `margin-block: 8px`
- [ ] Sections without headline: plain `<ul role="list">` (no `<section>` wrapper needed)

### 6. Logical-property modal animation — MUI uses `anchor="left"` physical prop for RTL

MUI requires `anchor="right"` to position the drawer on the correct side in RTL. kafUI uses `inset-inline-start: 0` and `translate` with an `[dir="rtl"]` override for the animation direction.

**Tasks:**
- [ ] Standard: `position: fixed; inset-block: 0; inset-inline-start: 0; width: var(--w); background: var(--surface); display: flex; flex-direction: column`
- [ ] Modal closed: `translate: -100% 0`; open: `translate: 0 0`; transition: `translate var(--duration-medium2) var(--easing-emphasized-decelerate)` on open, `var(--easing-emphasized-accelerate)` on close
- [ ] RTL override: `[dir="rtl"] .navigation-drawer--modal { translate: 100% 0 }` (closed in RTL = translate right)
- [ ] `@media (prefers-reduced-motion: reduce)`: `transition: none`; use `visibility: hidden/visible` toggle instead

### 7. Scrollable content with sticky header — MUI defers scroll handling to consumer

**Tasks:**
- [ ] `__content`: `overflow-y: auto; flex: 1; overscroll-behavior: contain`
- [ ] `__header`: outside `__content`; not in scroll flow; sticky above scroll naturally by flex order
- [ ] `scrollbar-width: thin` (Firefox) + `::-webkit-scrollbar` thin style for consistent cross-browser

### 8. Single-source theming — MUI needs `theme.palette` + `styleOverrides`

**Tasks:**
- [ ] Verify all color references use unprefixed tokens: `--surface`, `--secondary-container`, `--on-secondary-container`, `--on-surface-variant`, `--scrim`
- [ ] No hardcoded hex values anywhere in component CSS
- [ ] Dark mode test: `color-scheme: dark` flips all `light-dark()` roles — drawer looks correct without any extra class

---

## Styles (`@kafui/styles`)

- [ ] Create `src/components/navigation-drawer/_navigation-drawer.css` inside `@layer kafui { }`
- [ ] State layer on `<a>::after` (full row): `content: ""; position: absolute; inset: 0; background: currentColor; opacity: 0; transition: opacity 0.15s`
- [ ] `[data-hovered] > a::after { opacity: var(--state-hover) }`, focus + pressed similar
- [ ] Note: state layer on drawer items is full-row (unlike bar/rail which scope to indicator container)

## React (`@kafui/react`)

- [ ] Create `src/components/NavigationDrawer/NavigationDrawer.tsx`
- [ ] Export `NavigationDrawer`, `NavigationDrawer.Section`, `NavigationDrawer.Item`
- [ ] `NavigationDrawer.Item`: `isDisabled` → `aria-disabled="true"` + `.navigation-drawer__item--disabled`; suppress press

## Tests

- [ ] Standard: renders `<nav>` with `aria-label`; no scrim; no `aria-modal`
- [ ] Modal: renders `<dialog aria-modal="true">` (via RAC); scrim present; Escape fires `onClose`
- [ ] Scrim click fires `onClose`
- [ ] Active item `<a>` has `aria-current="page"` + `--active` modifier; inactive items have no `aria-current`
- [ ] Focus trap in modal: Tab does not escape drawer; Shift+Tab wraps
- [ ] Focus returns to trigger on close (test `userEvent.keyboard('{Escape}')`)
- [ ] Section with headline renders `<h2 id>` + `<section aria-labelledby>` pairing
- [ ] Divider renders `<hr>` (not `<div>`)
- [ ] RTL: `dir="rtl"` — drawer at `inline-start` (right side); `border-start-end-radius` on correct corners

## QA

- [ ] Full-row indicator pill width; `--corner-full` rounded ends; `--secondary-container` fill
- [ ] `--corner-large` on inline-end corners of modal drawer (visual diff vs standard)
- [ ] Modal open animation: slide-in from `inline-start` with decelerate easing
- [ ] Modal close animation: slide-out with accelerate easing
- [ ] Scrollable content: 20+ items; `__header` stays sticky; `__content` scrolls
- [ ] Screen reader: landmark "navigation"; active item "current page"
- [ ] Breakpoints: 360 dp (modal), 600 dp (modal), 840 dp+ (standard)
