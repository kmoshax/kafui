# FAB Menu — TODO ✦ M3 Expressive

## MUI Reference (what we are beating)

**MUI `SpeedDial`** (`@mui/material/SpeedDial`) — the closest analogue. It is not an M3
component, opens in 4 directions, uses `Tooltip` for labels (bad on touch), does not morph the
FAB shape, has no M3 token mapping, and relies on Emotion CSS-in-JS. Every item below is a
concrete way kafUI does it better.

---

## Beat-MUI Opportunities

### 1. Inline labels — no tooltip delay, always accessible on touch
**MUI SpeedDial**: action labels are `Tooltip` components — shown on mouse hover only. On touch
devices, tapping an action executes it without ever showing the label. This is a UX failure for
any action where the icon alone is ambiguous.
**kafUI win**: `FabMenu.Item` always renders its `label` as a visible `__item-label` element
beside the icon pill. Labels appear as part of the open animation — no hover required, no delay,
no invisible state on touch. Per M3 spec, this is the required behavior.

**Tasks:**
- [ ] `FabMenu.Item` renders `<span class="fab-menu__item-label">` unconditionally.
- [ ] Label positioned beside `__item-pill` via flex `gap: 12px`.
- [ ] `labelSide` prop: `--label-start` modifier flips label to `inline-start`.
- [ ] Label fades in as part of item entrance animation (same `animation-delay` as the pill).
- [ ] Test: labels are visible in DOM when menu is open; not visible when closed
  (`visibility: hidden` or `opacity: 0` per `[data-open]`).
- [ ] Storybook: show both `labelSide="start"` and `labelSide="end"` stories.

### 2. FAB shape morph + icon cross-fade — MUI SpeedDial does neither
**MUI**: the FAB changes its `openIcon` and `closeIcon` via a React state swap, but the shape
stays static. No border-radius transition; no spatial metaphor for "expanding."
**kafUI win**: on open, the FAB's `border-radius` transitions from `--corner-large` (16dp) to
`--corner-extra-large` (28dp) — the FAB becomes more "square" as it opens, visually growing to
contain the menu. Simultaneously, the open icon and close icon cross-fade via absolute-positioned
`opacity` transitions — no layout shift, no flicker.

**Tasks:**
- [ ] FAB `border-radius` transition: `var(--corner-large) → var(--corner-extra-large)` on
  `[data-open]`; `var(--duration-medium4)` `var(--easing-emphasized-decelerate)`.
- [ ] Both icon elements (`__fab-open-icon`, `__fab-close-icon`) always in DOM:
  `position: absolute` on both; opacity 1 / 0 swaps via `[data-open]`.
- [ ] Icon cross-fade: 200ms; slightly offset so open-icon fades out before close-icon reaches
  full opacity (avoids simultaneous full-opacity overlap).
- [ ] Extended FAB: `inline-size` transition required in addition to `border-radius` for the
  pill-to-square morph. Use `inline-size: var(--_fab-size)` in open state.
- [ ] `@media (prefers-reduced-motion: reduce)`: `transition: none; animation: none` on FAB and
  both icons.
- [ ] Storybook story: slow-motion morph (`transition-duration: 2s`).

### 3. Staggered item animation — MUI has a uniform zoom, no M3 stagger
**MUI SpeedDial**: items use a MUI `Zoom` transition — all items animate at the same time with
the same duration. No stagger.
**kafUI win**: items slide up + fade in with a 40ms per-item stagger, giving a "cascade" motion
that reads as a cohesive sequence. On close, the outermost item exits first (reverse stagger),
reinforcing the spatial hierarchy. All done in pure CSS `@keyframes` + `animation-delay` keyed
on `--_i` (index) and `--_i-reverse` (reverse index) CSS custom properties — zero JS per frame.

**Tasks:**
- [ ] `@keyframes fab-item-in` / `fab-item-out` per SPEC.
- [ ] `[data-open] .fab-menu__item`: `animation: fab-item-in 200ms …; animation-delay: calc(var(--_i) * 40ms)`.
- [ ] `[data-closing] .fab-menu__item`: `animation: fab-item-out 200ms …; animation-delay: calc(var(--_i-reverse) * 40ms)`.
- [ ] React: set `style={{ '--_i': index, '--_i-reverse': totalItems - 1 - index }}` on each
  item element. This is the only acceptable inline style usage.
