# Date Picker — TODO

## MUI equivalent

`@mui/x-date-pickers`: `DatePicker` (responsive), `DesktopDatePicker`, `MobileDatePicker`, `StaticDatePicker` — four separate components for presentation modes. Range picker (`DateRangePicker`) is in `@mui/x-date-pickers-pro` and **requires a paid MUI X commercial license**. All require `LocalizationProvider` + an adapter (`AdapterDayjs`, `AdapterDateFns`, `AdapterLuxon`, `AdapterMoment`) as a runtime dependency. Bundle: `@mui/x-date-pickers` ~80 kB gz + adapter 15–40 kB + Emotion runtime.

---

## Beat-MUI opportunities

### 1. Range picker is free — MUI charges for it

MUI's `DateRangePicker` is behind a paid Pro license (commercial use requires `@mui/x-date-pickers-pro`). kafUI's `DateRangePicker` is built on RAC `DateRangePicker` + `RangeCalendar` — both are MIT-licensed. **Actionable:** ensure `DateRangePicker` is in the same package as `DatePicker` with zero separate dependency, and call this out prominently in docs.

### 2. No adapter setup — @internationalized/date is already included

MUI requires choosing and installing a date adapter, wrapping the app in `LocalizationProvider`, and initializing the adapter. With `@internationalized/date` (a peer dep of RAC), no setup beyond `I18nProvider` is needed and non-Gregorian calendar support (Buddhist, Hebrew, Islamic, Japanese) is built in. **Actionable:** verify `@internationalized/date` is in the monorepo's shared `packages/` dependencies to avoid duplicate installs; document the `I18nProvider` setup as a one-time step.

### 3. Single `DatePicker` component — no mobile/desktop split

MUI exports four components for presentation modes. kafUI uses one `DatePicker` with a `mode` prop that controls whether the calendar appears as a `Popover` (docked) or inside a `Dialog` (modal). **Actionable:** implement mode switching internally; mode toggle between calendar and input within modal must be wired as internal state (not exposed as a prop) with an icon button in the dialog footer.

### 4. CalendarDate type — no timezone ambiguity by design

MUI's date pickers work with JS `Date` or adapter-specific types, all of which carry timezone information. Selecting "March 15, 2025" in a `DatePicker` can silently produce midnight UTC on one machine and midnight local time on another, causing off-by-one date bugs. `CalendarDate` is timezone-free by design. **Actionable:** add a type guard utility `toCalendarDate(date: Date): CalendarDate` in the package so apps migrating from MUI have an easy path.

### 5. Shared field anatomy — zero duplication vs text-field

MUI's date picker duplicates the text-field structure (InputLabel, OutlinedInput, FormHelperText) inside its own component. kafUI's date-picker field IS the text-field anatomy, with only `DateInput`/`DateSegment` replacing the `<input>` element. **Actionable:** share the `.text-field__container` CSS (or the shared field layer) with `.date-picker__field` by applying the same CSS classes; do not copy-paste text-field styles into date-picker styles. This is the central structural win — one field CSS file, two consumers.

### 6. Data-attribute-driven cell states — no className management

RAC `CalendarCell` exposes `data-selected`, `data-today`, `data-disabled`, `data-range-start`, `data-range-end`, `data-outside-visible-range`, etc. as data attributes on the DOM element. CSS targets these directly. No JS className toggling, no mapping from RAC state to BEM modifier. **Actionable:** audit which BEM modifier classes from the spec (`--today`, `--selected`, `--in-range`, `--range-start`, `--range-end`, `--preview`) are redundant with RAC data attributes; only emit explicit classes for states RAC does NOT provide (e.g. `--preview` which is RAC `data-range-selection-start` on `RangeCalendar` — verify exact RAC attribute name).

### 7. CSS-only range cell geometry — no inline style

MUI X's range picker injects inline styles for the half-cell background connecting range-start/end to the in-range strip. kafUI can implement this with CSS pseudo-elements: `::before` on the cell for the connecting rectangle, `::after` for the circular selection indicator — no inline style, no JS measuring. **Actionable:** prototype the range cell geometry with CSS only; document the pseudo-element approach in the component stylesheet. Ensure it works under `[dir=rtl]` with `inset-inline-start`/`end` positioning.

