# Text Field — TODO

## MUI equivalent

`@mui/material/TextField` composes `FormControl` + `InputLabel` + `OutlinedInput` / `FilledInput` + `FormHelperText`. Prop forwarding goes through three layers: `TextField` → `InputBase` → `<input>`, each layer accepting its own bag (`InputProps`, `InputLabelProps`, `FormHelperTextProps`, `inputProps`, `inputRef`). Theming requires `createTheme({ components: { MuiTextField: { styleOverrides: … } } })`. Runtime CSS-in-JS via Emotion.

---

## Beat-MUI opportunities

### 1. Kill the prop-bag soup — zero sub-component forwarding

MUI's flat `TextField` is a leaky abstraction over four internal components. Getting a `data-testid` on the actual `<input>` element requires `inputProps={{ 'data-testid': 'x' }}` (lowercase `inputProps`) while `InputProps` (capitalized) is for the Input wrapper — a perennial footgun. kafUI's `TextField` wraps RAC's context-propagation model: `Label`, `Input`, `FieldError`, and `Text` all read from the `TextField` context without any prop threading. **Actionable:** ensure the public API has exactly one flat props object and zero sub-component prop bags. Any prop that needs to reach the `<input>` element is passed directly on `TextField` and flows through RAC's `Input` slot.

### 2. Automatic aria wiring — never manage IDs by hand

MUI requires the developer to set matching `id` props on `TextField` and explicit `htmlFor` on `InputLabel`, or use `FormControl` to share an id context. `aria-describedby` on the `<input>` must be wired manually when using `FormHelperText` outside `TextField`. kafUI's RAC `TextField` + `FieldError` + `Text slot="description"` auto-generate and connect all IDs, auto-set `aria-invalid` from `isInvalid`, and include both description and error text in the `aria-describedby` chain. **Actionable:** write an integration test asserting the full `aria-describedby` chain is present with zero manually specified IDs.

### 3. CSS-first floating label — no Emotion, no JS animation

MUI's label float is a CSS-in-JS `transform` inside a `styled` component, gated by Emotion's class generation. kafUI's label float is a static CSS `transform` + `transition` driven by RAC's `data-focused` / `data-has-value` data attributes — zero runtime JS for the animation, zero Emotion in the bundle, zero SSR `CacheProvider` setup. **Actionable:** confirm the CSS transition covers all four triggers (`data-focused`, `data-has-value`, `data-placeholder-shown`, `isReadOnly` with value) in a single rule.

### 4. Better `errorMessage` API — validation result access

MUI: `error={bool}` + `helperText="..."` — developer must conditionally pass error text as `helperText` and toggle `error`. This breaks the separation between the helper text and the error text. kafUI: `errorMessage` accepts a `(ValidationResult) => string` function (RAC pattern), `isInvalid` is a boolean, and `description` is always the helper text. They never conflict. **Actionable:** also expose `validate` prop (RAC `TextField` `validate` function) so inline server-side validation messages can be returned as `ValidationResult` without extra state.

### 5. CSS-only outlined notch — no fieldset, no legend

MUI's outlined variant wraps content in `<fieldset>` and uses a visually-hidden `<legend>` (sized to the label width) to create the notch gap — two DOM nodes for one label, `min-inline-size: min-content` reset needed. kafUI uses three `<div>` segments with CSS borders and a single `--notch-w` variable. **Actionable:** the `ResizeObserver` that sets `--notch-w` must be in a custom hook (`useNotchWidth`) exported separately so tests can mock it, and an explicit `notchWidth` prop overrides it for SSR.

### 6. Prefix / suffix vs. adornments — semantic clarity

MUI mixes icon adornments and text adornments through the same `InputProps.startAdornment` / `endAdornment` slots (any `ReactNode`), requiring manual `InputAdornment` wrappers and correct `position` props. kafUI separates: `prefix`/`suffix` (string, static text) and `leadingIcon`/`trailingIcon` (icon name). Each maps to a distinct BEM element with distinct spacing. **Actionable:** confirm `prefix` and `suffix` are correctly `display: none` at rest (empty+unfocused) and become visible when `data-focused` or `data-has-value` — same trigger as label float.

### 7. `field-sizing: content` for textarea auto-resize — no shadow DOM

