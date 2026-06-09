# Time Picker

**M3 category:** Selection. Lets users enter or select a time value. M3 specifies two interaction modes that can coexist in a single component:

1. **Dial (clock) picker** — a circular analog clock face; user taps/drags to select hours then minutes.
2. **Input picker** — a segmented text input for direct keyboard entry (HH:MM + optional AM/PM).

Typically presented inside a **modal dialog** on mobile, or as an inline component on desktop. A mode-toggle icon button switches between dial and input.

**React Aria gap:** React Aria has no clock-dial primitive. Strategy: use RAC `TimeField` + `@internationalized/date` `Time` for **input mode** (full a11y, no custom code). Implement a **custom accessible dial** with manual ARIA management for **dial mode**. Justification is in the API section below.

---

## Anatomy / Parts

### Modal / inline container

```
.time-picker                         Root element (dialog or inline div).
.time-picker__header                 Time display bar.
  .time-picker__display              Large time text, e.g. "09:41".
    .time-picker__hours              Hours segment; tappable; activates hour-selection phase.
    .time-picker__colon              ":" separator.
    .time-picker__minutes            Minutes segment; tappable; activates minute-selection phase.
  .time-picker__period               "AM" / "PM" toggle strip (12h only).

.time-picker__body                   Swaps between dial and input sub-trees.
  .time-picker__dial-mode            Clock face (default on touch devices).
    .time-picker__clock              Circular dial, 256 dp.
      .time-picker__clock-face       Background circle.
      .time-picker__clock-hand       Selection line from center to active value.
      .time-picker__clock-center-dot Pivot dot.
      .time-picker__clock-number     (×12 or ×24) Hour or minute labels.
      .time-picker__selection-indicator  Filled primary circle overlaying the active number.
  .time-picker__input-mode           Keyboard input (default on pointer devices).
    .time-picker__time-field         Wraps RAC TimeField.
      .time-picker__segment--hours
      .time-picker__segment--minutes
      .time-picker__segment--period  AM/PM (12h only).

.time-picker__footer
  .time-picker__mode-toggle          Icon button: clock ↔ keyboard icon.
  .time-picker__action--cancel
  .time-picker__action--confirm
```

| BEM element | Role |
|---|---|
| `.time-picker__display` | Shows current time; `display-large` typography |
| `.time-picker__hours` | Tapping switches dial to hour-selection phase |
| `.time-picker__minutes` | Tapping switches dial to minute-selection phase |
| `.time-picker__period` | AM/PM segmented control (12h only) |
| `.time-picker__clock` | 256 dp circle; touch/pointer drag events |
| `.time-picker__clock-hand` | CSS `transform: rotate()` from center to value |
| `.time-picker__clock-number` | Each hour/minute label; `role="option"` |
| `.time-picker__selection-indicator` | `primary` circle overlaying the active number |
| `.time-picker__mode-toggle` | `<Button>` cycling `dial ↔ input` |

---

## Variants

| Variant | Prop | Notes |
|---|---|---|
| **Dial (default on touch)** | `defaultMode="dial"` | Clock face shown first; `pointer: coarse` auto-selects |
| **Input (default on pointer)** | `defaultMode="input"` | `TimeField` shown first; `pointer: fine` auto-selects |
| **12-hour** | `hourCycle={12}` | AM/PM period selector shown in header and input segments |
| **24-hour** | `hourCycle={24}` | No period selector; hours 0–23; dial has inner (13–00) and outer (1–12) rings |
| **Modal** | `isModal={true}` (default) | Wrapped in RAC `Dialog` + `ModalOverlay` |
| **Inline** | `isModal={false}` | Embedded without overlay |
| **Controlled** | `value` + `onChange` | Caller owns time state |
| **Uncontrolled** | `defaultValue` | Internal state |

---

## States

### Dial mode

