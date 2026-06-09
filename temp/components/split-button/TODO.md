# Split Button — TODO ✦ M3 Expressive

## MUI Reference (what we are beating)

**No first-class MUI component.** MUI docs offer a "Split Button" recipe that manually combines:
`ButtonGroup` + two `Button` elements + `ArrowDropDown` icon + `Popper` + `Grow` + `Paper` +
`ClickAwayListener` + `MenuList` — approximately 60–80 lines of consumer boilerplate, 7+ imports,
manual `aria-*` attributes, and zero shape-morph. It is an ad-hoc recipe, not a component.

This is an **M3 Expressive (2025)** component kafUI builds natively. Every item below is a
concrete way kafUI beats that MUI recipe.

---

## Beat-MUI Opportunities

### 1. One import vs seven — single compound component
**MUI recipe**: consumers import `ButtonGroup`, `Button`, `MenuItem`, `MenuList`, `Paper`,
`Popper`, `Grow`, `ClickAwayListener`, `ArrowDropDown` icon, plus write ~70 lines of wiring
(`anchorRef`, `open` state, `handleToggle`, `handleClose`, `handleMenuItemClick`, `handleListKeyDown`
— all verbatim from MUI docs). This is the definition of boilerplate.
**kafUI win**:

```tsx
// MUI recipe: ~70 lines + 7 imports
// kafUI: 8 lines, 1 import
<SplitButton variant="filled" label="Create" onPress={create} onAction={handleAction} aria-label="Create with options">
  <SplitButton.Item id="template">From template</SplitButton.Item>
  <SplitButton.Item id="import">Import…</SplitButton.Item>
</SplitButton>
```

**Tasks:**
- [ ] `SplitButton` renders: root `<div role="group">` + action `<Button>` + divider + RAC
  `<MenuTrigger><Button __trigger /><Popover><Menu>{children}</Menu></Popover></MenuTrigger>`.
- [ ] `SplitButton.Item` = RAC `MenuItem` with correct class.
- [ ] `onAction` forwarded to RAC `Menu`'s own `onAction` prop — zero consumer wiring.
- [ ] `onOpenChange` drives `data-open` attribute on root — CSS does the rest.
- [ ] Export `SplitButton` and `SplitButton.Item` from `@kafui/react`; single import.

### 2. Shape morph + chevron rotation — MUI recipe has none
**MUI**: static appearance regardless of menu state. The "split" visual is just two adjacent
buttons; no feedback that the menu is anchored/open.
**kafUI win**: opening the menu triggers:
1. Trailing corners of the trigger segment contract to `--corner-extra-small` (≈4dp) via CSS
   `border-radius` transition — the control visually "opens toward" the menu.
2. Chevron rotates 180° simultaneously.
3. Both transitions use M3 `emphasized-decelerate` easing on open, `emphasized-accelerate` on
   close — communicating intentional vs dismissive motion.
Zero JavaScript for the morph; `[data-open]` on the root drives all CSS.

**Tasks:**
- [ ] Implement trigger corner variables `--_rs-ss/se/es/ee` and `[data-open]` override per SPEC.
- [ ] Chevron `transition: transform var(--duration-medium1) var(--easing-emphasized)`.
- [ ] `[data-open] .split-button__trigger-icon { transform: rotate(180deg) }`.
- [ ] Handle open/close easing split: add `data-closing` attribute to root during the close
  animation (set on `onOpenChange(false)`, remove after `var(--duration-medium1)` ms via
  `setTimeout`). CSS: `[data-closing] .split-button__trigger { transition-timing-function:
  var(--easing-emphasized-accelerate) }`.
- [ ] `@media (prefers-reduced-motion: reduce)`: all transitions `none`.
- [ ] Storybook story: slow-motion morph demo (`transition-duration: 2s`).

