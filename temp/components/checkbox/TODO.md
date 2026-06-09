# Checkbox — TODO

## MUI Equivalent

`Checkbox` + `FormControlLabel` (label wrapper) + `FormGroup` (grouping) + `FormControl` / `FormHelperText` (error/description) from `@mui/material`. Five separate components to assemble what kafUI does in two (`Checkbox` + `CheckboxGroup`).

---

## Beat-MUI Opportunities

### 1. Kill the FormControlLabel ceremony — label is a first-class prop/child
**MUI:** Every labeled checkbox requires wrapping with `<FormControlLabel control={<Checkbox />} label="..." />` — a separate component that only exists to attach a label. This is boilerplate that slows down every usage.

**kafUI wins:** Label is a natural child of `<Checkbox>`. `<Checkbox value="cheese">Cheese</Checkbox>` — done. No wrapper component needed, no import to remember, no `control` prop indirection. Compound API reads like HTML; the label is semantically linked via RAC's `<label>` root element automatically.

**Task:** Ensure generated `<label>` from RAC's `Checkbox` wraps both the visual box and the text child — zero extra markup in consumer code.

---

### 2. CSS-only dark mode — no ThemeProvider re-render
**MUI:** Dark mode requires `ThemeProvider` with `mode: 'dark'` prop, causing a top-level React re-render when the user switches themes. Every styled component recalculates. SSR dark-mode flash requires additional setup.

**kafUI wins:** `color-scheme: dark` on `:root` (or the nearest ancestor) instantly flips every `light-dark()` token — zero JS, zero re-render, zero SSR flash. One CSS property change drives the entire palette.

**Task:** Define all checkbox color tokens via `light-dark()`: e.g. `--primary: light-dark(var(--ref-primary-40), var(--ref-primary-80))`. Verify in browser with `@media (prefers-color-scheme: dark)` + manual `color-scheme` override.

---

### 3. Zero-runtime styling — no Emotion class churn
**MUI:** Each `<Checkbox>` invocation runs Emotion's style injection pipeline. On first render, Emotion serialises the sx/styled template, hashes it, and injects a `<style>` tag. In large lists of checkboxes (e.g. a 200-row data grid), this creates measurable GC pressure.

**kafUI wins:** BEM classes are static strings resolved at authoring time. Every checkbox with the same state hits the same pre-parsed CSS rule — the browser's class-lookup is O(1). No runtime style injection, no cache invalidation, no `ServerStyleSheet` for SSR.

**Task:** Ensure the React component emits only BEM class strings and `data-*` attributes — no `style` prop, no CSS-in-JS, no inline color values.

---

### 4. `CheckboxGroup` owns error state — no FormControl nesting required
**MUI:** Error state for a checkbox group requires: `<FormControl error>` → `<FormGroup>` → `<FormControlLabel>` per item → `<FormHelperText>` for the message. Four nesting levels, and `error` on `FormControl` only changes the label/helper color — the individual checkboxes are not marked `aria-invalid`.

**kafUI wins:** `<CheckboxGroup isInvalid errorMessage="...">` propagates `isInvalid` via RAC context to each child `Checkbox`, setting `aria-invalid="true"` on every `<input>` in the group. One prop, full ARIA compliance, no wrapper cascade.

**Task:** Wire `isInvalid` through RAC `CheckboxGroup` context. Render `<FieldError>` from RAC (auto-linked via `aria-describedby`). Confirm with axe that each radio input in an invalid group shows `aria-invalid="true"`.

---

### 5. Indeterminate state is a first-class React prop — no DOM imperative needed
**MUI:** Setting `indeterminate` on an MUI `Checkbox` works, but because `<input type="checkbox">` has no `indeterminate` HTML attribute, MUI sets it imperatively via a `ref` + `useEffect`. This means the indeterminate state is invisible to ARIA until after mount and can misfire during SSR.

**kafUI wins:** RAC `Checkbox` uses `aria-checked="mixed"` declaratively from `isIndeterminate` — the ARIA state is correct at first paint, including SSR. No imperative DOM mutation, no `useEffect` ref gymnastics.

**Task:** Pass `isIndeterminate` directly to RAC `<Checkbox isIndeterminate>`. Confirm `aria-checked="mixed"` in rendered HTML. Add a Bun test asserting the attribute value before any user interaction.

---

### 6. Pixel-perfect M3 check animation via SVG stroke-dashoffset
**MUI:** MUI's checkmark is a static SVG icon that fades/scales in. There is no path-drawing animation — the check simply appears with an opacity transition.

**kafUI wins:** M3 specifies a stroke-drawing animation (`stroke-dashoffset` from `24` → `0`). The check path visually "draws itself" on selection, giving tactile confirmation of the action. This is a zero-JS CSS technique — `stroke-dasharray` + `stroke-dashoffset` transition.

**Task:** Implement checkmark SVG with `stroke-dasharray: 24; stroke-dashoffset: 24` on unselected, transitioning to `stroke-dashoffset: 0` on `.checkbox--selected`. Verify `@media (prefers-reduced-motion: reduce)` disables the path draw and falls back to instant opacity.

---

### 7. Touch target exceeds 48dp without padding hacks
**MUI:** MUI achieves its touch target via a `TouchRipple` overlay and `padding`; the visual size and touch size are conflated.

