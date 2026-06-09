# Time Picker — TODO

## MUI equivalent

`TimePicker` from `@mui/x-date-pickers`. Requires `LocalizationProvider` + a date adapter (`AdapterDayjs` is most common). Separate `MobileTimePicker` and `DesktopTimePicker` for presentation modes. Both dial and input modes are implemented as a custom clock face; no shared primitive with any other library. Bundle: ~60 kB gz (`@mui/x-date-pickers`) + 15–40 kB adapter + Emotion runtime.

---

## Beat-MUI opportunities

### 1. Input mode: React Aria TimeField is best-in-class for spinbutton a11y

MUI's input mode for time is a custom `<input>` with manual ARIA management. RAC's `TimeField` provides `role="spinbutton"` per segment with `aria-valuemin`, `aria-valuemax`, `aria-valuenow`, `aria-valuetext`, keyboard increment/decrement, locale-aware formatting, and `@internationalized/date` `Time` type — all for free. kafUI's input mode is literally just `<TimeField>` with BEM classes applied to segments. **Actionable:** write a Bun test verifying that in input mode, the rendered DOM has three spinbutton elements with correct ARIA attributes without any manual ARIA code in `TimePicker.tsx`.

### 2. `hourCycle` is a first-class prop — no adapter config required

MUI infers 12/24h from the locale but overriding it requires configuring the date adapter, which is adapter-specific. kafUI: `hourCycle` is a direct prop, forwarded to `I18nProvider` scope wrapping the `TimeField`. **Actionable:** implement a thin `I18nProvider` scope inside `TimePicker` that merges the `hourCycle` override into the ambient locale — so `<TimePicker hourCycle={24} />` works regardless of the surrounding locale.

### 3. No adapter dependency — just `@internationalized/date`

MUI requires one of four date adapters at runtime (Dayjs, date-fns, Luxon, Moment). `@internationalized/date` is already a peer dep of RAC. **Actionable:** confirm `@internationalized/date` is in the shared monorepo deps and NOT duplicated; the time picker gets it as a transitive dep of RAC with no extra install.

### 4. `defaultMode` auto-selects dial vs input based on pointer media query

MUI always opens in clock view on mobile, input on desktop, but offers no override short of using `MobileTimePicker` vs `DesktopTimePicker`. kafUI's `defaultMode` auto-selects via `window.matchMedia('(pointer: coarse)').matches` and is overridable with a prop. **Actionable:** implement media query detection in a `useDefaultMode` hook that respects SSR (default to `'input'` on server, hydrate from media query on client without a layout shift — the mode is not visually intrinsic before interaction).

### 5. Dial ARIA via listbox/option — documented and testable pattern

MUI's clock dial has limited keyboard support and partial ARIA. kafUI's dial uses `role="listbox"` on the clock container with `role="option"` on each number, roving tabindex, full keyboard navigation (arrow keys, Home/End), and an `aria-live` region for drag announcements. **Actionable:** write a Bun test for the dial's keyboard navigation: arrow key cycles through numbers; Enter confirms; Escape closes without confirming; focus returns to trigger.

### 6. `minValue` / `maxValue` disable clock numbers in dial mode

MUI does not disable out-of-range hours/minutes on the clock face. kafUI marks out-of-range clock numbers as `aria-disabled="true"` with `pointer-events: none`, and the hand cannot be dragged to a disabled position. **Actionable:** implement the clamping logic in `useDial`; pass `minValue` / `maxValue` as `Time` objects and compare against each clock number's value.

### 7. CSS-only clock hand and selection indicator — no canvas, no SVG

MUI uses SVG or a CSS-with-JS approach for the clock hand. kafUI renders the clock hand as a `<div>` with `transform: rotate(var(--hand-angle)) translateY(-50%)` — the angle is a CSS custom property updated by `useDial`. The selection indicator is an absolutely-positioned circle moved to the active number via JS-computed `top`/`left`. No canvas, no SVG for the hand itself. **Actionable:** implement `useDial` so it only sets a single CSS custom property `--hand-angle` on the clock element; the CSS does the rest. This enables the `prefers-reduced-motion` override to simply set `transition: none`.

### 8. `aria-activedescendant` during drag — no focus thrash

MUI's dial does not announce the current value to screen readers during drag. kafUI uses `aria-activedescendant` on the `role="listbox"` element pointing to the currently highlighted option, and debounces `aria-live` announcements to one per ~15° of rotation. **Actionable:** test with a screen reader (NVDA + Chrome, VoiceOver + Safari) that values are announced during drag without flooding. Ensure the `aria-live` region is outside the clock element so announcements are not interrupted.