### 3. Zero Popper.js, zero ClickAwayListener
**MUI recipe**: uses Popper.js (positioning library), `Grow` (transition), `Paper` (surface),
`ClickAwayListener` (dismiss) — all separate runtime dependencies with JS overhead.
**kafUI win**: RAC `MenuTrigger` + `Popover` handle positioning (using the Floating UI primitives
already in RAC), dismiss-on-outside-click, focus management, and keyboard navigation natively. No
additional runtime cost; everything is already in the RAC dependency.

**Tasks:**
- [ ] Use RAC `MenuTrigger` as the trigger wrapper (not a custom `useState` + `useRef` pattern).
- [ ] Use RAC `Popover` with `placement="bottom end"` and `shouldFlip={true}` for auto-flip.
- [ ] Confirm RAC `Popover` positions correctly relative to the trigger segment (not the full
  split-button root). Pass the trigger `Button` ref as the `triggerRef` if needed.
- [ ] Menu panel default minimum width = full split-button width. Set via inline CSS custom
  property `--_menu-min-width` from a ref on the root div.
- [ ] Test: outside click closes menu; Escape closes menu; focus returns to trigger.

### 4. Correct ARIA — MUI recipe requires manual wiring
**MUI recipe**: consumer must manually add `aria-controls`, `aria-expanded`, `aria-haspopup` to
the toggle button. If they forget (common), the component fails accessibility audits silently.
**kafUI win**: RAC `MenuTrigger` manages `aria-haspopup="menu"`, `aria-expanded`, `aria-controls`
automatically. The root `role="group"` + `aria-label` is enforced as a required prop in TypeScript.
The trigger segment gets a generated `aria-label` (`"More {label} actions"`) if not explicitly
provided.

**Tasks:**
- [ ] `aria-label` on root is TypeScript-required (non-optional in interface).
- [ ] `triggerLabel` prop defaults to `` `More ${label} actions` `` in the component; no consumer
  action needed for a11y compliance.
- [ ] Validate: `aria-expanded` on trigger segment flips correctly on open/close.
- [ ] Validate: focus moves to first menu item on open; returns to trigger on Escape/selection.

### 5. M3 design tokens — MUI recipe uses unsemantic hardcoded colors
**MUI recipe**: colors via `sx={{ bgcolor: open ? 'primary.dark' : undefined }}` — hardcoded
palette references, no M3 semantic role mapping, breaks with theme changes.
**kafUI win**: `--_container`, `--_content` CSS custom properties set per variant modifier class.
A single `--source` override re-derives all roles; a `.theme-brand` class changes the whole
component appearance with one line.

**Tasks:**
- [ ] Variant modifier classes set `--_container`, `--_content` per SPEC token map.
- [ ] Elevated variant: `box-shadow: var(--elevation-1)` at rest; `var(--elevation-2)` on hover.
- [ ] Outlined variant: `outline: 1px solid var(--outline)` on root; outline clips with
  `overflow: hidden` or use `border` + careful border-radius handling.
- [ ] Divider color: `var(--outline-variant)` for filled/tonal (less prominent);
  `var(--outline)` for outlined (matches border). Implement via per-variant override of a
  `--_divider-color` custom property.
- [ ] Test: snapshot all 4 variants with correct colors in light and dark schemes.

### 6. 5-level size scale with guaranteed touch targets
**MUI recipe**: no size prop; inherits `ButtonGroup` sizes (small/medium/large — MUI mapping).
Smallest MUI option (32dp) violates WCAG 2.5.8 (Target Size Minimum of 24dp × 24dp; WCAG 2.5.5
recommends 44dp).
**kafUI win**: all 5 sizes maintain `min-block-size: 48px` via `padding-block`; smallest visual
footprint is 32dp but the touch target is always 48dp. Compliant at every size.

**Tasks:**
- [ ] Size modifiers set `--_h`, `--_action-pad`, `--_trigger-w` per SPEC.
- [ ] `min-block-size: 48px` on root; visual height from `block-size: var(--_h)`.
- [ ] Test: all 5 sizes render expected visual height; each segment touch target ≥48px.

