# Button Group — TODO ✦ M3 Expressive

## MUI Reference (what we are beating)

**MUI `ButtonGroup`** (`@mui/material/ButtonGroup`) — a bordered row of buttons. No selection
model, no shape morphing, no 5-level size scale, no M3 token mapping. Three sizes
(small/medium/large), orientation prop, and a `disabled` prop for all-or-nothing disabling.

This component is **net-new territory**: MUI has nothing close to M3 Expressive's shape morph or
toggle model. The win here is not just "more faithful" — it is building something MUI cannot do.

---

## Beat-MUI Opportunities

### 1. Shape-morph press response — MUI cannot do this at all
**MUI**: static borders; press state is color-only (ripple on material). No spatial/shape feedback.
**kafUI win**: connected buttons physically "react" to press — the pressed item's inner corners
contract, adjacent items' touching corners soften — all via CSS `border-radius` transitions keyed
on RAC's `[data-pressed]` and CSS `:has()`. Zero JavaScript for the morph itself. This is a
defining differentiator for M3 Expressive and is invisible to MUI users.

**Tasks:**
- [ ] Implement per-item corner variables `--_rs-ss`, `--_rs-se`, `--_rs-es`, `--_rs-ee` (see
  SPEC: Shape Morph Details).
- [ ] `:has([data-pressed]) + .button-group__item` rule for sibling softening.
- [ ] `:has(+ .button-group__item[data-pressed])` rule for preceding sibling.
- [ ] `transition: border-radius var(--duration-short4) var(--easing-standard)` on items.
- [ ] `@media (prefers-reduced-motion: reduce)` → `transition: none`.
- [ ] Progressive-enhancement fallback: if `:has()` is not supported, set
  `data-adjacent-pressed` on siblings in React `onPointerDown`/`onPointerUp` handlers and write
  CSS rules targeting `[data-adjacent-pressed]`.
- [ ] Storybook story: slow-motion shape morph demo (`transition-duration: 2s`).

### 2. Built-in toggle/selection model — MUI has literally zero
**MUI**: no selection. Consumers wire `onClick` + `useState` + conditional `variant` props
manually, repeating this pattern for every toolbar/alignment-switcher use-case.
**kafUI win**: `selectionMode="single"|"multiple"` activates RAC `ToggleButtonGroup` with full
keyboard navigation, `aria-checked`, `aria-pressed`, roving tabindex — out of the box. One prop
replaces ~30 lines of hand-rolled state + aria wiring.

**Tasks:**
- [ ] `ButtonGroup` detects `selectionMode` and swaps between `<div role="group">` (action)
  and RAC `ToggleButtonGroup` (toggle). Keep the same consumer API; internals bifurcate.
- [ ] Pass `selectedKeys`, `defaultSelectedKeys`, `onSelectionChange`, `isDisabled` to RAC
  `ToggleButtonGroup` in toggle mode.
- [ ] `ButtonGroupItem` detects mode via React context: renders RAC `Button` (action) or
  RAC `ToggleButton` (toggle).
- [ ] Dev-mode invariant: warn if `id` is missing on items when `selectionMode !== "none"`.
- [ ] Test: single-select — selecting B deselects A. Multi-select — A and B can be co-selected.

### 3. 5-level size scale — MUI has 3, M3 Expressive has 5
**MUI**: small (32dp) / medium (36dp) / large (40dp) — compressed and not M3-aligned.
**kafUI win**: `xs` (32) / `sm` (40) / `md` (48, default) / `lg` (56) / `xl` (96). Crucially,
all sizes meet the 48-dp touch target via `min-block-size` + `padding-block`. MUI's `small`
(32dp) fails WCAG 2.5.8 (Target Size Minimum). kafUI never fails.

**Tasks:**
- [ ] Size modifier classes `--size-{xs|sm|md|lg|xl}` set CSS custom properties `--_h` and
  `--_pad-inline` on the group. Items inherit.
- [ ] For `xs` and `sm`: `min-block-size: 48px; padding-block: calc((48px - var(--_h)) / 2)` on
  items so touch target is always 48dp regardless of visual height.
- [ ] Test: each size renders expected height; touch target is always ≥48px.
- [ ] Storybook: all 5 sizes in one story.

### 4. CSS-first variant cascade — MUI requires SX or `createTheme`
**MUI**: variant per-button is a prop (`variant="outlined"` on each `Button`); group-level default
requires `createTheme` overrides or manual `sx` on every child.
**kafUI win**: `variant` on `<ButtonGroup>` cascades to all items via React Context + CSS custom
properties. A per-item `variant` prop overrides the group default. No theme wrapping, no SX, no
style duplication.

**Tasks:**
- [ ] Create `ButtonGroupContext` providing `{ variant, size, connected }`.
- [ ] `ButtonGroupItem` reads context; merges per-item `variant` override.
- [ ] CSS: variant modifiers on `.button-group` set `--_container-color`, `--_content-color`,
  `--_outline-color` custom properties; items consume these.
- [ ] Verify: item with explicit `variant` prop does NOT inherit group variant.

### 5. Connected mode with proper M3 dividers — MUI's dividers don't differentiate
**MUI**: borders are the button's own border, duplicated at adjacency — results in 2px double
borders. Must use `disableElevation` and `sx` hacks to fix.
**kafUI win**: connected mode uses `gap: 0` + `border-inline-end: 1px solid var(--outline-variant)`
on all items except `:last-child` — single 1dp divider, correct color, no doubled borders,
no JS.

