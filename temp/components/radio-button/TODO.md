# Radio Button — TODO

## MUI Equivalent

`Radio` + `RadioGroup` + `FormLabel` + `FormControlLabel` + `FormControl` + `FormHelperText` from `@mui/material`. Six interacting components for what kafUI handles with two (`RadioGroup` + `Radio`).

---

## Beat-MUI Opportunities

### 1. Roving tabindex that actually works — RAC vs. MUI's hand-rolled version
**MUI:** MUI's `RadioGroup` uses native radio button keyboard behavior relying on browser-native roving tabindex for `<input type="radio">` elements. This works in simple cases but breaks subtly in complex layouts (portals, shadow DOM, conditionally rendered radios) where the browser's built-in grouping logic fails. No `aria-invalid` propagation to individual inputs.

**kafUI wins:** RAC implements roving tabindex in JavaScript using `useRadioGroup` / `useRadio`, which works regardless of DOM structure, portals, or rendering context. It also propagates `aria-invalid` from group to each individual radio input via context — a WCAG requirement that MUI misses.

**Task:** Use RAC `RadioGroup` + `Radio` primitives directly. Write a Bun test confirming `aria-invalid="true"` appears on each `<input>` inside an `isInvalid` group, not just on the group wrapper.

---

### 2. Error state is one prop on one component
**MUI:** Showing an error on a radio group requires: `<FormControl error>` → `<FormLabel>` → `<RadioGroup>` → `<FormControlLabel>` × N → `<FormHelperText>`. The `error` boolean on `FormControl` only changes colors of the label and helper text — the individual radio inputs do NOT receive `aria-invalid`. Screen reader users cannot detect the error without reading the helper text.

**kafUI wins:** `<RadioGroup isInvalid errorMessage="...">` propagates `isInvalid` via RAC context. CSS custom property `--sl-color: var(--error)` scoped to `.radio--error` rethemes every ring + dot with a single token swap. `<FieldError>` from RAC auto-links to the group via `aria-describedby`. Zero nested wrappers.

**Task:** Implement `.radio--error` modifier that switches `--sl-color`, `--primary` usage in rings, and dot color to `--error`. Confirm with axe scan that every child radio gets `aria-invalid`.

---

### 3. CSS-only dark mode — one `color-scheme` flip, zero JS
**MUI:** Toggling dark mode in MUI requires `ThemeProvider mode="dark"`, which triggers a full React reconciliation of the component tree. Inline styles and Emotion-generated class hashes are recalculated.

**kafUI wins:** All radio tokens are defined as `light-dark(…)` values. `color-scheme: dark` on any ancestor instantly flips rings, dots, labels, and state-layers — zero JS, zero re-render.

**Task:** Confirm all color tokens in `.radio` and `.radio-group` CSS use `var(--primary)`, `var(--on-surface-variant)`, etc., which resolve through the `light-dark()` pipeline from `_TOKENS.md`. Test by toggling `color-scheme` in DevTools.

---

### 4. Inner-dot spring animation — not just a fade
**MUI:** MUI's radio button selection is indicated by an SVG icon swap (an unfilled circle replaced by a filled one with a centered dot). There is no scale animation — the dot simply appears. In M3 terms, this misses the "spring" feel that differentiates the design.

**kafUI wins:** `transform: scale(0)` → `scale(1)` on `.radio__inner-dot` with `transition: transform var(--duration-short2) var(--easing-standard)` creates a satisfying pop-in. This is a pure CSS transition — no JS animation library, zero paint-invalidation of surrounding layout, and automatically disabled by `@media (prefers-reduced-motion: reduce)`.

**Task:** Implement the scale transform on `.radio__inner-dot`. Create a Storybook story with "slow motion" (CSS `transition-duration: 3000ms` override) to visually verify the spring. Add visual regression snapshot for the mid-transition state.

---

### 5. Zero label wrapper ceremony — children is the label
**MUI:** Every labeled radio requires `<FormControlLabel control={<Radio />} label="Option" />`. This is pure boilerplate — a component whose only job is to wrap another component with a string.

**kafUI wins:** `<Radio value="a">Option A</Radio>` — the children *are* the label. RAC's `Radio` root is a `<label>` element so the association is semantic by default. No `FormControlLabel`, no `control` prop, no extra import.

**Task:** Document this clearly in Storybook with a side-by-side "kafUI vs. MUI" code snippet. Emphasize the line-count difference in the usage example.

---

### 6. Per-item `labelPlacement` without CSS-in-JS
**MUI:** `FormControlLabel` has a `labelPlacement` prop (`"start" | "end" | "top" | "bottom"`). It works by setting inline styles or Emotion classes. Changing placement per-item in a group requires per-item `FormControlLabel` configuration.

**kafUI wins:** `labelPlacement` on each `<Radio>` emits a `data-label-placement` attribute. CSS rule `.radio[data-label-placement="start"] { flex-direction: row-reverse }` handles it. No JS, no per-item style injection, arbitrary per-item overrides are trivially achievable.

**Task:** Implement `data-label-placement` attribute forwarding in the React component. Add a logical `margin-inline-start` on `.radio__label` that automatically swaps side in RTL.

---

