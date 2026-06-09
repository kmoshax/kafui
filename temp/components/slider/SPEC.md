# Slider

**M3 category:** Selection — lets users select a value (or range) from a continuous or discrete set by dragging a handle along a track.

M3 2024 refresh introduced:
- **Stop indicators** on discrete tracks (4dp circles at each step position; dual-color: `--on-primary` inside active segment, `--on-surface-variant` outside).
- A **gap** between the handle and the active/inactive track segments (~4dp each side).
- A **taller/thicker** track than pre-2024 (4dp height, but touch target remains 40dp tall via the state-layer).
- The handle **grows on press/drag** from 20dp → 26dp (`--duration-short2` spring).

---

## Anatomy / Parts

### Single-value slider

```
.slider
├── .slider__label                — optional; above track
├── .slider__track-wrapper        — full-width row; 40dp tall touch target
│   ├── .slider__track-inactive   — full-width background bar
│   ├── .slider__track-active     — filled portion from start to handle (or between handles for range)
│   ├── .slider__stop-indicators  — (discrete only) row of dots at step positions
│   │   └── .slider__stop         — single stop dot (×N)
│   │       └── .slider__stop--active  — stop within active segment
│   └── .slider__thumb-container  — absolutely positioned at value %
│       ├── .slider__state-layer  — 40dp circle overlay
│       ├── .slider__handle       — 20dp (resting) → 26dp (pressed) circle
│       └── .slider__value-label  — tooltip balloon above handle
└── .slider__output               — optional <output> element with live value
```

### Range slider (two thumbs)

```
.slider.slider--range
  (same track-wrapper)
  ├── .slider__track-active          — segment between the two handles
  ├── .slider__thumb-container.slider__thumb-container--start
  └── .slider__thumb-container.slider__thumb-container--end
```

| BEM element | Notes |
|---|---|
| `.slider__track-inactive` | `--secondary-container`; 4dp tall; `border-radius: var(--corner-full)` |
| `.slider__track-active` | `--primary`; 4dp tall; width/position from CSS vars |
| `.slider__stop` | 4dp × 4dp circle; `border-radius: 50%` |
| `.slider__stop--active` | `--on-primary` fill (inside active segment) |
| `.slider__stop` (inactive) | `--on-surface-variant` fill |
| `.slider__handle` | `--primary` fill; `border-radius: 50%`; size morphs 20dp → 26dp |
| `.slider__state-layer` | 40dp; absolute, centered on handle |
| `.slider__value-label` | `--primary` bg, `--on-primary` text; `--label-small-*` type; hidden at rest |

**Gap:** Active track ends 4dp short of the handle's edge; inactive track begins 4dp from the other edge. Implemented via CSS calc on the `--slider-active-end` / `--slider-active-start` CSS vars.

---

## Variants

| Variant | Prop | Notes |
|---|---|---|
| **Continuous** | `step` omitted (default) | No stops; smooth value |
| **Discrete** | `step={number}` | Renders stop indicators; value snaps to steps |
| **Single value** | default | One thumb |
| **Range** | `value` / `defaultValue` = `[number, number]` | Two thumbs; no separate `isRange` boolean — inferred from value type |
| **Centered origin** | `origin="center"` | Active track extends from midpoint outward (±value sliders) |
| **With value label** | `showValueLabel="auto|always|never"` | `auto`: show on drag/focus (default); `always`: always visible; `never`: hidden |
| **With output** | `showOutput` | Renders `<output>` element with live formatted value |

**Design decision:** `isRange` boolean is removed from the API. Range mode is inferred from whether `value` or `defaultValue` is a tuple `[number, number]`. This avoids the footgun where `isRange={true}` conflicts with `value={42}`. A single discriminated union is cleaner and type-safe.

---

## States

| State | Visual |
|---|---|
| **Resting** | Handle 20dp; no state-layer; value label hidden (auto) |
| **Hover** | State-layer at `--state-hover` (8%) tinted `--primary` |
| **Focus visible** | State-layer at `--state-focus` (10%); value label shown (auto) |
| **Pressed / dragging** | Handle 26dp (`--duration-short2` spring); state-layer at `--state-pressed` (10%); value label shown |
| **Disabled** | Handle + active track: `--on-surface` at 38%; inactive track: `--on-surface` at 12%; stops: disabled color; no interaction |
| **Overlap (range)** | When thumbs coincide, end thumb sits above start thumb (z-index); both remain keyboard-operable |

---

## Design Tokens

All references are unprefixed system roles per `_TOKENS.md`.

