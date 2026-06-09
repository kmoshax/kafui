# Date Picker

M3 category: **Date & Time Pickers**. Lets users select a specific date or date range. M3 defines four presentation modes: docked calendar, modal calendar, modal input, and range picker — all built on the same calendar grid logic and the same field anatomy as `text-field`.

---

## Anatomy / Parts

### Shared field anatomy (reuses text-field field pattern)

The date-picker input field follows the **same anatomy** as `text-field` (filled or outlined variant, floating label, supporting text, error text). See `text-field/SPEC.md` for the shared field anatomy. The only additions specific to date-picker are:

- `__date-input` — wraps RAC `DateInput` with its editable `DateSegment` children (replaces `__input`).
- `__calendar-button` — icon button inside the field that opens the calendar popover/dialog.

```
.date-picker                         Root component element.
.date-picker__field                  Field trigger — uses text-field container anatomy.
  .date-picker__date-input           RAC DateInput (replaces __input in text-field).
    .date-picker__segment            Each editable segment (month / day / year).
  .date-picker__calendar-button      Icon button to open calendar.
.date-picker__popover                Non-modal popover (docked mode).
.date-picker__dialog                 Modal dialog container (modal modes).
  .date-picker__calendar             Calendar surface.
    .date-picker__calendar-header    Header row: prev button, title, next button.
      .date-picker__nav-button       Prev / next month buttons.
      .date-picker__title            Month + year heading; tappable to drill into month/year picker.
    .date-picker__grid               7-column day grid.
      .date-picker__column-header    Day-of-week headers (Sun–Sat or locale equivalent).
      .date-picker__cell             Individual day cell.
  .date-picker__footer               Cancel + OK action row (modal modes); optional Today shortcut.
.date-picker__input-dialog           Modal input mode: "Enter date" heading + text input.
```

### Date range additions

```
.date-picker--range                  Modifier on root for range mode.
.date-picker__range-start-field      Start-date field trigger.
.date-picker__range-end-field        End-date field trigger.
.date-picker__cell--range-start      First selected day.
.date-picker__cell--range-end        Last selected day.
.date-picker__cell--in-range         Days between start and end.
.date-picker__cell--preview          Hover-preview of prospective range end.
```

---

## Variants

### 1. Docked Date Picker (`mode="docked"`)

Calendar opens inline below the text field via a non-modal `Popover`. Suitable for desktop layouts with sufficient vertical space. No scrim; focus stays in document flow. The `Popover` is not modal.

### 2. Modal Date Picker — Calendar (`mode="modal-calendar"`)

Calendar opens in a RAC `Dialog` (modal) with a scrim. Header shows the selected date. Footer: **Cancel** and **OK** buttons. Default for mobile. Modal calendar can toggle to input mode via an edit icon button.

### 3. Modal Date Input (`mode="modal-input"`)

Dialog opens with a text input field (`DateInput` with spinbutton segments) and "Enter date" heading. Useful for keyboard-first users. Has a calendar icon button to toggle back to calendar mode. Footer: Cancel / OK.

### 4. Date Range Picker (`isRange`)

Selects start and end dates. Calendar shows up to two months side by side. Uses RAC `DateRangePicker` + `RangeCalendar`. Can be docked or modal. The range picker is **free in kafUI** — there is no Pro tier.

---

## States

### Field / Segments

- Unfocused, focused (segment `background: primary @ 12%`), filled (segments display value), error, disabled.
- Each `DateSegment` (`role="spinbutton"`) supports type-to-edit, arrow up/down to increment/decrement.
- Segment placeholder text (e.g. "MM") color: `on-surface-variant`.
- Error state: field border switches to `--error`; `FieldError` text shown below.

### Calendar cells