---

## Actionable checklist

### Setup
- [ ] Create `packages/react/src/components/time-picker/` directory.
- [ ] Create `packages/styles/src/components/_time-picker.css`.
- [ ] Confirm `@internationalized/date` in monorepo shared deps (transitive via RAC).

### Styles (`@kafui/styles`)
- [ ] All rules inside `@layer kafui { … }`.
- [ ] Component-scoped vars inside `.time-picker { --dial-size: 256px; --number-r-outer: 96px; --number-r-inner: 64px; --number-target: 40px; --hand-w: 2px; }`.
- [ ] `.time-picker` modal: `border-radius: var(--corner-extra-large)`; `background: var(--surface-container-high)` (NOT `surface-container-highest` — see spec correction); `min-inline-size: 280px`.
- [ ] `.time-picker__header`: `padding: 24px`; `background: var(--surface-container-highest)`.
- [ ] `.time-picker__display`: `display-large` typography; `display: flex; align-items: baseline; gap: 2px`.
- [ ] `.time-picker__hours`, `.time-picker__minutes`: `padding: 4px 8px; border-radius: var(--corner-small); cursor: pointer`; `[data-active]`: `background: var(--secondary-container); color: var(--on-secondary-container)`.
- [ ] `.time-picker__period`: `display: flex; flex-direction: column; border: 1px solid var(--outline); border-radius: var(--corner-small); overflow: hidden`; each button `padding: 8px`; selected: `background: var(--tertiary-container); color: var(--on-tertiary-container)`.
- [ ] `.time-picker__clock`: `inline-size: var(--dial-size); block-size: var(--dial-size); border-radius: var(--corner-full); background: var(--surface-container-highest); position: relative`.
- [ ] `.time-picker__clock-hand`: `position: absolute; inset-block-end: 50%; inset-inline-start: calc(50% - var(--hand-w) / 2); inline-size: var(--hand-w); block-size: calc(var(--number-r-outer) - 20px); transform-origin: bottom center; rotate: var(--hand-angle, 0deg); background: var(--primary)`. Transition wrapped in `@media (prefers-reduced-motion: no-preference) { transition: rotate var(--duration-short3) var(--easing-standard); }`.
- [ ] `.time-picker__clock-center-dot`: `8px` circle; `background: var(--primary)`; centered via `position: absolute; inset: 50%; translate: -50% -50%`.
- [ ] `.time-picker__clock-number`: `position: absolute; inline-size: var(--number-target); block-size: var(--number-target); border-radius: var(--corner-full); display: flex; align-items: center; justify-content: center`; positions set via JS `style.top`/`style.left` from angle calc. `body-large` type; `color: var(--on-surface)`. Inner ring (24h): `color: var(--on-surface-variant)`.
- [ ] `.time-picker__selection-indicator`: `var(--number-target)` circle; `background: var(--primary); position: absolute`; moves to active number via JS-updated `style.top`/`style.left`. Transition `position` wrapped in `@media (prefers-reduced-motion: no-preference)`.
- [ ] Active number text: `color: var(--on-primary)` when `.time-picker__selection-indicator` overlaps — use `z-index` + color on the number when `[aria-selected="true"]`.
- [ ] `.time-picker__footer`: `display: flex; justify-content: space-between; align-items: center; padding: 8px 6px`.
- [ ] `.time-picker__input-mode .time-picker__time-field`: large, centered; apply `--display-large-size` to segments.
- [ ] Dark mode: all `light-dark()` tokens from shared palette — no component-specific overrides needed.
- [ ] RTL: logical properties throughout; clock face not mirrored; action button order maintained.