### 8. Bundle size — no Emotion, no adapter

MUI X date pickers pull in Emotion (~10 kB) + the adapter (15–40 kB depending on choice) + the pickers package itself (~80 kB). kafUI's total footprint for date picker: RAC (~15 kB gz) + `@internationalized/date` (~6 kB gz) + kafUI's component code. **Actionable:** add a bundle-size check in CI (using `bundlesize` or `size-limit`) that fails if the date-picker chunk exceeds 30 kB gz.

---

## Actionable checklist

### Dependencies
- [ ] Add `@internationalized/date` to the monorepo shared deps (or `packages/react/package.json`) — confirm not duplicated.
- [ ] Document `I18nProvider` as the one-time app-level setup in the "Getting Started" guide.
- [ ] Add `toCalendarDate(date: Date, timezone?: string): CalendarDate` migration utility to `@kafui/utils`.

### Styles (`@kafui/styles`)

#### Field (shared with text-field)
- [ ] All rules inside `@layer kafui { … }`.
- [ ] `.date-picker__field` — apply the same layout classes as `.text-field__container`. Do not copy styles; import / extend the text-field layer rules.
- [ ] `.date-picker__segment`: focused state `background: color-mix(in srgb, var(--primary) 12%, transparent)`; `border-radius: 2px`; `padding-inline: 2px`.
- [ ] `.date-picker__calendar-button`: `min-width: 48px; min-height: 48px`; icon color `var(--on-surface-variant)`.

#### Calendar
- [ ] `.date-picker__calendar`: `border-radius: var(--corner-extra-large)`; `background: var(--surface-container-high)`; `box-shadow: var(--elevation-2)` (docked) / `var(--elevation-3)` (modal).
- [ ] `.date-picker__calendar-header`: `display: flex; justify-content: space-between; align-items: center; block-size: var(--header-h, 56px); padding-inline: 4px`.
- [ ] `.date-picker__title`: `font: var(--headline-small-size)/var(--headline-small-line-height) var(--headline-small-font)`; hover state-layer.
- [ ] `.date-picker__nav-button`: `min-width: 48px; min-height: 48px`.
- [ ] `.date-picker__grid`: `display: grid; grid-template-columns: repeat(7, var(--cell-size, 40px)); gap: 0`.
- [ ] `.date-picker__column-header`: `font: var(--body-large-size)/var(--body-large-line-height) var(--body-large-font); color: var(--on-surface-variant); text-align: center`.
- [ ] `.date-picker__cell`: `inline-size: var(--cell-size, 40px); block-size: var(--cell-size, 40px); border-radius: var(--corner-full); display: flex; align-items: center; justify-content: center; font: var(--body-large-size)/var(--body-large-line-height) var(--body-large-font); cursor: pointer; position: relative`.
- [ ] `[data-today]`: label `color: var(--primary)`. Dot indicator: `::after { content:''; position:absolute; inset-block-end: 4px; inset-inline-start:50%; translate: -50% 0; inline-size:4px; block-size:4px; border-radius:50%; background:var(--primary) }`.
- [ ] `[data-selected]`: `background: var(--primary); color: var(--on-primary)`.
- [ ] `.date-picker__cell--in-range`: `background: var(--primary-container); color: var(--on-primary-container); border-radius: 0`.
- [ ] `.date-picker__cell--range-start`: circle overlay via `::after`; rectangular half-fill connecting to in-range strip via `::before { inset-inline-end:0; inset-inline-start:50%; background:var(--primary-container); border-radius:0 }`.
- [ ] `.date-picker__cell--range-end`: mirror of range-start (`inset-inline-start:0; inset-inline-end:50%`).
- [ ] `.date-picker__cell--preview`: `background: color-mix(in srgb, var(--primary) 38%, transparent)`.
- [ ] `[data-disabled]`, `.date-picker__cell--outside-month`: `opacity: 0.38; pointer-events: none`.
- [ ] RTL nav icon: `[dir=rtl] .date-picker__nav-button svg { transform: scaleX(-1); }` (or use `[dir=rtl]` logical icon approach).