| State | Visual |
|---|---|
| Default | Label `on-surface-variant`; no fill |
| Hover | State-layer `on-surface-variant` at `--state-hover` (8%) |
| Focus | State-layer at `--state-focus` (10%) + focus-visible outline |
| Pressed | State-layer at `--state-pressed` (10%) |
| Today (unselected) | Label `primary`; 2 dp dot indicator below label |
| Selected | Fill `primary`; label `on-primary` |
| Range start/end | Fill `primary` (full circle); label `on-primary`; adjacent range bg extends to half-cell |
| In-range | Fill `primary-container`; label `on-primary-container`; no border-radius on adjacent sides |
| Selection preview | Fill `primary` at 38%; label `primary` |
| Disabled | `opacity: 0.38`; `pointer-events: none` |
| Outside current month | `opacity: 0.38`; optionally hidden |

### Range cell geometry

Range start/end cells have a circular overlay (`border-radius: var(--corner-full)`) on top of a rectangular half-cell background that connects to the in-range strip. Achieved with two pseudo-elements or a `background-image: linear-gradient` that fills only the connecting half.

---

## Design Tokens

All system tokens referenced by unprefixed CSS custom property names. Component-internal variables inside `.date-picker { … }`.

### Color

| Role | CSS var | Usage |
|---|---|---|
| primary | `--primary` | Selected cell fill; focused segment bg; focused field border; confirm button |
| on-primary | `--on-primary` | Selected cell label |
| primary-container | `--primary-container` | In-range fill |
| on-primary-container | `--on-primary-container` | In-range label |
| surface-container-highest | `--surface-container-highest` | Field background (filled variant) |
| surface-container-high | `--surface-container-high` | Calendar surface; dialog surface |
| on-surface | `--on-surface` | Field text; unselected day label |
| on-surface-variant | `--on-surface-variant` | Field label; segment placeholder; nav icon; column headers |
| outline | `--outline` | Field border (outlined variant); segment dividers |
| error | `--error` | Invalid field border; error text |
| scrim | `--scrim` | Modal overlay at 32% opacity |

### Shape

| Part | CSS var |
|---|---|
| Calendar container | `--corner-extra-large` (28 dp) |
| Day cell | `--corner-full` (circle) |
| Input dialog | `--corner-extra-large` |
| Text field | `--corner-extra-small` (field top corners for filled) |

```css
@layer kafui {
  .date-picker {
    --cell-size: 40px;       /* day cell diameter */
    --grid-cols: 7;          /* always 7 */
    --header-h: 56px;        /* calendar header height */
  }
}
```

### Typography

| Role | CSS var | Usage |
|---|---|---|
| headline-small | `--headline-small-size` etc. | Month/year title in calendar header |
| body-large | `--body-large-size` etc. | Day-of-week column headers; day cell labels; field segments |
| label-large | `--label-large-size` etc. | "Enter date" dialog title; footer action buttons |
| body-small | `--body-small-size` etc. | Field supporting text / error text (inherited from text-field) |

### Elevation

| Surface | CSS var |
|---|---|
| Calendar popover (docked) | `--elevation-2` |
| Modal dialog | `--elevation-3` |

### Motion

| Transition | CSS vars |
|---|---|
| Calendar open/close | `--duration-medium2` + `--easing-emphasized-decelerate` |
| Month slide (prev/next) | `--duration-short3` + `--easing-standard` |
| Cell hover state-layer | `--duration-short2` |

---

## Interaction & Accessibility

### React Aria Primitives

| Purpose | RAC Primitive |
|---|---|
| Single date field | `DatePicker` > `Group` > `DateInput` > `DateSegment` |
| Docked calendar | `DatePicker` with `Popover` (not modal) |
| Modal calendar | `DatePicker` with `ModalOverlay` + `Dialog` |
| Modal date input | `DatePicker` with `ModalOverlay` + `Dialog` + `DateInput` segments |
| Range picker | `DateRangePicker` > `RangeCalendar` |
| Standalone calendar | `Calendar` |
| Range calendar | `RangeCalendar` |

**`@internationalized/date`** — RAC `DatePicker`/`Calendar` work with `CalendarDate` / `CalendarDateTime` / `ZonedDateTime`. Locale-aware formatting via `I18nProvider`. Supports Gregorian, Buddhist, Hebrew, Islamic, Japanese calendars out of the box — zero adapter setup.