| Token | Usage |
|---|---|
| `--primary` | Active track, handle, value-label background, state-layer tint |
| `--on-primary` | Value-label text; stops inside active segment |
| `--secondary-container` | Inactive track |
| `--on-surface-variant` | Stops outside active segment |
| `--on-surface` | Disabled tint |
| `--corner-full` | Handle and stop `border-radius: 50%` |
| `--label-small-font` | Value-label text |
| `--duration-short2` | Handle size morph (~100 ms) |
| `--easing-emphasized` | Handle morph easing (M3 spring; maps to emphasized decelerate on expand, accelerate on shrink) |
| `--state-hover` | 0.08 |
| `--state-focus` | 0.10 |
| `--state-pressed` | 0.10 |

**Token correction:** The original spec referenced `--md-sys-motion-easing-spring` which is not in the M3 motion token set. The correct token is `--easing-emphasized` (the M3 "emphasized" spring easing used for expressive interactions). Fallback to `cubic-bezier(0.05, 0.7, 0.1, 1.0)`.

---

## Interaction & Accessibility

### ARIA roles
- Slider root → `role="group"` with `aria-label` or `aria-labelledby` (required when labeled).
- Each thumb hidden input → `role="slider"` (RAC `SliderThumb` renders a hidden `<input type="range">`).
- `aria-valuenow`: current numeric value.
- `aria-valuemin` / `aria-valuemax`: track bounds.
- `aria-valuetext`: formatted string (e.g. `"$42"`, `"42%"`, `"42°C"`); supplied via `getValueLabel` prop.
- `aria-label` on each thumb — for range sliders, label each thumb distinctly (default: `"Minimum"` / `"Maximum"`; override via `startThumbLabel` / `endThumbLabel`).
- `aria-disabled` when disabled.
- `aria-orientation="horizontal"` (default); `"vertical"` if a vertical variant is added in future.

### Keyboard
| Key | Action |
|---|---|
| `Tab` | Focus thumb (single) or first/next thumb (range) |
| `Arrow Right` / `Arrow Up` | Increment by `step` (default 1% of range for continuous) |
| `Arrow Left` / `Arrow Down` | Decrement |
| `Home` | Jump to `min` |
| `End` | Jump to `max` |
| `Page Up` | Increment by 10 × `step` (or 10% of range) |
| `Page Down` | Decrement by 10 × `step` |

### Touch / pointer
- Pointer capture on `pointerdown`; value updates on `pointermove`; released on `pointerup`. RAC handles this natively.
- 40dp state-layer on each thumb provides ≥40dp hit target; the track itself is an additional tap target.

### RTL
- React Aria's `Slider` reads direction from `I18nProvider`; min is at inline-end in RTL. Track active fill and thumb position flip automatically.
- Stop indicators are symmetric; no asset flip needed.
- `origin="center"` still works — active track expands from center.

### Reduced motion
```css
@layer kafui {
  @media (prefers-reduced-motion: reduce) {
    .slider__handle { transition: none; }
    .slider__value-label { transition: none; opacity: 1; } /* always visible if shown */
  }
}
```

---

## CSS Architecture

Active track and thumb positions are driven by CSS custom properties set by the React component from RAC's `state.getThumbPercent()`. This keeps CSS pure and the React component as the single source of truth for value math.