- [ ] `data-closing` lifecycle: set on `onOpenChange(false)`, remove after
  `(totalItems * 40 + 200)ms` via `setTimeout`.
- [ ] Verify: `animation-fill-mode: both` so items start hidden and end hidden correctly.
- [ ] `@media (prefers-reduced-motion: reduce)`: `animation: none` on items;
  menu toggles via `visibility` only.

### 4. Correct `role="menu"` + `role="menuitem"` — MUI SpeedDial uses `role="presentation"`
**MUI SpeedDial**: the action list uses `role="presentation"` on the container. Individual actions
are `<SpeedDialAction>` rendered as `Tooltip` + `Fab` — technically a button, but not `menuitem`.
Screen readers have no menu semantics; they read a list of buttons without context.
**kafUI win**: menu panel is `role="menu"` with `aria-label` matching the FAB label. Each item
is `role="menuitem"`. Screen readers announce "Create actions menu" when items receive focus. Menu
semantics communicate the grouped-action pattern correctly.

**Tasks:**
- [ ] Menu panel `<div role="menu" [aria-label]>` wrapping items.
- [ ] Each `FabMenu.Item` renders a RAC `Button` with `role="menuitem"` via `elementType` or
  RAC `slot` override.
- [ ] `aria-label` on `FabMenu` is TypeScript-required (non-optional).
- [ ] FAB `aria-haspopup="menu"`, `aria-expanded`, `aria-controls` — wired correctly.
- [ ] `aria-disabled="true"` on disabled items; disabled items not focusable via arrow keys.
- [ ] Test: screen reader test (axe or jest-axe) passes with no violations.

### 5. Focus scope with non-modal trapping — MUI SpeedDial uses Backdrop
**MUI SpeedDial**: optionally shows a backdrop overlay when open; focus is not programmatically
managed — clicking outside dismisses but keyboard focus can escape to the page behind.
**kafUI win**: RAC `FocusScope` wraps the menu panel with `contain={true}` — arrow keys are
trapped in the menu; Tab exits intentionally (and closes the menu). `useInteractOutside` from
RAC handles outside-click dismiss without a backdrop overlay. Result: no page content is obscured,
keyboard users cannot accidentally interact with background content while the menu is open.

**Tasks:**
- [ ] Wrap `__menu` contents in RAC `FocusScope` with `contain`, `restoreFocus`, `autoFocus`.
- [ ] `useInteractOutside` on root element to close menu on outside pointer event.
- [ ] `closeOnScroll`: `useEffect` scroll listener on `window`; closes when `closeOnScroll=true`.
- [ ] On open: after `[data-open]` set, `focus()` the first non-disabled `__item`.
- [ ] On close (Escape or item activation): `focus()` returns to `__fab`.
- [ ] Test: Tab from last item closes menu and moves focus past the component.
- [ ] Test: ArrowUp/Down cycles within items (wraps at ends); does not escape to page.

### 6. Vertical-only direction — MUI SpeedDial's 4-direction prop is spec-breaking
**MUI SpeedDial**: `direction="up|down|left|right"` — expanding left or right is not an M3
Expressive pattern. It creates a complex, potentially overlapping layout.
**kafUI win**: M3 specifies vertical upward only. kafUI has no `direction` prop — one fewer API
surface to document, test, and support, and no risk of consumers using a layout that clashes with
Material 3. Document this as a deliberate spec-faithful decision.

**Tasks:**
- [ ] No `direction` prop — document explicitly in JSDoc and Storybook.
- [ ] Docs note: "FAB Menu expands upward only per M3 Expressive spec. For directional expansion,
  consider a custom menu composition."
- [ ] Items stack in `flex-direction: column-reverse` with `position: absolute; bottom: calc(100% + 8px)`
  on the menu panel — items appear above the FAB.

### 7. Zero Emotion, CSS-first animations
**MUI SpeedDial**: Emotion injects transition styles on each open/close; `Zoom` component
generates style objects at runtime.
**kafUI win**: all animations are static `@keyframes` in `@layer kafui`. The only runtime DOM
mutation is setting `data-open` / `data-closing` attributes and the `--_i` / `--_i-reverse`
inline CSS custom properties. No style injection; no new CSS classes generated at runtime; no
re-renders triggered by hover/animation state.

**Tasks:**
- [ ] Static `@keyframes fab-item-in` and `fab-item-out` in stylesheet.
- [ ] No `style` objects in JSX except `--_i` / `--_i-reverse` custom properties per item.
- [ ] State layer `::before` pseudo-elements on FAB and item pill; opacity keyed on RAC
  `data-hovered`, `data-pressed`, `data-focus-visible`.
