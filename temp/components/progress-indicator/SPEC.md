# Progress Indicator

**Purpose:** Communicates ongoing process duration and status. Used for loading, file transfers, multi-step forms, and any operation where progress is measurable (determinate) or unknown (indeterminate).
**M3 category:** Communication → Progress Indicator.

---

## Anatomy / Parts → BEM Elements

### Linear

```
.progress                          root <div> — wraps RAC ProgressBar
.progress--linear                  modifier
.progress--determinate             modifier (indeterminate when absent)
.progress__track                   full-width background bar
.progress__indicator               active fill (determinate: width%; indeterminate: animated)
.progress__stop                    stop indicator dot — determinate only (M3 2024+)
```

The **stop indicator** (M3 2024): a 4 dp filled circle positioned at the leading edge of the unfilled section, signaling where progress ends. Rendered as an absolutely positioned pseudo-element or dedicated `<span>` at `inset-inline-start: calc(var(--_value) * 1%)`. Hidden at `value=100`.

The **4 dp gap**: the `.progress__indicator` width is `calc(var(--_value) * 1% - 4px)` (clamped to 0 minimum); the stop indicator sits at `calc(var(--_value) * 1% - 2px)` (centered on the gap edge). The gap makes the endpoint visually distinct from a full track.

### Circular

```
.progress--circular                modifier on root
.progress__svg                     <svg> container, sized to match component diameter
.progress__track-circle            <circle> background stroke
.progress__indicator-circle        <circle> active stroke (stroke-dashoffset driven)
```

> **No `.progress__gap` element needed.** The circular gap is achieved purely via `stroke-dasharray`: leave a fixed-length gap (4 dp equivalent in SVG units) between the start and end of the arc. This applies in *both* determinate and indeterminate states per M3 2024+. No extra DOM node required.

---

## Variants

| Variant | `variant` prop | `indeterminate` | Description |
|---------|---------------|-----------------|-------------|
| Linear determinate | `"linear"` | `false` (default) | `value` 0–100 drives fill width + stop indicator |
| Linear indeterminate | `"linear"` | `true` | Two-segment looping animation; no `value` |
| Circular determinate | `"circular"` | `false` (default) | SVG stroke-dashoffset maps to `value` |
| Circular indeterminate | `"circular"` | `true` | Rotating arc with variable-width stroke breathing |

`variant` is **required** — no default. `indeterminate` defaults to `false`.

---

## M3 2024+ Design Updates

### Linear — Gap + Stop Indicator
- A **4 dp gap** appears between the active indicator's leading edge and the stop indicator.
- A **4 dp stop dot** sits at the boundary of the unfilled track.
- Both are hidden when `indeterminate={true}` or `value === 100`.
- Gap + stop = no ambiguity about "is the bar full or just near 100%".

### Circular — Variable Stroke (Wavy)
- Indeterminate circular uses **variable-width stroke**: stroke-width animates between ~3 dp and ~6 dp in a breathing pattern layered on top of the arc rotation.
- Both determinate and indeterminate carry a **4 dp arc gap** (dasharray gap) between the arc head and tail — this is the visual "open end" of the circle that distinguishes it from a pie/filled shape.
- Track stroke-width: fixed at 4 dp. Active indicator stroke-width: 4 dp (determinate); 3–6 dp breathing (indeterminate).

> **Wavy / active-indicator track gap — spec clarification:** M3 uses the term "active indicator" for the filled portion. The "track gap" is the 4 dp empty arc between the indicator's leading edge (head) and its tail (where it started). This gap is part of the indeterminate animation behavior — it ensures the arc never closes into a complete circle, maintaining visual motion even at slow rotation speeds.

---

## States

| State | Behavior |
|-------|----------|
| Determinate | `value` 0–100 drives indicator; stop indicator visible |
| Indeterminate | CSS keyframe animation; `aria-valuenow` absent |
| `value=0` | Empty track; stop indicator at position 0 (visible if `value > 0` only — hide at exactly 0) |
| `value=100` | Full track; stop indicator hidden; gap removed |
| Hidden/unmounted | Not an M3 state; consumer unmounts or uses `hidden` attribute |

---

## Design Tokens

| Token | CSS custom property (unprefixed) | Usage |
|-------|----------------------------------|-------|
| `md.sys.color.primary` | `--primary` | active indicator color |
| `md.sys.color.secondary-container` | `--secondary-container` | track color |
| `md.sys.shape.corner.full` | `--corner-full` | track + indicator border-radius (linear) |
| `md.sys.motion.duration.long2` | `--duration-long2` | indeterminate cycle (~1200 ms) |
| `md.sys.motion.easing.emphasized` | `--easing-emphasized` | indeterminate segment motion |