### ARIA Roles

- `DateInput` → `role="group"` with `aria-label` / `aria-labelledby`.
- `DateSegment` → `role="spinbutton"` with `aria-label`, `aria-valuemin`, `aria-valuemax`, `aria-valuenow`, `aria-valuetext`.
- `Calendar` → contains `role="grid"` (month grid) with `aria-label="Month Year"`.
- `CalendarGridRow` → `role="row"`.
- Column headers → `role="columnheader"`.
- Day cells → `role="gridcell"` with `aria-label` (localized date string), `aria-selected`, `aria-disabled`.
- Today cell → `aria-current="date"`.
- Nav buttons → `role="button"` with localized `aria-label="Previous month"` / `"Next month"`.
- Dialog → `role="dialog"`, `aria-modal="true"`, `aria-label="Choose date"` (or `aria-labelledby` the dialog heading).

### Keyboard Navigation

| Context | Key | Action |
|---|---|---|
| DateField segments | `←/→` | Move between segments |
| DateField segments | `↑/↓` | Increment/decrement segment value |
| DateField segments | `0–9` | Type value |
| DateField segments | `Backspace` | Clear segment |
| Calendar button | `Space`/`Enter` | Open calendar |
| Calendar | `←/→/↑/↓` | Move focus by day/week |
| Calendar | `Page Up/Down` | Previous/next month |
| Calendar | `Ctrl+Page Up/Down` | Previous/next year |
| Calendar | `Home/End` | First/last day of week |
| Calendar | `Space`/`Enter` | Select date |
| Calendar | `Escape` | Close without selection |
| Modal footer | `Tab` | Move to Cancel/OK buttons |
| Modal footer | `Enter` on OK | Confirm and close |

### Locale & i18n

- Wrap app with `<I18nProvider locale="ar-SA">`.
- Calendar grid direction flips for RTL locales via RAC internals.
- Day/month names via `Intl.DateTimeFormat`.
- Week start day from locale.
- Date display format in segments follows locale.
- `minValue`/`maxValue` accept `CalendarDate` — no timezone ambiguity.

### RTL

- `dir` propagated to grid; logical `padding-inline` on cells.
- Nav button icons: RAC handles aria-label swap; CSS `transform: scaleX(-1)` on icon inside `[dir=rtl] .date-picker__nav-button` for visual directional icons, or use `[dir=rtl]` + `:dir(rtl)` selectors.

### Reduced Motion

```css
@layer kafui {
  @media (prefers-reduced-motion: reduce) {
    .date-picker__calendar { transition: none; }
    .date-picker__grid { animation: none; }
    .date-picker__popover { transition: none; }
  }
}
```

---

## Proposed kafUI React API