### React (`@kafui/react`)
- [ ] `TimePicker.tsx` — top-level component; compose modal (RAC `ModalOverlay` + `Dialog` if `isModal`) + `TimePickerInner`.
- [ ] `TimePickerInner.tsx` — header + body + footer. State: `{ phase: 'hours'|'minutes', mode: 'dial'|'input' }`. Value state lifted from props or internal `defaultValue`.
- [ ] `useDefaultMode.ts` — detect `pointer: coarse` via `window.matchMedia`; return `'dial'` or `'input'`; SSR-safe (default `'input'`; update after mount).
- [ ] `TimePickerDial.tsx` — renders clock; accepts current `phase`, `hourCycle`, `value`, `onChange(value, phase)`, `minValue`, `maxValue`.
- [ ] `useDial.ts`:
  - [ ] Accepts `dialRef` (clock element ref), `phase`, `hourCycle`, `value`, `onChange`.
  - [ ] `pointerdown` → `dialRef.current.setPointerCapture(e.pointerId)`.
  - [ ] `pointermove` → compute `angle = Math.atan2(dy, dx)` from clock center; derive value from angle and ring (inner/outer for 24h); clamp to `minValue`/`maxValue`; call `onChange`.
  - [ ] Set `--hand-angle` on clock element via `style.setProperty`.
  - [ ] `pointerup` → confirm; if `phase === 'hours'`, call `onAdvancePhase('minutes')`.
  - [ ] Debounce `aria-live` announcements: only announce when value changes, max once per 15°.
  - [ ] Compute clock number positions: `top = 50% - r × cos(θ); left = 50% + r × sin(θ)` for each value.
  - [ ] Expose `handAngle`, `activeValue`, `numberPositions`.
- [ ] `aria-activedescendant` on `.time-picker__clock` pointing to current `[aria-selected="true"]` option.
- [ ] Roving tabindex on clock numbers: only the selected/focused number has `tabindex="0"`.
- [ ] Keyboard handler on clock: `←/→/↑/↓` cycles through numbers; `Home`/`End` jump to first/last; `Enter`/`Space` confirms.
- [ ] `TimePickerInput.tsx` — wraps RAC `TimeField`; applies `time-picker__segment--*` class names to `DateSegment` children via `className` render prop.
- [ ] `useHourCycleProvider.tsx` — wraps children in a scoped `I18nProvider` that overrides `hourCycle` while preserving ambient locale.
- [ ] Mode toggle: `<Button className="time-picker__mode-toggle">` with `<Icon name="schedule">` (dial→input) / `<Icon name="keyboard">` (input→dial).
- [ ] Confirm button: call `onChange(currentTime)` then `onOpenChange(false)`. Cancel: `onOpenChange(false)` only.
- [ ] `ref` forwarded to root dialog element.

### Tokens
- [ ] Confirm `--secondary-container`, `--on-secondary-container`, `--tertiary-container`, `--on-tertiary-container` are in the shared palette.
- [ ] Confirm `--corner-extra-large` (28 dp) is defined.
- [ ] Confirm `--duration-medium1` (~200 ms) and `--duration-short3` (~150 ms) are defined.

### Tests
- [ ] Unit: input mode renders `role="group"` with three `role="spinbutton"` elements.
- [ ] Unit: `↑/↓` increments/decrements hours segment value.
- [ ] Unit: `Tab` advances hours → minutes → AM/PM in 12h mode.
- [ ] Unit: `hourCycle={24}` renders no AM/PM segment.
- [ ] Unit: `useDial` — given a specific pointer angle on `pointermove`, derives the correct hour/minute value.
- [ ] Unit: dial auto-advances to minute phase after `pointerup` in hours phase.
- [ ] Unit: `minValue`/`maxValue` — numbers outside range have `aria-disabled="true"`.
- [ ] Unit: Escape closes modal without calling `onChange`.
- [ ] Unit: Confirm calls `onChange` with correct `Time` and closes.
- [ ] Unit: focus returns to trigger element after modal close.
- [ ] Unit: `useDefaultMode` returns `'input'` on SSR; updates to `'dial'` after mount if `pointer: coarse`.
- [ ] Visual regression: dial at 3 o'clock (hour), 45 (minute), hand + selection indicator position.
- [ ] a11y: axe scan — input mode, dial mode, modal open.
- [ ] a11y: keyboard nav in dial — arrow keys cycle; Enter advances phase; Escape cancels.

### Documentation / Storybook
- [ ] Story: modal with trigger button, 12h dial mode.
- [ ] Story: modal, 24h dial (inner + outer ring visible).
- [ ] Story: input mode only (`defaultMode="input"`, `allowModeToggle={false}`).
- [ ] Story: inline (non-modal).
- [ ] Story: disabled; `minValue`/`maxValue` (some clock numbers disabled).
- [ ] Story: reduced motion (hand snaps instantly).
- [ ] Story: dark mode.
- [ ] Docs: `Time` is timezone-agnostic by design.
- [ ] Docs: `hourCycle` override vs locale default.
- [ ] Docs: dial a11y approach (listbox pattern) with ARIA spec reference.