### 7. RTL is zero-config — no theme direction property
**MUI:** RTL for MUI radio buttons requires setting `theme.direction = 'rtl'` and using `jss-rtl` or `stylis-plugin-rtl`. This is a global configuration change; you can't have mixed-direction subtrees without a nested `ThemeProvider`.

**kafUI wins:** Logical CSS properties (`gap`, `margin-inline-start`, `flex-direction` already RTL-aware via `[data-label-placement]`) mean `dir="rtl"` on any ancestor instantly mirrors the layout. Per-subtree RTL is free.

**Task:** Add a Storybook story with `dir="rtl"` wrapper and verify label placement, ring/dot alignment, and error message alignment all mirror correctly.

---

## Actionable TODO Checklist

### Setup
- [ ] Create `packages/react/src/components/radio-button/` directory
- [ ] Create `packages/styles/src/components/_radio-button.css` with `@layer kafui { … }` wrapper

### Styles (`@kafui/styles`)
- [ ] Declare component-internal vars on `.radio`: `--ring`, `--dot`, `--sl`, `--dur`, `--ease`, `--sl-color`
- [ ] Map system tokens (unprefixed): `--primary`, `--on-surface-variant`, `--on-surface`, `--error`, `--body-large-font`, `--body-medium-font`, `--body-small-font`, `--corner-full`, `--duration-short2`, `--easing-standard`, `--state-hover`, `--state-focus`, `--state-pressed`
- [ ] `.radio-group`: `display: flex; flex-direction: column; gap: 4px`
- [ ] `.radio-group--horizontal .radio-group__items`: `flex-direction: row; flex-wrap: wrap; gap: 16px`
- [ ] `.radio__container`: 40×40dp, `border-radius: 50%`, `position: relative`, flex-center
- [ ] `.radio__state-layer`: `position: absolute; inset: 0; border-radius: 50%`; transitions via `data-*`
- [ ] `.radio__outer-circle`: 20×20dp; `border: 2px solid var(--on-surface-variant)`; `border-radius: 50%`; color transition `short2/standard`
- [ ] `.radio__inner-dot`: 10×10dp; `border-radius: 50%`; `transform: scale(0)` → `scale(1)` selected; transition
- [ ] `.radio--selected`: `--sl-color: var(--primary)`; outer circle `border-color: var(--primary)`; inner-dot `transform: scale(1)`
- [ ] `.radio--error`: `--sl-color: var(--error)`; outer circle `border-color: var(--error)`; inner-dot `background: var(--error)`
- [ ] `.radio--disabled`: 38% opacity via `color-mix`; `pointer-events: none; cursor: not-allowed`
- [ ] `[data-hovered]` / `[data-focus-visible]` / `[data-pressed]` state-layer opacity rules
- [ ] `.radio[data-label-placement="start"]`: `flex-direction: row-reverse`
- [ ] `@media (prefers-reduced-motion: reduce)`: remove transitions on inner-dot, outer-circle, state-layer
- [ ] Logical properties for all spacing (gap, margins)

### React (`@kafui/react`)
- [ ] `RadioGroup.tsx` — wrap RAC `RadioGroup`; render `.radio-group` BEM structure; include `<Label>`, `<Text slot="description">`, `<FieldError>`; add `--horizontal` modifier for `orientation="horizontal"`
- [ ] `Radio.tsx` — wrap RAC `Radio`; render `.radio__container` / `.radio__state-layer` / `.radio__outer-circle` / `.radio__inner-dot` + label; forward `data-label-placement` attribute
- [ ] Map RAC render props (`isSelected`, `isDisabled`, `isFocusVisible`, `isHovered`, `isPressed`, `isInvalid`) to BEM modifier classes + `data-*` attributes
- [ ] Apply `.radio--error` to each child when group `isInvalid` (via RAC context read in Radio component)
- [ ] Remove `label` prop from `Radio` — use `children` only
- [ ] `errorMessage` as `React.ReactNode` in `RadioGroup`
- [ ] Forward `ref` on both components
- [ ] Named exports from `@kafui/react` index

### Tokens
- [ ] Confirm `--primary`, `--on-surface-variant`, `--on-surface`, `--error` are in base palette
- [ ] Confirm `--state-hover`, `--state-focus`, `--state-pressed` (0.08 / 0.10 / 0.10) are defined
- [ ] Confirm `--duration-short2` and `--easing-standard` tokens exist

### Tests
- [ ] Bun test: group renders `role="radiogroup"` with accessible name from `label` prop
- [ ] Bun test: arrow keys move selection; only one radio is `aria-checked="true"` at a time
- [ ] Bun test: disabled radio is not focusable; not selectable via keyboard
- [ ] Bun test: `isInvalid` on group → every child radio `<input>` has `aria-invalid="true"`
- [ ] Bun test: `Space` selects focused, not-yet-selected radio
- [ ] Visual regression: selected / unselected / error / disabled states
- [ ] a11y: axe scan on vertical group + horizontal group + error state

### Documentation
- [ ] Usage examples: vertical, horizontal, error, disabled items, per-item labelPlacement
- [ ] Storybook story with interactive state controls for all variants
- [ ] Storybook story: `dir="rtl"` wrapper to validate RTL
- [ ] Side-by-side code comparison with MUI in docs (kafUI 2 imports vs MUI 6 imports)