### 7. RTL — CSS logical properties, zero JS
**MUI recipe**: RTL requires `theme.direction = 'rtl'` plus Popper direction adjustment.
**kafUI win**: all CSS uses logical properties. `[dir="rtl"]` flips the layout via browser
rendering, not JS. Shape-morph corner references (`border-start-end-radius` etc.) resolve
correctly in both directions.

**Tasks:**
- [ ] Audit entire stylesheet for physical properties; replace with logical equivalents.
- [ ] Shape morph: verify `border-start-end-radius` / `border-end-end-radius` on trigger map to
  the correct visual trailing corners in both LTR and RTL.
- [ ] Storybook RTL story.

---

## Implementation Checklist

### Styles (`@kafui/styles`)

- [ ] `.split-button`: inline-flex, stretch, `border-radius: var(--corner-full)`,
  `overflow: hidden`, `block-size: var(--_h)`, `min-block-size: 48px`.
- [ ] Variant modifiers (Beat-MUI #5).
- [ ] Size modifiers (Beat-MUI #6).
- [ ] Action + trigger segments: flex, align-center, `background: var(--_container)`,
  `color: var(--_content)`.
- [ ] `.split-button__action`: `padding-inline: var(--_action-pad)`.
- [ ] `.split-button__trigger`: `inline-size: var(--_trigger-w)`, justify-content: center.
- [ ] `.split-button__divider`: `inline-size: 1px; background: var(--_divider-color);
  block-size: 20px; align-self: center`.
- [ ] Shape morph on `[data-open]` (Beat-MUI #2).
- [ ] State layers `::before` on each segment; opacity rules per RAC data-attributes.
- [ ] `[data-disabled]`: `opacity: 0.38; pointer-events: none`.
- [ ] RTL (Beat-MUI #7).
- [ ] Reduced motion (Beat-MUI #2).

### React Component (`@kafui/react`)

- [ ] `SplitButton` compound structure (Beat-MUI #1).
- [ ] `SplitButton.Item` = RAC `MenuItem` (Beat-MUI #1).
- [ ] RAC `MenuTrigger` integration (Beat-MUI #3).
- [ ] `[data-open]` attribute management via `onOpenChange` (Beat-MUI #2).
- [ ] `[data-closing]` attribute + timeout for close easing (Beat-MUI #2).
- [ ] Generated `triggerLabel` default (Beat-MUI #4).
- [ ] `aria-label` TypeScript-required (Beat-MUI #4).
- [ ] Menu min-width = full split-button width via CSS custom property.
- [ ] Export `SplitButtonProps`, `SplitButtonItemProps` from package index.

### Tests

- [ ] Pressing action fires `onPress`; menu does NOT open.
- [ ] Pressing trigger opens menu; `aria-expanded="true"` on trigger.
- [ ] `data-open` applied to root when open; removed when closed.
- [ ] `onAction` fires with correct key on menu item selection; menu closes; focus returns.
- [ ] Escape closes menu; focus returns to trigger.
- [ ] ArrowDown from focused trigger opens menu; first item focused.
- [ ] Disabled split button: both segments non-interactive; `aria-disabled` set.
- [ ] All 4 variants: correct container/content colors (snapshot, light + dark).
- [ ] All 5 sizes: correct height; touch targets ≥48px.
- [ ] Shape morph: `[data-open]` causes trigger corner-radius change (computed style check).
- [ ] RTL: action is visually trailing, trigger leading (logical layout check).

### Documentation / Storybook

- [ ] Story: filled, default size — standard usage.
- [ ] Story: all 4 variants side by side.
- [ ] Story: all 5 sizes side by side.
- [ ] Story: slow-motion shape morph demo.
- [ ] Story: controlled open state (`isOpen` / `onOpenChange`).
- [ ] Story: disabled state.
- [ ] Story: RTL layout.
- [ ] Docs callout: "M3 Expressive — no MUI equivalent. Compare to MUI's 70-line recipe; kafUI
  achieves the same in 8 lines with shape-morph, full a11y, and M3 tokens."
- [ ] Docs note: `text` variant intentionally omitted; use `Button` + `Menu` separately if needed.