MUI auto-resizes `multiline` via a hidden "shadow" `<textarea>` in the DOM that mirrors content and reports scroll height — fragile and adds extra DOM nodes. kafUI uses `field-sizing: content` (CSS 2024, Chrome 123+, Firefox 130+) with a `ResizeObserver` JS fallback. **Actionable:** add a Bun test asserting the `field-sizing` CSS property is applied and that a `ResizeObserver` listener is attached only when `field-sizing` is unsupported (detected via `CSS.supports('field-sizing', 'content')`).

### 8. Zero runtime cost theming — one CSS variable per design decision

MUI `createTheme` for a custom text field color requires a deeply nested `components.MuiTextField.styleOverrides.root` with hard-coded selectors that break when MUI's internal class names change. kafUI: override `--primary` at a `.theme-brand` class and every focused element updates — no JS, no `createTheme`. **Actionable:** document the per-instance override pattern (inline `style={{ '--primary': '#0b57d0' }}`) and the global theme pattern (`.theme-brand { --primary: … }`) in Storybook.

---

## Actionable checklist

### Structure & markup
- [ ] Create `packages/react/src/components/text-field/TextField.tsx` wrapping RAC `TextField`, `Label`, `Input`/`TextArea`, `FieldError`, `Text slot="description"`.
- [ ] Implement three-div notch: `__notch-leading`, `__notch`, `__notch-trailing` inside `__outline` wrapper (outlined variant only).
- [ ] Implement `__active-indicator` inside `__container` (filled variant only).
- [ ] Wire `multiline` prop: render `TextArea` instead of `Input`; add `--multiline` modifier class; pass `rows`.
- [ ] Implement `useNotchWidth(labelRef)` hook: `ResizeObserver` on label element; returns `width` as string; set `--notch-w` via inline style on `__notch`. Accept `notchWidth` prop override.
- [ ] Add `data-has-value` attribute to root `TextField` wrapper whenever `value.length > 0` or `defaultValue` is non-empty (via RAC render prop or `onChange` tracking).
- [ ] Add `data-placeholder-shown` to root when `placeholder` prop is defined.
- [ ] Auto-swap `trailingIcon` to `"error"` when `isInvalid && !suppressErrorIcon`.

### Styles (`@kafui/styles`)
- [ ] All rules inside `@layer kafui { … }`.
- [ ] Component-scoped vars inside `.text-field { --h: 56px; --notch-w: 0px; --indicator-h: 1px; --label-dy: -22px; }`.
- [ ] `.text-field--filled`: `background-color: var(--surface-container-highest)`; `border-start-start-radius: var(--corner-extra-small)`; `border-start-end-radius: var(--corner-extra-small)`; `border-end-start-radius: 0`; `border-end-end-radius: 0`.
- [ ] `.text-field--outlined`: transparent background; `border-radius: var(--corner-extra-small)` on `__outline` segments.
- [ ] Label resting: centered vertically (`top: 50%; transform: translateY(-50%)`), `body-large` type, `color: var(--on-surface-variant)`.
- [ ] Label floating (wrap in `@media (prefers-reduced-motion: no-preference)` for transition only): `transform: translateY(var(--label-dy)) scale(0.75)`, `color: var(--primary)`.
- [ ] Label float triggers: `:is([data-focused], [data-has-value], [data-placeholder-shown]) .text-field__label { … }`.
- [ ] Notch: `.text-field__notch { width: var(--notch-w); }`. On float: `border-block-start: none`. On resting: `border-block-start: 1px solid var(--outline)`.
- [ ] Active indicator resting: `height: var(--indicator-h, 1px); background: var(--on-surface-variant)`. Focused: `--indicator-h: 2px; background: var(--primary)`.
- [ ] Outlined border resting: `1px solid var(--outline)`. Focused: `2px solid var(--primary)`.
- [ ] State layer on filled (`.state-layer`): `background: var(--on-surface); opacity: var(--state-hover)` on `[data-hovered]`; `background: var(--primary); opacity: var(--state-focus)` on `[data-focused]`.
- [ ] Error state (use `[data-invalid]` selector, not a class modifier): label `color: var(--error)`; indicator/outline `var(--error)`; supporting text `color: var(--error)`.
- [ ] Disabled state (`[data-disabled]`): `pointer-events: none`; container `color-mix(in srgb, var(--on-surface) 4%, transparent)`; indicator/outline `color-mix(in srgb, var(--on-surface) 12%, transparent)` dashed; label/text/prefix/suffix `color-mix(in srgb, var(--on-surface) 38%, transparent)`.
- [ ] Prefix/suffix: `display: none` → `display: flex` on `[data-focused]` or `[data-has-value]`; `color: var(--on-surface-variant)`.
- [ ] Leading icon offset: `.text-field--has-leading-icon .text-field__label, .text-field--has-leading-icon .text-field__input { padding-inline-start: 52px; }` (icon 24 dp + 12 dp padding each side).
- [ ] Trailing icon offset: `.text-field--has-trailing-icon .text-field__input { padding-inline-end: 52px; }`.
- [ ] `__supporting-text-row`: `display: flex; justify-content: space-between; padding-inline: 16px`.
- [ ] Character counter: `text-align: end; color: var(--on-surface-variant); font: var(--body-small-size) / var(--body-small-line-height)`.
- [ ] Multiline textarea: `field-sizing: content; resize: none; min-block-size: calc(var(--rows, 3) * 1lh)`.
- [ ] Dark mode: all via `light-dark()` in token definitions — no component-level override needed.
- [ ] RTL: all spacing logical (`padding-inline-*`, `margin-inline-*`, `inset-inline-*`); `text-align: end` on counter.