```css
@layer kafui {
  .slider {
    /* ── component-internal vars ── */
    --handle-rest:  20px;
    --handle-press: 26px;
    --sl:           40px;
    --track-h:      4px;
    --gap:          4px;   /* gap between active track and handle */
    --dur:          var(--duration-short2);
    --ease:         var(--easing-emphasized);

    /* ── set by React from RAC state.getThumbPercent() ── */
    --slider-value-pct:    0%;    /* single thumb */
    --slider-start-pct:    0%;    /* range: start thumb */
    --slider-end-pct:    100%;    /* range: end thumb */

    display: grid;
    gap: 8px;
    /* rows: [label] [track-wrapper] [output] */
  }

  /* ── track wrapper ── */
  .slider__track-wrapper {
    position: relative;
    height: var(--sl);   /* 40dp touch target */
    display: flex;
    align-items: center;
    cursor: pointer;
  }

  .slider__track-inactive {
    position: absolute;
    inset-inline: 0;
    height: var(--track-h);
    border-radius: var(--corner-full);
    background: var(--secondary-container);
  }

  .slider__track-active {
    position: absolute;
    height: var(--track-h);
    border-radius: var(--corner-full);
    background: var(--primary);
    /* single slider: start=0, end=value - gap */
    inset-inline-start: 0;
    width: calc(var(--slider-value-pct) - var(--gap));
  }

  /* range: active segment between two thumbs */
  .slider--range .slider__track-active {
    inset-inline-start: calc(var(--slider-start-pct) + var(--gap));
    width: calc(var(--slider-end-pct) - var(--slider-start-pct) - 2 * var(--gap));
  }

  /* ── stops (discrete) ── */
  .slider__stop-indicators {
    position: absolute;
    inset-inline: 0;
    height: var(--track-h);
    display: flex;
    align-items: center;
    pointer-events: none;
  }

  .slider__stop {
    position: absolute;
    width: 4px;
    height: 4px;
    border-radius: 50%;
    background: var(--on-surface-variant);
    /* inset-inline-start set per-stop by React as inline style: calc(var(--stop-pct) * 100%) */
    transform: translateX(-50%);
  }

  .slider__stop--active {
    background: var(--on-primary);
  }

  /* ── thumb container ── */
  .slider__thumb-container {
    position: absolute;
    width: var(--sl);
    height: var(--sl);
    top: 50%;
    transform: translateY(-50%);
    inset-inline-start: calc(var(--slider-value-pct) - var(--sl) / 2);
    display: flex;
    align-items: center;
    justify-content: center;
    border-radius: 50%;
    touch-action: none;
  }

  .slider__thumb-container--start {
    inset-inline-start: calc(var(--slider-start-pct) - var(--sl) / 2);
    z-index: 1;
  }
  .slider__thumb-container--end {
    inset-inline-start: calc(var(--slider-end-pct) - var(--sl) / 2);
    z-index: 2;
  }

  /* ── state-layer ── */
  .slider__state-layer {
    position: absolute;
    inset: 0;
    border-radius: 50%;
    background: var(--primary);
    opacity: 0;
    pointer-events: none;
    transition: opacity 150ms var(--ease);
  }

  /* ── handle ── */
  .slider__handle {
    width: var(--handle-rest);
    height: var(--handle-rest);
    border-radius: 50%;
    background: var(--primary);
    transition:
      width var(--dur) var(--ease),
      height var(--dur) var(--ease);
    flex-shrink: 0;
  }

  .slider__thumb-container[data-dragging] .slider__handle,
  .slider__thumb-container[data-pressed]  .slider__handle {
    width: var(--handle-press);
    height: var(--handle-press);
  }

  /* ── interaction states ── */
  .slider__thumb-container[data-hovered]       .slider__state-layer { opacity: var(--state-hover); }
  .slider__thumb-container[data-focus-visible] .slider__state-layer { opacity: var(--state-focus); }
  .slider__thumb-container[data-pressed],
  .slider__thumb-container[data-dragging]      { /* state-layer */ }
  .slider__thumb-container[data-dragging]      .slider__state-layer { opacity: var(--state-pressed); }

  /* ── value label ── */
  .slider__value-label {
    position: absolute;
    bottom: calc(100% + 4px);
    left: 50%;
    transform: translateX(-50%);
    background: var(--primary);
    color: var(--on-primary);
    font: var(--label-small-font);
    padding: 2px 8px;
    border-radius: var(--corner-full);
    white-space: nowrap;
    opacity: 0;
    pointer-events: none;
    transition: opacity 150ms var(--ease);
  }

  /* showValueLabel="always" */
  .slider--value-label-always .slider__value-label { opacity: 1; }

  /* auto: show on focus or drag */
  .slider__thumb-container[data-focus-visible] .slider__value-label,
  .slider__thumb-container[data-dragging]      .slider__value-label { opacity: 1; }

  /* ── disabled ── */
  .slider--disabled {
    pointer-events: none;
    cursor: not-allowed;
  }
  .slider--disabled .slider__track-active,
  .slider--disabled .slider__handle {
    background: color-mix(in srgb, var(--on-surface) 38%, transparent);
  }
  .slider--disabled .slider__track-inactive {
    background: color-mix(in srgb, var(--on-surface) 12%, transparent);
  }

  /* ── label / output ── */
  .slider__label {
    font: var(--body-large-font);
    color: var(--on-surface);
  }

  .slider__output {
    font: var(--body-medium-font);
    color: var(--on-surface-variant);
    text-align: end;
  }

  /* ── reduced motion ── */
  @media (prefers-reduced-motion: reduce) {
    .slider__handle     { transition: none; }
    .slider__value-label { transition: none; }
  }
}
```

---

## Proposed kafUI React API