**Tasks:**
- [ ] `--connected` modifier: `--_gap: 0` on group; `border-inline-end: 1px solid var(--outline-variant)`
  on `.button-group__item:not(:last-child)`.
- [ ] Group itself gets `border: 1px solid var(--outline)` and `border-radius: var(--corner-full)`
  in connected mode; `overflow: hidden` clips item state layers to the pill shape.
- [ ] First and last items: use `:first-child` / `:last-child` for pill-end corners. Add
  `data-first` / `data-last` attributes from React as a backup for wrapped-children edge cases.
- [ ] Test: no doubled borders; divider is 1dp; outer corners are pill; inner corners are square
  (at rest).

### 6. Zero Emotion, zero runtime style injection
**MUI**: Emotion injects style objects on each render; ButtonGroup generates new CSS rules on
each interaction (ripple, hover).
**kafUI win**: all styles live in `@layer kafui` in a static stylesheet. State layers are
`::before` pseudo-elements driven by RAC `data-*` attributes. No CSS is injected at runtime.
First meaningful paint is faster; long-task budget is not consumed by style injection.

**Tasks:**
- [ ] State layer `::before` on `.button-group__item`: `background: currentColor;
  opacity: 0; position: absolute; inset: 0; border-radius: inherit`.
- [ ] Opacity rules for `[data-hovered]`, `[data-focus-visible]`, `[data-pressed]`.
- [ ] Verify no inline `style` attributes are emitted from the React component.

### 7. RTL and reduced motion — zero config
**MUI**: `theme.direction = "rtl"` required; no built-in reduced motion.
**kafUI win**: logical CSS properties throughout; `dir="rtl"` flips layout automatically.
`@media (prefers-reduced-motion)` block zeroes all transitions.

**Tasks:**
- [ ] Audit all directional CSS; replace physical properties with logical equivalents
  (`padding-inline`, `border-start-*-radius`, `border-inline-end`, etc.).
- [ ] Add `@media (prefers-reduced-motion: reduce) { .button-group__item { transition: none; } }`.
- [ ] Storybook: RTL story.

---

## Implementation Checklist

### Styles (`@kafui/styles`)

- [ ] `.button-group`: inline-flex; `gap: var(--_gap)` (4px standard / 0 connected).
- [ ] `--connected` modifier: group border + border-radius + overflow hidden; dividers (Beat-MUI #5).
- [ ] Per-item corner vars + shape morph (Beat-MUI #1).
- [ ] Size modifiers (Beat-MUI #3).
- [ ] Variant modifiers: `--_container-color`, `--_content-color`, `--_outline-color` (Beat-MUI #4).
- [ ] Elevated variant: `box-shadow: var(--elevation-1)` at rest; `var(--elevation-2)` on hover.
- [ ] State layer `::before` (Beat-MUI #6).
- [ ] `[data-disabled]`: `opacity: 0.38; pointer-events: none`.
- [ ] RTL + reduced motion (Beat-MUI #7).

### React Component (`@kafui/react`)

- [ ] `ButtonGroup`: mode detection + RAC primitive swap (Beat-MUI #2).
- [ ] `ButtonGroupContext` for variant/size cascade (Beat-MUI #4).
- [ ] `ButtonGroupItem`: action vs toggle detection via context (Beat-MUI #2).
- [ ] `data-first` / `data-last` attributes on first/last items.
- [ ] `:has()` fallback via `data-adjacent-pressed` (Beat-MUI #1).
- [ ] `aria-label` / `aria-labelledby` dev-mode guard on group.
- [ ] `id` required guard on items in toggle mode.
- [ ] Export all types from package index.

### Tests

- [ ] Action mode: each `onPress` fires independently; no cross-contamination.
- [ ] Toggle single: selecting B deselects A; `onSelectionChange` called correctly.
- [ ] Toggle multi: A and B co-selected; independent deselect works.
- [ ] Keyboard: Tab in action mode; arrow keys in toggle mode; Space/Enter activates.
- [ ] Shape morph: `[data-pressed]` attribute present on press; adjacent item `[data-adjacent-pressed]`
  present (fallback path).
- [ ] Disabled item: `aria-disabled` set; not interactive.
- [ ] Connected vs standard: correct gap; dividers present only in connected mode.
- [ ] All 5 sizes: correct heights; touch target ≥48px.
- [ ] All 5 variants: correct container/content colors.
- [ ] Snapshots: connected+tonal, standard+outlined, toggle+single.

### Documentation / Storybook

- [ ] Story: action group — outlined connected (text formatting).
- [ ] Story: toggle single — tonal connected (alignment).
- [ ] Story: toggle multi — outlined standard.
- [ ] Story: all 5 sizes comparison.
- [ ] Story: all 5 variants comparison.
- [ ] Story: shape morph slow-motion (`transition-duration: 2s`).
- [ ] Story: RTL layout.
- [ ] Docs callout: "M3 Expressive — no MUI equivalent. Shape morph, 5-level size scale, and
  built-in toggle selection are unique to kafUI."
- [ ] Docs note: `:has()` browser baseline and graceful degradation behavior.