- [ ] Test: no Emotion class mutations observed on open/close cycle.

---

## Implementation Checklist

### Styles (`@kafui/styles`)

- [ ] `.fab-menu`: flex column; `align-items: flex-end` (label-end default) (Beat-MUI #1, #6).
- [ ] Variant modifiers: `--_fab-container` / `--_fab-icon-color` per variant (Beat-MUI #2).
- [ ] Size modifiers: `--_fab-size` / `--_icon-size` per size.
- [ ] `.fab-menu__fab`: border-radius + morph + elevation (Beat-MUI #2).
- [ ] Icon cross-fade: `__fab-open-icon` / `__fab-close-icon` (Beat-MUI #2).
- [ ] Extended modifier: `inline-size` morph (Beat-MUI #2).
- [ ] `__menu`: absolute position above FAB; `pointer-events: none` when closed (Beat-MUI #6).
- [ ] Item animations + stagger (Beat-MUI #3).
- [ ] Item pill: 56dp circle; `--surface-container` fill; `--on-surface` icon.
- [ ] State layers on FAB and item pill (Beat-MUI #7).
- [ ] FAB hover: `box-shadow: var(--elevation-4)`.
- [ ] `[data-disabled]`: `opacity: 0.38; pointer-events: none`.
- [ ] `--label-start` modifier (Beat-MUI #1).
- [ ] RTL: logical properties on label positioning.
- [ ] `@media (prefers-reduced-motion: reduce)` (Beat-MUI #2, #3).

### React Component (`@kafui/react`)

- [ ] `FabMenu`: root div with `[data-open]` / `[data-closing]` attribute management.
- [ ] `FocusScope` + `useInteractOutside` (Beat-MUI #5).
- [ ] Scroll dismiss via `useEffect` (Beat-MUI #5).
- [ ] `data-closing` timeout (Beat-MUI #3).
- [ ] Per-item `--_i` / `--_i-reverse` inline custom props (Beat-MUI #3).
- [ ] `FabMenu.Item` → RAC `Button` with `role="menuitem"` (Beat-MUI #4).
- [ ] ARIA wiring on FAB (Beat-MUI #4).
- [ ] `aria-label` TypeScript-required (Beat-MUI #4).
- [ ] Discriminated union for `extended` / `fabLabel` (SPEC).
- [ ] Controlled/uncontrolled open state via `useControlledState`.
- [ ] Export all types from package index.

### Tests

- [ ] FAB press opens menu; `aria-expanded="true"`; first item focused.
- [ ] Escape closes; focus returns to FAB.
- [ ] Item press fires `onPress`; closes menu; focus returns.
- [ ] Click outside closes menu.
- [ ] Scroll closes when `closeOnScroll=true`; stays open when `false`.
- [ ] ArrowUp/Down navigate items; wrap at ends.
- [ ] Disabled item: not reachable via arrows; `aria-disabled` set.
- [ ] `isOpen` controlled: component respects external state.
- [ ] `data-open` present when open; `data-closing` present during close animation.
- [ ] `--_i` / `--_i-reverse` correct on each item.
- [ ] Snapshot: all 4 variants × light+dark.
- [ ] Snapshot: all 3 sizes.
- [ ] Extended FAB: label visible at rest; hidden in open state.
- [ ] jest-axe: no a11y violations on open state.

### Documentation / Storybook

- [ ] Story: default primary medium FAB with 3 actions.
- [ ] Story: all 4 FAB variants.
- [ ] Story: all 3 FAB sizes.
- [ ] Story: extended FAB at rest morphing on open.
- [ ] Story: `labelSide="start"` (left-positioned FAB).
- [ ] Story: slow-motion animation demo.
- [ ] Story: RTL layout.
- [ ] Story: controlled `isOpen`.
- [ ] Docs callout: "Unlike MUI SpeedDial, kafUI FAB Menu always shows labels inline (no tooltip
  hover required), expands upward only per M3 spec, and morphs the FAB shape on open."
- [ ] Docs note: "Position the `.fab-menu` element using `position: fixed; inset-block-end: 16px;
  inset-inline-end: 16px` in your layout CSS. The component does not impose page-level
  positioning."
- [ ] Docs note: "No `direction` prop — vertical upward only per M3 Expressive."
