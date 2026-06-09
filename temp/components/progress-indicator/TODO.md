# Progress Indicator — TODO

## MUI Equivalents
`@mui/material/LinearProgress` + `@mui/material/CircularProgress` — two separate components. kafUI merges them under one API, adds M3 2024 geometry (stop indicator, arc gap, variable-width stroke), and eliminates emotion overhead.

---

## Beat-MUI Opportunities

| # | kafUI wins by… | vs MUI |
|---|----------------|--------|
| 1 | **One component, one import** — `<ProgressIndicator variant="linear|circular">` vs importing two separate MUI components with diverged prop APIs | MUI: `LinearProgress` + `CircularProgress` are unrelated components; different `variant` strings, different props |
| 2 | **M3 2024 stop indicator + arc gap** ships out of the box — visually correct M3 geometry that MUI has never implemented | MUI circular is constant-width rotating arc (M2-era); no stop indicator, no arc gap |
| 3 | **Variable-width stroke (wavy)** on circular indeterminate — expressive, immediately distinguishable from a plain spinner | MUI indeterminate circular = constant stroke-width; generic "spinner" look |
| 4 | **Named size aliases** (`"sm"` / `"md"` / `"lg"`) + raw `number` — predictable presets without magic numbers | MUI `size` is raw px only; no semantic aliases |
| 5 | **`--_value` CSS var for determinate** — pure CSS, no JS DOM manipulation | MUI injects `width` or `strokeDashoffset` as inline style on every re-render |

---

## Actionable TODO Checklist

### Core implementation
- [ ] `packages/react/src/progress-indicator/ProgressIndicator.tsx` — wrap RAC `ProgressBar`
- [ ] Forward `value` (0–100), `minValue=0`, `maxValue=100` to RAC; for indeterminate omit `value`
- [ ] Inject `--_value` as inline CSS var: `style={{ "--_value": value ?? 0 } as React.CSSProperties}`
- [ ] Resolve `size` alias: `"sm"` → 24, `"md"` → 48, `"lg"` → 64
- [ ] Inject `--_circ-size` on circular root via inline style: `style={{ "--_circ-size": resolvedSize + "px" }}`
- [ ] Dev warning: if both `aria-label` and `aria-labelledby` absent, `console.warn` in development
- [ ] `packages/styles/src/progress-indicator/progress.css` — single file, `@layer kafui { … }`

### Linear styles
```css
@layer kafui {
  .progress--linear {
    --_value: 0;
    --_track-h: 4px;
    --_gap: 4px;
    --_stop-r: 2px;
    position: relative;
    width: 100%;
    height: var(--_track-h);
    overflow: visible; /* stop indicator can protrude */
  }

  .progress__track {
    position: absolute;
    inset: 0;
    background: var(--secondary-container);
    border-radius: var(--corner-full);
    overflow: hidden; /* clip indicator within track */
  }

  .progress__indicator {
    height: 100%;
    background: var(--primary);
    border-radius: var(--corner-full);
    /* determinate */
    width: clamp(0px, calc(var(--_value) * 1% - var(--_gap)), calc(100% - var(--_gap)));
  }

  .progress--indeterminate .progress__indicator {
    width: auto; /* animation overrides */
    animation: linear-indeterminate var(--duration-long2) var(--easing-emphasized) infinite;
  }

  /* Stop indicator — determinate only, hidden at value=0 and value=100 */
  .progress__stop {
    position: absolute;
    inset-block: calc(50% - var(--_stop-r));
    inset-inline-start: clamp(0px, calc(var(--_value) * 1% - var(--_stop-r)), calc(100% - var(--_stop-r) * 2));
    width: calc(var(--_stop-r) * 2);
    height: calc(var(--_stop-r) * 2);
    border-radius: var(--corner-full);
    background: var(--primary);
    display: block;
  }

  /* Hide stop at boundaries */
  .progress--linear[style*="--_value: 0"] .progress__stop,
  .progress--linear[style*="--_value: 100"] .progress__stop {
    display: none;
  }
  /* Better: use JS to add .progress--at-zero / .progress--complete modifiers */
  .progress--at-zero .progress__stop,
  .progress--complete .progress__stop { display: none; }

  /* Indeterminate hides stop */
  .progress--indeterminate .progress__stop { display: none; }

  @keyframes linear-indeterminate {
    /* Two-segment M3 timings — exact keyframes TBD from M3 motion spec */
    0%   { margin-inline-start: -35%; width: 35%; }
    60%  { margin-inline-start: 100%; width: 100%; }
    100% { margin-inline-start: 100%; width: 35%; }
  }

  @media (prefers-reduced-motion: reduce) {
    .progress--indeterminate .progress__indicator {
      animation: none;
      width: 50%;
    }
  }
}
```
- [ ] Implement above; replace inline `style*=` attribute selector approach with React-driven modifier classes `.progress--at-zero` / `.progress--complete`
- [ ] Add second animation segment for the two-segment M3 indeterminate behavior (primary + secondary bar)