| State | Visual |
|---|---|
| **Hour selection phase** | Hours segment in header highlighted (`secondary-container` bg); dial shows hours; hand points to selected hour |
| **Minute selection phase** | Minutes segment highlighted; dial switches to minute values |
| **Hover (number)** | State-layer at `--state-hover` (8%) on hovered number |
| **Drag** | Hand follows pointer continuously via `pointermove`; number highlights as pointer passes |
| **After hour select** | Dial auto-advances to minute phase; hand rotates to current minute (`--duration-short3`) |

### Input mode

| State | Visual |
|---|---|
| **Hours focused** | Hours segment highlighted `primary @ 12%`; accepts digits |
| **Minutes focused** | Minutes segment highlighted |
| **Period focused** | AM/PM segment; toggled by `↑/↓`, `A`, `P` keys |
| **Invalid** | Segment outline switches to `--error`; `isInvalid` set on `TimeField` |

### Modal states

- **Open:** dialog fades in (`--duration-medium1` + `--easing-emphasized-decelerate`); focus trap active; focus moves to hours display or first number.
- **Confirm:** `onChange(currentTime)` fires; dialog closes; focus returns to trigger.
- **Cancel / Escape:** dialog closes; `onChange` NOT called; focus returns to trigger.

---

## Design Tokens

All system tokens referenced by unprefixed CSS custom property names. Component-internal variables inside `.time-picker { … }`.

### Color

| Role | CSS var | Usage |
|---|---|---|
| primary | `--primary` | Clock hand; selection indicator; selected number text; confirm button; focused segment |
| on-primary | `--on-primary` | Text on selection indicator |
| secondary-container | `--secondary-container` | Active display segment highlight (hours or minutes) |
| on-secondary-container | `--on-secondary-container` | Text in active display segment |
| tertiary-container | `--tertiary-container` | AM/PM period button active state |
| on-tertiary-container | `--on-tertiary-container` | AM/PM period button active text |
| surface-container-highest | `--surface-container-highest` | Clock face background; header background |
| surface-container | `--surface-container` | Modal dialog surface |
| on-surface | `--on-surface` | Clock numbers (unselected); display text |
| on-surface-variant | `--on-surface-variant` | Clock numbers in outer ring (24h inner numbers) |
| outline | `--outline` | AM/PM strip border |
| error | `--error` | Invalid input segment outline |

> **Correction from original:** the modal container background should be `--surface-container` (M3 2024: dialogs use `surface-container-high`), not `surface-container-highest`. The clock face background is `surface-container-highest`. Verified against M3 guidelines.

### Shape

| Part | CSS var |
|---|---|
| Modal dialog | `--corner-extra-large` (28 dp) |
| Clock face | `--corner-full` (circle) |
| Selection indicator | `--corner-full` (circle) |
| AM/PM toggle strip | `--corner-small` (8 dp) |
| Display active segment | `--corner-small` (8 dp) |

```css
@layer kafui {
  .time-picker {
    --dial-size: 256px;      /* clock face diameter */
    --number-r-outer: 96px;  /* outer ring radius */
    --number-r-inner: 64px;  /* inner ring radius (24h) */
    --number-target: 40px;   /* touch target per number */
    --hand-w: 2px;           /* clock hand width */
  }
}
```

### Typography

| Role | CSS var | Usage |
|---|---|---|
| display-large | `--display-large-size` etc. | Time display HH:MM |
| body-large | `--body-large-size` etc. | Clock face numbers |
| label-large | `--label-large-size` etc. | AM/PM toggle; action buttons |

### Motion

| Transition | CSS vars |
|---|---|
| Dial hand rotation | `--duration-medium1` + `--easing-standard` |
| Modal open/close | `--duration-medium1` + `--easing-emphasized-decelerate` |
| Phase transition (hour→minute hand) | `--duration-short3` + `--easing-standard` |
| State-layer transitions | `--duration-short2` |

---

## Interaction & Accessibility

### ARIA — Input mode

React Aria `TimeField` handles everything automatically:
- Root → `role="group"` with `aria-label` / `aria-labelledby`.
- Each segment → `role="spinbutton"` with `aria-label="Hours"` / `"Minutes"` / `"AM/PM"`, `aria-valuemin`, `aria-valuemax`, `aria-valuenow`, `aria-valuetext`.

