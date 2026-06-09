# Slider — TODO

## MUI Equivalent

`Slider` from `@mui/material`. One component, but with Emotion inline styling for thumb position, mark generation, and value label display — all computed in JS per render.

---

## Beat-MUI Opportunities

### 1. Range mode is a type, not a boolean — no footgun
**MUI:** Range mode in MUI is `value={[a, b]}` (the array implicitly enables range). But MUI has a subtle bug: if you pass `defaultValue={[0, 50]}` and then switch to `value={50}` (controlled scalar), it doesn't error — it silently stops using range mode. There is no TypeScript discriminant that prevents this mismatch.

**kafUI wins:** The `value` prop type is `number | [number, number]`. TypeScript narrows the `onChange` callback signature based on the input type — `onChange: (v: number) => void` for scalar, `onChange: (v: [number, number]) => void` for tuple. Passing a scalar `value` after a tuple `defaultValue` is a type error. The API is self-documenting and safe.

**Task:** Define a discriminated generic or function overload so `Slider<T extends number | [number, number]>` infers the correct `onChange` type. Write a type-level test (using `@ts-expect-error`) confirming the mismatch is caught at compile time.

---

### 2. CSS-driven thumb position — no inline style, no JS layout math
**MUI:** MUI computes thumb position as `transform: translateX(${thumbPosition}%)` or similar inline style calculated in JS per render. Any value update triggers a new inline style string, invalidating the element's style cache in the browser.

**kafUI wins:** Thumb position is expressed as a CSS custom property (`--slider-value-pct`) set once per value change. CSS `calc()` handles all positioning — the handle, the active track, the value label, and the gap are all derived from that one number. No `transform` string construction, no `translateX` math in JS.

**Task:** Set `--slider-value-pct`, `--slider-start-pct`, `--slider-end-pct` as inline custom properties on the track-wrapper from `state.getThumbPercent(index)`. Verify in DevTools that updating the slider changes only the custom property value — no `transform` inline styles anywhere.

---

### 3. Stop indicators with dual-color — a visible detail MUI omits
**MUI:** MUI supports `marks` (tick marks at custom positions) but uses a single color for all marks. In M3 2024, stops inside the active segment use `--on-primary` (white-on-primary) and stops outside use `--on-surface-variant`. This dual coloring is a meaningful visual cue that MUI entirely misses.

**kafUI wins:** Each `.slider__stop` element has an `--active` modifier class applied by the React component based on whether the stop's value falls within the current active range. CSS handles the color change — zero per-render style computation. The visual communicates "filled" vs "unfilled" progress on discrete tracks at a glance.

**Task:** In the React component, after computing stop positions, also compute which stops are within the active range and add `.slider__stop--active` accordingly. Update on every `onChange`. Write a visual regression snapshot for a discrete slider with mixed active/inactive stops.

---

### 4. Value label via CSS opacity — no portal, no JS position math
**MUI:** MUI's value label renders in a `Popper` component positioned absolutely above the thumb using `getBoundingClientRect()` calculations in JS. This means: a DOM query per frame during drag, possible position flash on first render, and dependency on a portal for z-index management.

**kafUI wins:** The `.slider__value-label` is a child of `.slider__thumb-container`, positioned with `position: absolute; bottom: calc(100% + 4px); left: 50%; transform: translateX(-50%)`. Visibility is driven by CSS `opacity` based on `[data-focus-visible]` / `[data-dragging]` data attributes — no JS position math, no portal, no `getBoundingClientRect` call.

**Task:** Implement the value label as a positioned child of thumb-container. Verify it appears above the thumb without overflow clipping (parent `overflow: hidden` must be avoided on track-wrapper). Test label visibility transitions for `showValueLabel="auto"`, `"always"`, and `"never"`.

---

### 5. M3 2024 handle morph and track gap — details MUI doesn't have
**MUI:** MUI's slider thumb does not grow on press. The track has no gap between the thumb and the active/inactive segments. These are the two biggest visual differences from M3 2024.

**kafUI wins:** Both features are pure CSS:
- Handle morph: `width/height` transition from `--handle-rest` (20dp) to `--handle-press` (26dp) on `[data-pressed]` / `[data-dragging]`.
- Track gap: `width: calc(var(--slider-value-pct) - var(--gap))` on the active track and `inset-inline-start: calc(var(--slider-value-pct) + var(--gap))` on the inactive track starting point.