**Component-internal variables** (scoped inside `.progress { … }`):
```css
@layer kafui.components {
.progress {
  --_value:        0;          /* set via inline style by React: style={{ "--_value": value }} */
  --_track-h:      4px;
  --_circ-size:    48px;       /* overridden by size prop */
  --_stroke-w:     4px;
  --_stroke-w-min: 3px;        /* indeterminate breathing min */
  --_stroke-w-max: 6px;        /* indeterminate breathing max */
  --_gap:          4px;
  --_stop-r:       2px;        /* stop indicator radius = half of 4dp */
}
}
```

> **No inline color styles.** Value is the only inline-style CSS var needed. All color/shape tokens are resolved from `--primary`, `--secondary-container`, etc. via the global token sheet.

---

## Interaction & Accessibility

### ARIA (via React Aria `ProgressBar`)
```html
<div
  role="progressbar"
  aria-label="Uploading file"
  aria-valuenow="42"
  aria-valuemin="0"
  aria-valuemax="100"
>
```
- Omit `aria-valuenow` (and min/max) for indeterminate — React Aria `ProgressBar` does this automatically when `value` is not passed.
- `aria-label` or `aria-labelledby` is **required**. kafUI warns in dev if both are absent.
- Do not replicate the value in `aria-label` ("Loading 42%") — `aria-valuenow` already communicates it; the label should describe *what* is loading.

### Screen reader — milestone announcements
AT reads `role="progressbar"` + value on initial render and on `aria-valuenow` change. Do **not** announce every 1% change. For long operations, expose a separate `aria-live="polite"` region that fires at milestones (25%, 50%, 75%, 100%) — this is out-of-scope for the component but documented as the recommended hosting pattern.

### Keyboard
Non-interactive — no keyboard target, no focus ring.

### Reduced motion
```css
@media (prefers-reduced-motion: reduce) {
  .progress--linear .progress__indicator { animation: none; width: 50%; }
  .progress--circular .progress__indicator-circle { animation: none; }
  /* indeterminate fallback: static 50% fill / 180° arc */
}
```

### RTL
- Linear indicator fills from `inline-start`. `.progress__track` and `.progress__indicator` use logical widths; the stop indicator uses `inset-inline-start`.
- Circular SVG rotation is direction-neutral (clockwise is universal). No RTL override needed for circular.

---

## Proposed kafUI React API

```tsx
interface ProgressIndicatorProps {
  /** Shape variant — required, no default */
  variant: "linear" | "circular";
  /** true = indeterminate animation. Default: false */
  indeterminate?: boolean;
  /**
   * Progress value 0–100.
   * Required when indeterminate={false}; ignored when indeterminate={true}.
   */
  value?: number;
  /**
   * Diameter in px for circular variant. Default: 48.
   * Named size aliases: "sm"=24, "md"=48 (default), "lg"=64.
   */
  size?: number | "sm" | "md" | "lg";
  /** Accessible name — required; dev warning if absent */
  "aria-label"?: string;
  "aria-labelledby"?: string;
  className?: string;
}

// Determinate linear
<ProgressIndicator variant="linear" value={60} aria-label="Uploading file" />

// Indeterminate circular
<ProgressIndicator variant="circular" indeterminate aria-label="Loading" />

// Circular small
<ProgressIndicator variant="circular" size="sm" indeterminate aria-label="Loading" />

// Determinate circular
<ProgressIndicator variant="circular" value={75} aria-label="Analyzing" />
```

**React Aria primitive:** `ProgressBar` from `react-aria-components`. Pass `value` and `maxValue=100` for determinate; omit `value` for indeterminate. RAC automatically manages `aria-valuenow` presence.

**BEM classes emitted:**
```
.progress  .progress--linear | .progress--circular
           .progress--determinate | .progress--indeterminate
.progress__track  /  .progress__track-circle
.progress__indicator  /  .progress__indicator-circle
.progress__stop   (linear determinate only)
```

**Unified component (vs MUI's split):**
MUI ships `<LinearProgress>` and `<CircularProgress>` as separate components with diverged APIs (`buffer` variant, `thickness` prop, etc.). kafUI uses one `<ProgressIndicator variant>` — shared ARIA logic, shared token system, half the bundle.

**`buffer` variant decision:**
M3 does not define a buffer variant. kafUI does not ship it in v1. If needed, the track can be split into two layers with a separate `bufferValue` prop — document as a "M3 extension" and gate behind a feature flag comment.

**`size` prop — named aliases:**
- `"sm"` = 24 px (inline / tight spaces)
- `"md"` = 48 px (default)
- `"lg"` = 64 px (prominent loading screens)
- Raw `number` for custom sizes. Sizes apply only to `variant="circular"`; linear always tracks container width.