### ARIA — Dial mode (custom)

- Clock container → `role="listbox"` + `aria-label="Select hour"` / `"Select minute"` (switches on phase).
- Each number on the clock face → `role="option"`, `aria-selected="true|false"`.
- Selected/focused number: receives DOM focus via roving tabindex.
- `aria-live="polite"` region (visually hidden, outside the clock) announces the current value during drag on every value change, but **debounced to every ~15° increment** to avoid flooding screen readers.
- Clock container also exposes `aria-activedescendant` pointing to the currently selected option, so AT users get the value without needing to move DOM focus during drag.

### ARIA — Dialog / Modal

- Modal root → `role="dialog"`, `aria-modal="true"`, `aria-labelledby` pointing to the dialog heading or the display element.
- Focus trapped inside modal via RAC `Modal` + `Dialog`.
- On open: focus moves to the hours display segment or, in dial mode, the selected clock number.
- On close: focus returns to the trigger element.

### Keyboard — Dial mode

| Key | Action |
|---|---|
| `Tab` | Move focus between clock numbers (or out of dial to footer) |
| `←/→/↑/↓` | Navigate clock numbers clockwise/counter-clockwise |
| `Home` | Jump to 12 (or 0 in 24h) |
| `End` | Jump to 11 (or 23) |
| `Enter` / `Space` | Select focused number; if hours phase, advance to minutes |
| `Escape` | Cancel / close modal |

### Keyboard — Input mode

| Key | Action |
|---|---|
| `Tab` / `→` | Move to next segment |
| `←` | Move to previous segment |
| `0–9` | Type digits; auto-advance when segment is full |
| `↑` / `↓` | Increment/decrement value in segment |
| `A` / `P` | In period segment: set AM/PM |
| `Backspace` | Clear segment |

### Touch — Dial

- `pointerdown` on clock → capture pointer; begin drag.
- `pointermove` → compute angle from pointer position relative to clock center using `Math.atan2(dy, dx)`; derive value; update hand and selection indicator continuously.
- `pointerup` → confirm selection; if hours phase, auto-advance to minutes phase.
- Numbers remain individually tappable (tap without drag = discrete select).
- Pointer capture ensures the hand tracks the pointer even if it leaves the clock face during drag.

### 12h / 24h hour cycle

- `hourCycle` prop is first-class and overrides locale default.
- `@internationalized/date` `Time` stores hours in 0–23; the display layer formats according to `hourCycle`.
- In 24h dial mode: numbers 1–12 on the outer ring, 13–00 on the inner ring. The hand auto-selects the correct ring based on whether the current hour is ≥ 13 or 0.
- In 12h dial mode: single outer ring with numbers 1–12; AM/PM controlled separately by the period selector.

### minValue / maxValue

- Passed to `TimeField` in input mode (RAC handles segment clamping).
- In dial mode: clock numbers outside the allowed range have `aria-disabled="true"` and `pointer-events: none`. The hand cannot be dragged to a disabled position.

### Reduced motion

```css
@layer kafui {
  @media (prefers-reduced-motion: reduce) {
    .time-picker__clock-hand,
    .time-picker__selection-indicator {
      transition: none;
    }
    .time-picker {
      animation: none;
    }
  }
}
```

Input mode is unaffected (no motion). Under `reduce`, the hand and selection indicator snap to new positions instantly.

### RTL

- Modal layout uses logical properties; action buttons maintain M3-prescribed order.
- Clock numbers remain in clockwise order regardless of `dir` — the clock face is not mirrored.
- Input segment order: `@internationalized/date` `TimeField` handles RTL segment reordering automatically via locale.
- AM/PM strip uses `flex-direction: column` — not direction-dependent.

---

## Proposed kafUI React API