Both features are disabled under `prefers-reduced-motion` (handle stays at 20dp; gap persists since it's non-motion).

**Task:** Verify gap is consistent across the full range (value at 0%, 50%, 100%) and does not cause the active track to go negative width at the extremes. Use `max(0%, calc(...))` as a guard.

---

### 6. `getValueLabel` powers both ARIA and display — one API, two benefits
**MUI:** MUI has `valueLabelFormat` (for display) and relies on `aria-valuetext` being automatically set to the same string. In practice, the display label and the ARIA text can diverge if you use `valueLabelDisplay: "off"` — the ARIA `aria-valuetext` still gets the formatted value, but the visual label doesn't appear. The two concerns are weakly coupled.

**kafUI wins:** `getValueLabel` is the single source of formatting truth. It populates both `aria-valuetext` on the hidden input AND the `.slider__value-label` text content. The consumer writes one formatter, and both ARIA and visual representations are guaranteed to match.

**Task:** Pass `getValueLabel(thumbValue)` to both `aria-valuetext` on `SliderThumb` and as the text content of `.slider__value-label`. Default to `String(value)` when not provided. Write a test confirming they match.

---

### 7. RTL is zero-config — `I18nProvider` + logical CSS
**MUI:** RTL for MUI Slider requires `theme.direction = 'rtl'` + `jss-rtl` / `stylis-plugin-rtl` to flip the track fill direction. The thumb position uses `left` percentage which must be transformed to `right` percentage in RTL — a non-trivial style override.

**kafUI wins:** RAC `Slider` reads direction from `I18nProvider` and inverts `getThumbPercent()` values accordingly. All CSS positioning uses `inset-inline-start` (logical). The track fill, thumb position, and stop indicators all mirror in RTL automatically. Per-subtree RTL is free.

**Task:** Wrap the Storybook story in `<I18nProvider locale="ar-SA">` (RTL locale) and verify that min is at inline-end, max at inline-start, and dragging right decrements the value.

---

### 8. Dark mode — zero JS, one `color-scheme` property
**MUI:** MUI Slider dark mode requires `ThemeProvider mode="dark"`. All the Emotion-generated class hashes for active track, inactive track, thumb, and value label are recalculated and re-injected.

**kafUI wins:** All slider tokens (`--primary`, `--secondary-container`, `--on-primary`, `--on-surface-variant`) resolve through `light-dark()` in the system token layer. `color-scheme: dark` flips all slider colors — active track, inactive track, handle, value label, and stops — instantly.

**Task:** Add a dark-mode row in Storybook for all slider variants. Verify programmatically that active track color differs between light and dark by toggling `color-scheme` in a test environment.

---

## Actionable TODO Checklist

### Setup
- [ ] Create `packages/react/src/components/slider/` directory
- [ ] Create `packages/styles/src/components/_slider.css` with `@layer kafui { … }` wrapper

### Styles (`@kafui/styles`)
- [ ] Declare component-internal vars on `.slider`: `--handle-rest`, `--handle-press`, `--sl`, `--track-h`, `--gap`, `--dur`, `--ease`
- [ ] Map system tokens (unprefixed): `--primary`, `--on-primary`, `--secondary-container`, `--on-surface-variant`, `--on-surface`, `--corner-full`, `--label-small-font`, `--body-large-font`, `--body-medium-font`, `--duration-short2`, `--easing-emphasized`, `--state-hover`, `--state-focus`, `--state-pressed`
- [ ] `.slider__track-wrapper`: `position: relative; height: var(--sl)`; flex/grid layout to vertically center track
- [ ] `.slider__track-inactive`: `height: var(--track-h); border-radius: var(--corner-full); background: var(--secondary-container)`
- [ ] `.slider__track-active`: `height: var(--track-h); border-radius: var(--corner-full); background: var(--primary)`; width/position from CSS vars + gap calc; guard with `max(0%, ...)`
- [ ] `.slider--range .slider__track-active`: two-thumb bounds from `--slider-start-pct` + `--slider-end-pct`
- [ ] `.slider__stop`: 4×4dp; `border-radius: 50%`; positioned via `--stop-pct` inline custom property; `transform: translateX(-50%)` for centering
- [ ] `.slider__stop--active`: `background: var(--on-primary)`; default stop: `background: var(--on-surface-variant)`
- [ ] `.slider__thumb-container`: `position: absolute`; centered on handle via `inset-inline-start` calc; 40×40dp
- [ ] `.slider__state-layer`: `position: absolute; inset: 0; border-radius: 50%; opacity: 0`; data-attr transitions
- [ ] `.slider__handle`: 20dp resting; transitions to 26dp on `[data-pressed]` / `[data-dragging]`
- [ ] `.slider__value-label`: `position: absolute; bottom: calc(100% + 4px)`; opacity 0 at rest; 1 on focus/drag; `always` modifier
- [ ] Disabled: 38% handle/active via `color-mix()`; 12% inactive track
- [ ] `@media (prefers-reduced-motion: reduce)`: disable handle size transition and value-label opacity transition
- [ ] `.slider--range` z-index: `--start` z-index 1, `--end` z-index 2

### React (`@kafui/react`)
- [ ] `Slider.tsx` — wrap RAC `Slider`; compose `SliderTrack` + `SliderThumb`(s) based on value type
- [ ] Infer range mode from `value` / `defaultValue` type (tuple detection); render two `SliderThumb` for range
- [ ] Compute `--slider-value-pct` / `--slider-start-pct` / `--slider-end-pct` from `state.getThumbPercent(index)` — set as inline custom properties on track-wrapper
- [ ] Compute stop positions: `(step * i - min) / (max - min) * 100`; render as `.slider__stop` elements; add `--stop-pct` inline custom property per stop
- [ ] Mark `.slider__stop--active` for stops within current active range
- [ ] Map RAC render props (`isPressed`, `isDragging`, `isFocusVisible`, `isHovered`) on `SliderThumb` to `data-*` attributes on thumb-container
- [ ] `showValueLabel` logic: `auto` → opacity via data attrs in CSS; `always` → add `.slider--value-label-always`; `never` → add `.slider--value-label-never`
- [ ] Pass `getValueLabel(value)` to both `aria-valuetext` on `SliderThumb` and as value-label text content
- [ ] Render `<output>` for `showOutput`; bind `htmlFor` to thumb `id`
- [ ] `origin="center"`: compute `--slider-active-start` = midpoint %, adjust active track calc
- [ ] `startThumbLabel` / `endThumbLabel` → `aria-label` on respective `SliderThumb`; defaults "Minimum" / "Maximum"
- [ ] Remove `isRange` boolean — infer from value shape
- [ ] Forward `ref` to root element

### Tokens
- [ ] Confirm `--secondary-container`, `--on-primary`, `--on-surface-variant` are in base palette
- [ ] Confirm `--easing-emphasized` exists (replaces erroneous `--easing-spring` from original spec)
- [ ] Confirm `--duration-short2` (~100 ms) is defined

### Tests
- [ ] Bun test: single slider renders `role="slider"` with `aria-valuenow`, `aria-valuemin`, `aria-valuemax`
- [ ] Bun test: arrow keys increment / decrement value; Home/End jump to min/max
- [ ] Bun test: Page Up / Page Down increment / decrement by 10 × step
- [ ] Bun test: `getValueLabel` result appears in `aria-valuetext` AND in value label text content
- [ ] Bun test: range slider — both thumbs independently navigable; values don't cross
- [ ] Bun test: `value={42}` (scalar) → `onChange` called with `number`; `value={[0,50]}` → `onChange` called with `[number, number]`
- [ ] Type test: mismatching scalar `value` with tuple `defaultValue` is a TypeScript error
- [ ] Bun test: disabled slider — no interaction; `aria-disabled="true"`
- [ ] Visual regression: discrete stops at correct positions; active vs inactive colors correct
- [ ] Visual regression: handle 20dp resting, 26dp pressed
- [ ] Visual regression: value label visibility in auto/always/never modes

### Documentation
- [ ] Usage: continuous single, discrete (step=10), range, centered origin
- [ ] Storybook story for each variant + all states (hover, focus, pressed, disabled)
- [ ] Storybook: dark mode row
- [ ] Storybook: RTL with `I18nProvider locale="ar-SA"`
- [ ] Document `getValueLabel` as the single formatting API (vs MUI's two-prop approach)
- [ ] Document range inference from value type (vs MUI's implicit array detection)