#### Dialog / Modal
- [ ] `.date-picker__dialog`: `border-radius: var(--corner-extra-large)`; `background: var(--surface-container-high)`; `padding: 24px`.
- [ ] Scrim: `background: color-mix(in srgb, var(--scrim) 32%, transparent)`.
- [ ] `.date-picker__footer`: `display: flex; justify-content: flex-end; gap: 8px; padding-block-start: 8px`.
- [ ] `.date-picker__input-dialog`: heading "Enter date" in `label-large`; input `DateInput` styled like the field.

#### Dark mode / RTL / Motion
- [ ] All color tokens use `light-dark()` from the shared palette — no component-level `@media (prefers-color-scheme)` needed.
- [ ] All spacing via logical properties.
- [ ] `@media (prefers-reduced-motion: reduce)`: remove calendar slide/fade; remove popover transition.

### React components (`@kafui/react`)

- [ ] `DatePicker.tsx`:
  - [ ] Mode-switch logic: `mode="docked"` → RAC `Popover`; `mode="modal-calendar"` / `"modal-input"` → RAC `ModalOverlay` + `Dialog`.
  - [ ] Internal toggle state between calendar and input in modal (icon button in header/footer).
  - [ ] `showTodayButton` → footer button calling `onChange(today(getLocalTimeZone()))` then closing.
  - [ ] Cancel: close without `onChange`. OK: call `onChange` then close.
  - [ ] Forward `variant` to field container classes.
- [ ] `DateRangePicker.tsx`:
  - [ ] RAC `DateRangePicker` + `RangeCalendar`.
  - [ ] `visibleMonths` → `<RangeCalendar visibleDuration={{ months: visibleMonths }}>`.
  - [ ] Two field triggers (`__range-start-field`, `__range-end-field`).
- [ ] `Calendar.tsx` — standalone export wrapping RAC `Calendar` + `CalendarGrid`.
- [ ] `toCalendarDate` migration utility in `@kafui/utils`.
- [ ] Export `DatePicker`, `DateRangePicker`, `Calendar` from `packages/react/src/index.ts`.

### Testing
- [ ] Unit: segment keyboard navigation (↑/↓ changes value; ←/→ switches segment; 0–9 type; Backspace clears).
- [ ] Unit: calendar cell keyboard navigation (arrow keys; Page Up/Down for month; Ctrl+Page Up/Down for year).
- [ ] Unit: date selection fires `onChange` with `CalendarDate` (not `Date`).
- [ ] Unit: `minValue`/`maxValue` disables out-of-range cells (`data-disabled`).
- [ ] Unit: `isDateUnavailable` marks cells `data-unavailable`.
- [ ] Unit: range picker start/end selection; `--in-range` cells present between start and end.
- [ ] Unit: locale switch via `I18nProvider` changes day names and week start.
- [ ] Unit: RTL — prev/next buttons have swapped aria-labels.
- [ ] Unit: `showTodayButton` → today's date selected on click.
- [ ] Unit: modal cancel → `onChange` NOT called; modal OK → `onChange` called.
- [ ] Unit: `toCalendarDate(new Date('2025-03-15'))` returns `new CalendarDate(2025, 3, 15)`.
- [ ] a11y: axe scan — field open, calendar open, range calendar open.
- [ ] a11y: `aria-current="date"` on today cell.
- [ ] Bundle-size CI check: date-picker chunk ≤ 30 kB gz.
- [ ] Visual regression: all presentation modes × light/dark × LTR/RTL.

### Docs / Storybook
- [ ] Story: docked date picker.
- [ ] Story: modal calendar picker.
- [ ] Story: modal input picker + toggle to calendar.
- [ ] Story: date range picker (2-month view, then 1-month on mobile).
- [ ] Story: localization — Arabic RTL (`ar-SA`), Japanese (`ja-JP`), Hebrew (`he`).
- [ ] Story: `min`/`max`/`isDateUnavailable`.
- [ ] Story: disabled + read-only + error states.
- [ ] Story: `showTodayButton`.
- [ ] Docs: range picker has no Pro tier; comparison table vs MUI X.
- [ ] Docs: `toCalendarDate` migration guide for MUI X users.