```tsx
import type { Time } from "@internationalized/date";

interface TimePickerProps {
  // Value
  value?: Time | null;              // controlled
  defaultValue?: Time | null;       // uncontrolled
  onChange?: (value: Time) => void;

  // Display
  hourCycle?: 12 | 24;             // default: locale-dependent via I18nProvider
  /**
   * Initial mode. If omitted, auto-selects based on pointer media query:
   * 'dial' for pointer:coarse (touch), 'input' for pointer:fine (mouse).
   */
  defaultMode?: "dial" | "input";
  /** Allow user to toggle between modes via the footer icon button. Default: true */
  allowModeToggle?: boolean;

  // Modal behavior
  isModal?: boolean;               // default: true
  isOpen?: boolean;                // controlled open state (modal only)
  defaultOpen?: boolean;
  onOpenChange?: (isOpen: boolean) => void;
  /** Trigger element for modal variant (e.g. a TextField with clock icon). */
  trigger?: React.ReactNode;

  // State
  isDisabled?: boolean;
  isReadOnly?: boolean;
  isInvalid?: boolean;
  errorMessage?: React.ReactNode;
  minValue?: Time;
  maxValue?: Time;

  // Labels
  label?: React.ReactNode;         // dialog title / aria-label
  cancelLabel?: string;            // default: "Cancel"
  confirmLabel?: string;           // default: "OK"

  // ARIA
  "aria-label"?: string;
  "aria-labelledby"?: string;
  "aria-describedby"?: string;

  className?: string;
}
```

### BEM classes emitted

```
.time-picker
.time-picker--modal
.time-picker--inline
.time-picker--dial-mode
.time-picker--input-mode
.time-picker--12h
.time-picker--24h
.time-picker--disabled
.time-picker--phase-hours   ← dial is in hour-selection phase
.time-picker--phase-minutes ← dial is in minute-selection phase

.time-picker__header
.time-picker__display
  .time-picker__hours[data-active]
  .time-picker__colon
  .time-picker__minutes[data-active]
.time-picker__period        ← 12h only

.time-picker__body
.time-picker__dial-mode
  .time-picker__clock[role="listbox"][aria-label="Select hour|Select minute"]
    .time-picker__clock-face
    .time-picker__clock-hand
    .time-picker__clock-center-dot
    .time-picker__clock-number[role="option"][aria-selected][aria-disabled?]
    .time-picker__selection-indicator
.time-picker__input-mode
  .time-picker__time-field    ← RAC TimeField root

.time-picker__footer
.time-picker__mode-toggle
.time-picker__action--cancel
.time-picker__action--confirm
```

RAC `TimeField` also puts `data-focused`, `data-hovered`, `data-invalid` on the group and `data-placeholder` on each segment — CSS targets these directly.

### React Aria primitives used

| Primitive | Purpose |
|---|---|
| `TimeField` + `DateInput` + `DateSegment` | Input mode: full a11y, locale-aware, `Time` value |
| `ModalOverlay` + `Dialog` | Modal: focus trap, `aria-modal`, Escape key, return focus |
| `Button` | Cancel / Confirm / mode-toggle |
| `I18nProvider` | Locale + `hourCycle` propagation |

### Why no React Aria dial primitive

React Aria has no `ClockDial` or radial-selector primitive. The custom dial is justified because:
1. No accessible radial/circular selector exists in any major a11y component library.
2. `role="listbox"` + `role="option"` is the closest ARIA pattern for "visual selection from a circular list".
3. `TimeField` (input mode) covers the accessibility baseline; the dial is a progressive enhancement over the same `Time` value.
4. Building on RAC `ListBox` is infeasible because the layout is circular (CSS/SVG-positioned), not a DOM list — ARIA attributes must be applied manually.

Strategy: dial mode → custom `useDial` hook + manual ARIA; input mode → RAC `TimeField` (zero ARIA work). Both modes share the same `Time` value state.

### CSS layer encapsulation

```css
@layer kafui {
  .time-picker { … }
  .time-picker__clock-hand { … }
  /* etc. */
}
```