```tsx
import { CalendarDate, parseDate, today, getLocalTimeZone } from "@internationalized/date";

// ── Shared types ──────────────────────────────────────────────
type DatePickerMode = "docked" | "modal-calendar" | "modal-input";

// ── Single Date Picker ────────────────────────────────────────
interface DatePickerProps {
  /**
   * "docked": calendar inline below field (Popover).
   * "modal-calendar": dialog with calendar (default).
   * "modal-input": dialog with text entry + toggle to calendar.
   */
  mode?: DatePickerMode;
  /** Variant forwarded to the field. Default: 'filled'. */
  variant?: "filled" | "outlined";
  label?: string;
  description?: string;
  errorMessage?: string | ((v: ValidationResult) => string);
  isRequired?: boolean;
  isDisabled?: boolean;
  isReadOnly?: boolean;
  isInvalid?: boolean;
  value?: CalendarDate | null;
  defaultValue?: CalendarDate;
  onChange?: (date: CalendarDate | null) => void;
  minValue?: CalendarDate;
  maxValue?: CalendarDate;
  isDateUnavailable?: (date: CalendarDate) => boolean;
  /** Show a "Today" shortcut button in the calendar footer. Default: false */
  showTodayButton?: boolean;
  /** Locale string. If omitted, inherits from nearest I18nProvider. */
  locale?: string;
  className?: string;
}

// ── Date Range Picker ─────────────────────────────────────────
interface DateRangePickerProps {
  mode?: "docked" | "modal-calendar";
  variant?: "filled" | "outlined";
  startLabel?: string;
  endLabel?: string;
  description?: string;
  errorMessage?: string | ((v: ValidationResult) => string);
  isRequired?: boolean;
  isDisabled?: boolean;
  isReadOnly?: boolean;
  isInvalid?: boolean;
  value?: { start: CalendarDate; end: CalendarDate } | null;
  defaultValue?: { start: CalendarDate; end: CalendarDate };
  onChange?: (range: { start: CalendarDate; end: CalendarDate } | null) => void;
  minValue?: CalendarDate;
  maxValue?: CalendarDate;
  isDateUnavailable?: (date: CalendarDate) => boolean;
  /**
   * Number of months visible in the calendar simultaneously.
   * Default: 2 on desktop, 1 on mobile — consumer controls this, not CSS media query.
   */
  visibleMonths?: 1 | 2;
  className?: string;
}

// ── Standalone Calendar ───────────────────────────────────────
interface CalendarProps {
  value?: CalendarDate | null;
  defaultValue?: CalendarDate;
  onChange?: (date: CalendarDate) => void;
  minValue?: CalendarDate;
  maxValue?: CalendarDate;
  isDateUnavailable?: (date: CalendarDate) => boolean;
  "aria-label"?: string;
  "aria-labelledby"?: string;
  className?: string;
}

// ── Usage examples ────────────────────────────────────────────
// Docked, filled:
<DatePicker
  mode="docked"
  label="Check-in date"
  value={date}
  onChange={setDate}
  minValue={today(getLocalTimeZone())}
/>

// Modal range picker, outlined:
<DateRangePicker
  mode="modal-calendar"
  variant="outlined"
  startLabel="Departure"
  endLabel="Return"
  value={range}
  onChange={setRange}
  visibleMonths={2}
/>

// Modal input picker:
<DatePicker
  mode="modal-input"
  label="Date of birth"
  onChange={setDob}
/>
```

### BEM modifier classes emitted

```
.date-picker--docked
.date-picker--modal-calendar
.date-picker--modal-input
.date-picker--range
.date-picker--filled / --outlined    (field variant)

.date-picker__cell--today
.date-picker__cell--selected
.date-picker__cell--in-range
.date-picker__cell--range-start
.date-picker__cell--range-end
.date-picker__cell--preview
.date-picker__cell--disabled
.date-picker__cell--outside-month
.date-picker__cell--unavailable
```

RAC also adds `data-selected`, `data-disabled`, `data-focused`, `data-hovered`, `data-pressed`, `data-today`, `data-outside-visible-range`, `data-range-start`, `data-range-end`, `data-range-selection-start` on calendar cells — CSS targets these directly for most state styling; BEM modifier classes are only needed for states not covered by RAC data attributes.

### Design decisions and deviations

- **`mode` unifies M3's three presentation variants** into one component instead of three exports (`DesktopDatePicker`, `MobileDatePicker`, `StaticDatePicker` as MUI does). Mode switching within modal (calendar ↔ input toggle) is wired internally.
- **`CalendarDate` throughout** avoids the midnight-UTC-vs-local-timezone class of bugs inherent in JS `Date`. This is RAC's intended usage.
- **`visibleMonths` is a prop**, not a CSS media query swap, because server-side rendering would cause a hydration mismatch if the server and client disagree on viewport width.
- **`isDateUnavailable` follows RAC naming** (not M3's hypothetical `disabledDates`) for API consistency with RAC's full Calendar API.
- **The range picker is always included** — no licensed Pro tier. `DateRangePicker` is a separate export but shares the same calendar/token infrastructure.

### CSS layer encapsulation

```css
@layer kafui {
  .date-picker { … }
  .date-picker__cell { … }
  .date-picker__cell--selected { … }
  /* etc. */
}
```