### Behavior / JS
- [ ] `useNotchWidth` hook: `ResizeObserver` on `labelRef`; set `--notch-w` on parent via `style` prop; memoize; clean up on unmount. Export separately for testing.
- [ ] Track `data-has-value`: add to root element via RAC render prop `(renderProps) => { … }` — fires on each render; no extra event listener.
- [ ] `data-placeholder-shown`: set via React `style` attribute or render prop when `placeholder !== undefined`.
- [ ] Character counter: `{value.length.toLocaleString(locale)} / {maxLength.toLocaleString(locale)}`. Render only when `maxLength` is set.
- [ ] `validate` prop passthrough to RAC `TextField` for built-in constraint validation.

### Accessibility
- [ ] Integration test: renders with zero manually specified `id` / `htmlFor` props, but the DOM has a matching `for`/`id` pair and correct `aria-describedby` value.
- [ ] Integration test: `aria-invalid="true"` present when `isInvalid`.
- [ ] Integration test: `aria-describedby` references both description-text id and error-text id when both rendered.
- [ ] Integration test: `aria-required="true"` present when `isRequired`.
- [ ] All icon elements: `aria-hidden="true"` in `<Icon>` (confirm not conditional).
- [ ] Character counter: `aria-hidden="true"`.
- [ ] Error trailing icon: `aria-hidden="true"`.

### Testing
- [ ] Unit: label association — `htmlFor`/`id` present without explicit props.
- [ ] Unit: `aria-invalid` set by `isInvalid`, cleared when `isInvalid` is false/absent.
- [ ] Unit: `aria-describedby` includes description id; on error, also includes error id.
- [ ] Unit: disabled has `pointer-events: none` and `aria-disabled="true"`.
- [ ] Unit: `data-has-value` present when controlled `value` is non-empty; absent when empty.
- [ ] Unit: `useNotchWidth` — ResizeObserver is attached; `--notch-w` updated on label resize; overridden by `notchWidth` prop.
- [ ] Unit: `field-sizing: content` applied to textarea; `ResizeObserver` fallback only when `CSS.supports('field-sizing','content')` is false.
- [ ] Visual regression: filled + outlined × enabled / hover / focus / error / disabled / filled-value, light + dark.
- [ ] Visual regression: RTL (icons/affixes/counter on correct sides).
- [ ] Visual regression: reduced-motion — label changes position instantly with no keyframe.
- [ ] a11y: axe-core zero violations for all states.
- [ ] Cross-browser: outlined notch gap in Safari, Firefox, Chrome.

### Storybook
- [ ] Story: empty (resting label), focused (floating label), filled+blurred.
- [ ] Story: error state with `errorMessage` + auto error icon.
- [ ] Story: disabled.
- [ ] Story: multiline (auto-resize).
- [ ] Story: prefix + suffix + leading + trailing icons.
- [ ] Story: character counter at limit.
- [ ] Story: RTL decorator (`dir="rtl"`).
- [ ] Story: dark mode (`color-scheme: dark`).
- [ ] Story: custom `--primary` override at instance scope via inline `style`.