```tsx
interface SliderProps {
  // Value — range mode inferred from tuple type; no isRange prop
  min?: number;                           // default 0
  max?: number;                           // default 100
  step?: number;                          // omit = continuous
  value?: number | [number, number];      // controlled; tuple = range mode
  defaultValue?: number | [number, number];
  onChange?: (value: number | [number, number]) => void;
  onChangeEnd?: (value: number | [number, number]) => void;

  // Presentation
  /**
   * "start"  — active track from inline-start to thumb (default)
   * "center" — active track expands from midpoint outward
   */
  origin?: "start" | "center";
  /**
   * "auto"   — show value label on focus or drag (default)
   * "always" — always visible
   * "never"  — never shown
   */
  showValueLabel?: "auto" | "always" | "never";
  /** Render an <output> element showing the live formatted value */
  showOutput?: boolean;
  /** Format value for display (aria-valuetext + value label). Default: String(value). */
  getValueLabel?: (value: number) => string;

  // State
  isDisabled?: boolean;

  // Labels
  label?: React.ReactNode;
  /** Label at the track start (min end) — visual annotation */
  startLabel?: React.ReactNode;
  /** Label at the track end (max end) — visual annotation */
  endLabel?: React.ReactNode;
  /** aria-label for start thumb in range mode. Default: "Minimum" */
  startThumbLabel?: string;
  /** aria-label for end thumb in range mode. Default: "Maximum" */
  endThumbLabel?: string;

  // ARIA
  "aria-label"?: string;
  "aria-labelledby"?: string;
}
```

**BEM classes emitted:**

```
.slider
  .slider--discrete                   ← step prop is set
  .slider--range                      ← value is tuple
  .slider--disabled
  .slider--value-label-always         ← showValueLabel="always"
  .slider--value-label-never          ← showValueLabel="never"
  .slider__label
  .slider__track-wrapper
    .slider__track-inactive
    .slider__track-active
    .slider__stop-indicators          ← discrete only
      .slider__stop
      .slider__stop--active
    .slider__thumb-container          ← single, or --start / --end for range
      .slider__thumb-container--start
      .slider__thumb-container--end
      .slider__state-layer
      .slider__handle
      .slider__value-label
  .slider__output
```

**React Aria primitives used:**
- `Slider` — group container; value/state management; RTL direction via `I18nProvider`.
- `SliderTrack` — track element; pointer event handling; `data-orientation`.
- `SliderThumb` — per-thumb hidden `<input type="range">`; keyboard handling; `isPressed`, `isDragging`, `isFocusVisible`, `isHovered` render props.

**Usage examples:**

```tsx
// Continuous single:
<Slider label="Volume" defaultValue={50} getValueLabel={(v) => `${v}%`} />

// Discrete:
<Slider label="Step" min={0} max={100} step={10} showValueLabel="always" />

// Range:
<Slider
  label="Price range"
  min={0} max={500}
  defaultValue={[100, 300]}
  getValueLabel={(v) => `$${v}`}
/>

// Centered origin (±10):
<Slider label="Balance" min={-10} max={10} origin="center" />
```

**Design decisions / deviations:**

- `isRange` boolean removed — range mode is inferred from value type (`number` vs `[number, number]`). This is type-safe: TypeScript narrows the `onChange` callback type based on `value`'s type. No mismatch footgun.
- Stop indicators are rendered as separate `<span>` elements in kafUI React; React Aria has no stop concept. Positions computed as `(stepValue - min) / (max - min) * 100` and set as `--stop-pct` inline custom properties on each stop element.
- Active track bounds (`--slider-value-pct`, `--slider-start-pct`, `--slider-end-pct`) are computed from RAC's `state.getThumbPercent(index)` and set as inline custom properties on the track-wrapper. This keeps CSS rule count minimal — all positioning is a CSS calc.
- `origin="center"` computes `--slider-active-start` as the midpoint percentage and `--slider-active-end` as the value percentage (or vice versa for negative values). The React component handles this math; CSS sees only two custom properties.
- The `easing-spring` token reference in the original spec is replaced with `--easing-emphasized` which is the correct M3 token name and maps to the expressive spring cubic-bezier.
- `getValueLabel` serves double duty: `aria-valuetext` on the hidden input AND the displayed value label text. If omitted, defaults to `String(value)`. This avoids two separate formatting props.
- Range thumbs in an overlap scenario: the end thumb has `z-index: 2` and the start thumb `z-index: 1`. When they overlap exactly, keyboard focus can still reach either thumb via `Tab`; no JS z-index swapping required.