### Circular styles
- [ ] SVG: `viewBox="0 0 {size} {size}"` driven by `--_circ-size`; center = `size/2`
- [ ] Track circle: `stroke: var(--secondary-container)`, `stroke-width: var(--_stroke-w)`, `fill: none`
- [ ] Indicator circle: `stroke: var(--primary)`, `stroke-width: var(--_stroke-w)`
- [ ] Determinate dasharray: `circumference = 2π * r`; dashoffset = `circumference * (1 - value/100)` — set via CSS var `--_dashoffset`; inject inline
- [ ] 4 dp arc gap: build into dasharray as fixed gap segment (`stroke-dasharray: {filled} {gap} {rest}`)
- [ ] Indeterminate animation: `@keyframes` for rotation + separate `@keyframes` for stroke-width 3→6→3 breathing
- [ ] Layer both animations: `animation: circ-rotate Xms linear infinite, circ-breathe Yms ease-in-out infinite`
- [ ] `prefers-reduced-motion`: static 270° arc, no rotation, fixed stroke-width

### Accessibility
- [ ] Confirm RAC `ProgressBar` omits `aria-valuenow`/`aria-valuemin`/`aria-valuemax` when `value` not passed
- [ ] Forward `aria-label` and `aria-labelledby` to RAC `ProgressBar` (it renders the ARIA attrs on the root div)
- [ ] Emit `.progress--at-zero` when `value === 0`, `.progress--complete` when `value === 100` for CSS boundary handling

### Tokens (unprefixed names match `_TOKENS.md` convention)
- [ ] `--primary` → indicator color
- [ ] `--secondary-container` → track color
- [ ] `--corner-full` → border-radius (linear)
- [ ] `--duration-long2` → animation cycle period
- [ ] `--easing-emphasized` → animation easing

### Decision: `buffer` variant
- [ ] Resolve: M3 has no `buffer` variant; kafUI v1 will not ship it. Document in SPEC as "M3 extension, not in scope". If added later: `bufferValue` prop, second track layer in `.progress__buffer`, same token system.

### Decision: circular size presets
- [x] Named aliases `"sm"=24`, `"md"=48`, `"lg"=64` — resolved in SPEC. Document px equivalents in Storybook.

### Tests
- [ ] Unit: `value=50` → `aria-valuenow="50"` on root
- [ ] Unit: `indeterminate` → no `aria-valuenow`, no `aria-valuemin`, no `aria-valuemax`
- [ ] Unit: `value=100` → `.progress--complete` modifier present; stop indicator absent
- [ ] Unit: `value=0` → `.progress--at-zero` present; stop indicator absent
- [ ] Unit: `size="sm"` → `--_circ-size: 24px` in inline style
- [ ] Visual regression: linear 0 / 50 / 100; circular 0 / 50 / 100; both indeterminate states
- [ ] A11y: missing `aria-label` triggers dev warning

### Storybook
- [ ] Linear determinate (slider control for value)
- [ ] Linear indeterminate
- [ ] Circular determinate (slider control)
- [ ] Circular indeterminate
- [ ] Size variants (sm / md / lg)
- [ ] RTL (linear — confirm inline-start fill direction)
- [ ] Reduced motion
- [ ] Dark mode
