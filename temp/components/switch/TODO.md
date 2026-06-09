# Switch — TODO

## MUI Equivalent

`Switch` + `FormControlLabel` (for label) from `@mui/material`. Two components for what kafUI does in one (`Switch` with `children` as label).

---

## Beat-MUI Opportunities

### 1. `role="switch"` — semantically correct out of the box
**MUI:** MUI's `Switch` renders `<input type="checkbox">` with no `role="switch"`. It is announced by screen readers as a "checkbox" with "checked/unchecked" state — the wrong semantic for a binary setting that takes immediate effect. The M3 spec and ARIA spec both require `role="switch"` with `aria-checked`.

**kafUI wins:** RAC `Switch` renders `<input type="checkbox" role="switch">` declaratively. Screen readers announce the correct role ("switch", "on/off") out of the box. This is a genuine accessibility improvement over MUI that requires zero consumer effort.

**Task:** Confirm in the rendered HTML that `role="switch"` is present on the hidden input. Write a Bun test asserting `role="switch"` and `aria-checked="false"` in initial render. Include the semantic contrast with MUI in Storybook documentation.

---

### 2. CSS-only thumb morph — no inline style, no JS measurement
**MUI:** MUI's Switch thumb position is implemented via `transform: translateX(...)` computed as an inline style from the checked state. The thumb size does not morph on press — MUI never implemented the M3 2024 handle size change. Any size customization requires `sx` prop or component slot override with Emotion.

**kafUI wins:** Handle size (`--handle-size`) is a CSS custom property toggled between `--handle-rest` (16dp), `--handle-sel` (24dp), and `--handle-press` (28dp) by BEM modifiers and `[data-pressed]` attribute. The translate position is driven by `inset-inline-start` calculated entirely in CSS using the same `--handle-size` var. Zero JS measurement, zero inline styles, RTL-automatic.

**Task:** Implement `--handle-size` CSS var toggling. Verify morph in a Storybook story with `transition-duration: 2000ms` override for slow-motion inspection. Add visual regression for 16dp / 24dp / 28dp states.

---

### 3. Dual icon slots — check + cross as separate controlled elements
**MUI:** MUI Switch has no built-in icon support. Adding icons requires custom rendering via `icon` / `checkedIcon` props on the underlying `SwitchBase`, which replaces the entire thumb SVG. There is no "both" variant with different icons for each state; you'd need to manage that yourself.

**kafUI wins:** `iconVariant="check"` and `iconVariant="both"` are first-class. The two icons (`.switch__icon--checked` / `.switch__icon--unchecked`) are separate DOM elements; CSS `opacity` transitions independently based on selected state. Custom icon override via `checkedIcon` / `uncheckedIcon` props slots in without touching CSS.

**Task:** Implement conditional rendering of icon elements in React. Verify that `iconVariant="check"` shows check on select and hides it on deselect via opacity (not display:none, to preserve transition). Verify `iconVariant="both"` shows cross when unselected.

---

### 4. Dark mode with zero JS — `color-scheme` flip drives everything
**MUI:** Dark mode requires `ThemeProvider` with `mode: "dark"`. The Switch has several color states (track, handle, border, icons) that are all Emotion-generated class names; switching modes re-runs all style calculations and re-injects CSS.

**kafUI wins:** All switch tokens (`--primary`, `--on-primary`, `--surface-container-highest`, `--outline`, etc.) are defined once as `light-dark()` values in the system token layer. `color-scheme: dark` on any ancestor flips all switch colors instantly — including mid-state (hover, focus, pressed) colors and disabled states.

**Task:** Verify in browser that adding `color-scheme: dark` to a parent `<div>` correctly flips all visual states of the switch. Add a dark-mode row to the Storybook story.

---

### 5. RTL thumb positioning is automatic — no transform math
**MUI:** MUI uses `transform: translateX(...)` for thumb positioning. In RTL contexts, the transform direction must be reversed (positive → negative). MUI achieves this via theme `direction: 'rtl'` + the RTL stylesheet transform, which is a global config and requires a separate RTL build step.

**kafUI wins:** `inset-inline-start` calculates thumb position relative to the inline-start edge. In LTR, OFF = inline-start; in RTL, the same property places OFF at the visual right (inline-start in RTL = the right edge). No transform direction flip, no RTL stylesheet, no global config — a single `dir="rtl"` on any ancestor works, including per-subtree.

**Task:** Add a Storybook story with `<div dir="rtl">` wrapper. Verify the OFF state is on the right, ON state is on the left, and the animation direction is correct in RTL.

---

### 6. `isReadOnly` — a real state, not a disabled hack
**MUI:** MUI `Switch` does not have a `readOnly` prop. To prevent interaction while keeping the switch focusable and visible, consumers must use custom CSS (`pointer-events: none` + `tabIndex={0}`) — which breaks keyboard semantics and ARIA.

**kafUI wins:** `isReadOnly` is a first-class prop. RAC maps it to `aria-readonly="true"` on the hidden input, keeping the element focusable, announcing its current state to screen readers, and preventing value changes. This is the correct pattern for a "view mode" form that shows current settings without allowing edits.

**Task:** Wire `isReadOnly` through to RAC `Switch`. Write a test confirming `aria-readonly="true"` is set and that pressing Space does not call `onChange`.

---

### 7. Zero bundle overhead — no Emotion, no StyleSheet
**MUI:** `@mui/material/Switch` includes Emotion dependencies; even with tree shaking, importing `Switch` pulls in the Emotion runtime. In a setting that lists 20+ switches, each instance triggers styled-component creation.