**kafUI wins:** The `.checkbox__state-layer` is a separate 40×40dp circle element (centered on the 18×18dp box via negative `inset`). The clickable `<label>` root extends naturally over the label text too, achieving well-over-48dp total tap area. No padding inflation, no z-index stacking hacks.

**Task:** Confirm state-layer `inset: calc((40px - 18px) / -2)` (≈ −11px) in CSS. Verify no overflow clipping from ancestor containers by testing in a constrained list layout.

---

### 8. RTL is automatic — no theme direction re-config
**MUI:** RTL support in MUI requires setting `theme.direction = 'rtl'` and often importing `jss-rtl` or `stylis-plugin-rtl` to auto-flip physical CSS properties. A direction change requires re-creating the theme and re-rendering.

**kafUI wins:** All spacing uses CSS logical properties (`margin-inline-start`, `padding-inline`, `inset-inline-*`). Flipping `dir="rtl"` on any ancestor instantly reorders the label and control — zero JS, zero theme change, works per-subtree.

**Task:** Verify that `<Checkbox labelPlacement="end">` places label to the right in LTR and to the left in RTL by testing with `<div dir="rtl">` ancestor. Also verify `labelPlacement="start"` reversal works in both directions.

---

## Actionable TODO Checklist

### Styles (`@kafui/styles`)
- [ ] Create `packages/styles/src/components/_checkbox.css` with `@layer kafui { … }` wrapper
- [ ] Declare component-internal vars on `.checkbox` block: `--box`, `--sl`, `--sl-offset`, `--radius`, `--dur`, `--ease`
- [ ] Map system tokens (unprefixed): `--primary`, `--on-primary`, `--on-surface-variant`, `--error`, `--on-error`, `--surface`, `--on-surface`, `--error-container`, `--state-hover`, `--state-focus`, `--state-pressed`, `--state-dragged`, `--corner-extra-small`, `--duration-short3`, `--easing-standard`, `--label-large-font`
- [ ] Implement 18×18dp container with `border-radius: var(--corner-extra-small)`
- [ ] Implement 40×40dp state-layer via `.checkbox__state-layer` absolutely positioned with `inset: var(--sl-offset)` (≈ −11px); `border-radius: 50%`
- [ ] Implement checkmark SVG animation: `stroke-dashoffset` path draw for check; separate `--` width transition for dash (indeterminate)
- [ ] Color transitions for `.checkbox--selected`, `.checkbox--indeterminate`, `.checkbox--error` using system tokens
- [ ] Interaction states via RAC `data-hovered` / `data-focus-visible` / `data-pressed` attrs on root
- [ ] Disabled states using `color-mix(in srgb, var(--on-surface) 38%, transparent)` (no JS opacity)
- [ ] `@media (prefers-reduced-motion: reduce)` block
- [ ] Logical properties throughout: `margin-inline-start` between container and label; no physical left/right
- [ ] Export from `packages/styles/src/index.css`

### React component (`@kafui/react`)
- [ ] `packages/react/src/components/Checkbox/Checkbox.tsx`: wrap RAC `<Checkbox>` — emit BEM classes from render props (`isSelected`, `isIndeterminate`, `isDisabled`, `isFocusVisible`, `isHovered`, `isPressed`, `isInvalid`)
- [ ] `packages/react/src/components/Checkbox/CheckboxGroup.tsx`: wrap RAC `<CheckboxGroup>` — include `<Label>`, `<Text slot="description">`, `<FieldError>`; add `.checkbox-group--horizontal` modifier for `orientation="horizontal"`
- [ ] Emit `data-label-placement` on root element (not a BEM modifier) for CSS-only label reorder
- [ ] `errorMessage` accepts `React.ReactNode`; render inside `<FieldError>` slot
- [ ] Forward `ref` on both `Checkbox` and `CheckboxGroup`
- [ ] Export named exports from `packages/react/src/index.ts`

### Testing
- [ ] Unit: selected / unselected / indeterminate state transitions
- [ ] Unit: `isInvalid` → `aria-invalid="true"` on `<input>`; `<FieldError>` renders error message
- [ ] Unit: `isIndeterminate` → `aria-checked="mixed"` (not after mount; present in initial render output)
- [ ] Unit: `CheckboxGroup` propagates `isDisabled` to all children via context
- [ ] Unit: `CheckboxGroup` `onChange` fires with correct `string[]` value
- [ ] Unit: `CheckboxGroup isInvalid` propagates `aria-invalid` to each child checkbox
- [ ] a11y: axe scan on all variants (selected, unselected, indeterminate, error, disabled) in isolation and in group
- [ ] Visual regression: snapshot each variant in light + dark

### Docs / Storybook
- [ ] Story: all states side-by-side (selected, unselected, indeterminate, error, disabled)
- [ ] Story: `CheckboxGroup` — vertical + horizontal
- [ ] Story: select-all (indeterminate) parent wired manually
- [ ] Story: error state with message
- [ ] Story: RTL layout (`dir="rtl"` ancestor)
- [ ] Story: `labelPlacement="start"`
- [ ] Storybook control for `iconVariant` (N/A checkbox, but confirm dark-mode toggle wired)