**kafUI wins:** The component emits static BEM class strings. All visual variation is via CSS attribute selectors targeting RAC's `data-*` render props. No runtime style injection. Bundle for `@kafui/react` Switch = ~1–2 kB JS + pre-built CSS.

**Task:** After implementation, measure bundle contribution using `bun build --analyze`. Verify no Emotion/styled-components imports anywhere in the dependency tree.

---

## Actionable TODO Checklist

### Setup
- [ ] Create `packages/react/src/components/switch/` directory
- [ ] Create `packages/styles/src/components/_switch.css` with `@layer kafui { … }` wrapper

### Styles (`@kafui/styles`)
- [ ] Declare component-internal vars on `.switch`: `--track-w`, `--track-h`, `--handle-rest`, `--handle-sel`, `--handle-press`, `--sl`, `--border-w`, `--sl-color`, `--handle-color`, `--handle-size`
- [ ] Map system tokens (unprefixed): `--primary`, `--on-primary`, `--on-primary-container`, `--surface-container-highest`, `--outline`, `--on-surface-variant`, `--on-surface`, `--surface`, `--corner-full`, `--body-large-font`, `--duration-short3`, `--easing-standard`, `--state-hover`, `--state-focus`, `--state-pressed`
- [ ] `.switch__track`: `width: var(--track-w); height: var(--track-h); border-radius: var(--corner-full); position: relative`
- [ ] `.switch__track-bg`: `position: absolute; inset: 0`; unselected: `--surface-container-highest` bg + 2dp `--outline` border; selected: `--primary` bg + no border; transitions
- [ ] `.switch__thumb-container`: 40×40dp; `inset-inline-start` driven by `--handle-size`; transition on `inset-inline-start`
- [ ] `.switch__state-layer`: `position: absolute; inset: 0; border-radius: 50%; pointer-events: none; opacity: 0`
- [ ] `.switch__handle`: size via `--handle-size`; color via `--handle-color`; transitions on width/height/background
- [ ] Selected state: `--sl-color: var(--primary)`, `--handle-color: var(--on-primary)`, `--handle-size: var(--handle-sel)`; translate thumb to end
- [ ] Hover / pressed handle color overrides via `[data-hovered]` / `[data-pressed]` data attrs
- [ ] `[data-pressed]` → `--handle-size: var(--handle-press)`
- [ ] State-layer opacity rules: `[data-hovered]` / `[data-focus-visible]` / `[data-pressed]`
- [ ] `.switch__icon--checked` / `.switch__icon--unchecked`: opacity transitions; colors `--on-primary-container` / `--on-surface-variant`
- [ ] `.switch--icon-both`: show unchecked icon when unselected
- [ ] Disabled states: 12% bg/border + 38% handle opacity via `color-mix()`
- [ ] `@media (prefers-reduced-motion: reduce)`: remove `inset-inline-start` transition; keep size + color transitions
- [ ] `.switch[data-label-placement="start"]`: `flex-direction: row-reverse`

### React (`@kafui/react`)
- [ ] `Switch.tsx` — wrap RAC `Switch`; render track/handle structure; hidden input inside via RAC
- [ ] Remove `label` prop — use `children` as label content (consistent with other kafUI controls)
- [ ] Map RAC render props (`isSelected`, `isDisabled`, `isHovered`, `isFocusVisible`, `isPressed`) to BEM modifier classes + `data-*` attributes on thumb-container
- [ ] Render `.switch__icon--checked` if `iconVariant !== 'none'`; render `.switch__icon--unchecked` if `iconVariant === 'both'`
- [ ] Accept `checkedIcon` / `uncheckedIcon` override slots; fall back to default SVGs
- [ ] `labelPlacement` → `data-label-placement` attribute on root (not BEM modifier)
- [ ] Wire `name`, `value`, `isReadOnly` to RAC Switch
- [ ] Forward `ref` to root element

### Tokens
- [ ] Confirm `--on-primary-container`, `--surface-container-highest`, `--outline` are defined
- [ ] Confirm `--duration-short3` (~150 ms) token exists
- [ ] Confirm `--easing-standard` and `--easing-emphasized-decelerate` / `--easing-emphasized-accelerate` are defined (for M3 Expressive thumb translate)

### Tests
- [ ] Bun test: renders `role="switch"` with `aria-checked="false"` initially
- [ ] Bun test: `Space` toggles `aria-checked`; `onChange` called with correct boolean
- [ ] Bun test: `isDisabled` → not interactive; `aria-disabled="true"`
- [ ] Bun test: `isReadOnly` → `aria-readonly="true"`; Space does not call `onChange`
- [ ] Bun test: `iconVariant="check"` renders `.switch__icon--checked` element
- [ ] Bun test: `iconVariant="both"` renders both `.switch__icon--checked` and `.switch__icon--unchecked`
- [ ] Visual regression: 16dp handle unselected, 24dp selected, 28dp pressed
- [ ] Visual regression: track border visible on unselected, absent on selected
- [ ] Visual regression: RTL layout (OFF on right, ON on left)

### Documentation
- [ ] Usage: simple toggle, with check icon, with both icons, with children label, disabled, readOnly
- [ ] Storybook story: interactive controls for `isSelected`, `iconVariant`, `labelPlacement`, `isDisabled`
- [ ] Storybook story: dark mode row
- [ ] Storybook story: RTL with `dir="rtl"` wrapper
- [ ] Document semantic distinction: Switch = immediate action vs. Checkbox = form-commit state
- [ ] Accessibility note: `role="switch"` vs MUI's `role="checkbox"` — why it matters
