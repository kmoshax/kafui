# Loading Indicator ✦ — TODO

## MUI Equivalent
**None.** Nearest is `@mui/material/CircularProgress` (indeterminate) — a constant-width rotating arc with zero morphing. kafUI ships what MUI cannot: M3 Expressive shape morphing, a four-phase animation sequence, and a `complete` + `onExitComplete` lifecycle. This component is a primary **visual differentiation point** for kafUI.

---

## Beat-MUI Opportunities

| # | kafUI wins by… | vs MUI |
|---|----------------|--------|
| 1 | **Morphing 4-phase shape animation** (arc → wavy → squircle → dot) — expressive, brand-differentiating, impossible with MUI CircularProgress | MUI: constant-width rotating arc; identical to every other spinner on the web |
| 2 | **`complete` + `onExitComplete` lifecycle** — animation plays to a natural "calm" exit before the element disappears; no jarring cuts | MUI: unmount or `hidden` immediately — visual pop, no graceful exit |
| 3 | **`aria-busy` hosting pattern documented** — correct a11y for section-level loading without duplicate announcements | MUI docs show standalone CircularProgress with no guidance on `aria-busy` composition |
| 4 | **Named size aliases `"sm"|"md"|"lg"|"xl"`** — semantic meaning without memorizing px values | MUI `size` prop is raw px; `40` and `48` look the same in code |
| 5 | **Separate `contained` / `uncontained` variants** — right surface for every context; track background is a token swap | MUI has no contained/uncontained distinction; surface is always transparent |

---

## Actionable TODO Checklist

### Research / Design (must complete before animation work)
- [ ] Reconstruct exact M3 Expressive morphing SVG path data for all 4 phases from Figma M3 Expressive kit or m3.material.io
- [ ] Document exact durations per phase (confirm total cycle duration: ~1400 ms)
- [ ] Extract stroke-width breathing curve (min ~3 dp, max ~6 dp; which phase triggers peak?)
- [ ] Define the "calm keyframe" (Phase 1 arc end state) — the reference shape used for exit
- [ ] Verify which animation properties to composite independently: `d` (morph), `stroke-width` (breathe), `transform: rotate()` (spin)

### Core implementation
- [ ] `packages/react/src/loading-indicator/LoadingIndicator.tsx`
- [ ] `packages/styles/src/loading-indicator/loading-indicator.css` inside `@layer kafui { … }`
- [ ] Resolve `size` alias: `"sm"→32`, `"md"→48`, `"lg"→64`, `"xl"→80`
- [ ] Uncontained default size: 40 px (override if `size` explicitly provided)
- [ ] Inject `--_size` via inline style: `style={{ "--_size": resolvedSize + "px" } as React.CSSProperties}`
- [ ] `complete={true}` → add `.loading-indicator--completing`
- [ ] Listen for `animationend` on `.loading-indicator__active` when `.loading-indicator--completing` is active → call `onExitComplete()`

### Animation — CSS keyframes
- [ ] `@keyframes li-morph` — SVG `d` property transitions across 4 phases (0% / 25% / 50% / 75% / 100%)
- [ ] `@keyframes li-breathe` — `stroke-width` from `var(--_stroke-w)` → `var(--_stroke-max)` → `var(--_stroke-min)` → `var(--_stroke-w)` in sync with phases 2–3
- [ ] `@keyframes li-rotate` — `transform: rotate(0deg)` → `rotate(360deg)` at constant speed (~800 ms period)
- [ ] Compose on `.loading-indicator__active`: `animation: li-morph var(--duration-long2) var(--easing-emphasized) infinite, li-breathe … infinite, li-rotate 800ms linear infinite`
- [ ] RTL: `:dir(rtl) .loading-indicator__active { }` — apply `animation-direction: reverse` to `li-rotate` only (not morph or breathe). Technique: separate `animation-name` list allows per-animation direction override with `animation-direction: normal, normal, reverse`.
- [ ] SMIL fallback: add `<animate attributeName="d" …>` inside SVG for browsers without CSS `d` support (progressive enhancement; CSS takes precedence when supported)
- [ ] `prefers-reduced-motion`: `animation: none !important`; freeze at static Phase-1 arc shape (hardcode the Phase-1 `d` value as the default static state)

### Exit animation
```css
@layer kafui {
  .loading-indicator--completing .loading-indicator__active {
    animation:
      li-to-calm   var(--duration-short4) var(--easing-emphasized-accelerate)           forwards,
      li-exit-fade var(--duration-short4) var(--easing-emphasized-accelerate)
                   var(--duration-short4) forwards; /* delayed by short4 to chain after to-calm */
  }

  @keyframes li-to-calm {
    to { d: path("/* Phase-1 arc path data */"); stroke-width: var(--_stroke-w); }
  }

  @keyframes li-exit-fade {
    from { opacity: 1; transform: scale(1); }
    to   { opacity: 0; transform: scale(0); }
  }
}
```
- [ ] Implement above; test that `animationend` fires reliably after both chained animations
- [ ] Consider using a `Promise`-based exit in the React component: wrap `animationend` in a `useEffect` cleanup pattern

### Styles — Contained
```css
@layer kafui {
  .loading-indicator {
    display: inline-flex;
    align-items: center;
    justify-content: center;
  }

  .loading-indicator__container {
    width: var(--_size, 48px);
    height: var(--_size, 48px);
    position: relative;
  }

  .loading-indicator__track {
    position: absolute;
    inset: 0;
    background: var(--_track-bg, var(--surface-container-high));
    border-radius: var(--corner-full);
  }

  .loading-indicator__active {
    position: absolute;
    inset: 0;
    width: 100%;
    height: 100%;
    stroke: var(--_active-color, var(--primary));
    fill: none;
    stroke-linecap: round;
    stroke-width: var(--_stroke-w, 4px);
    overflow: visible;
  }
}
```

### Styles — Uncontained
```css
@layer kafui {
  .loading-indicator--uncontained .loading-indicator__container {
    width: var(--_size, 40px);
    height: var(--_size, 40px);
  }

  .loading-indicator--uncontained .loading-indicator__track {
    display: none;
  }
}
```
- [ ] Implement all above styles

### Accessibility
- [ ] `role="progressbar"` on root; NO `aria-valuenow`, `aria-valuemin`, `aria-valuemax`
- [ ] Default `aria-label="Loading"` from i18n key `loadingIndicator.label`
- [ ] Inner `.loading-indicator__container` and SVG: `aria-hidden="true"`, SVG gets `focusable="false"` (IE11 SVG focus quirk — belt-and-suspenders)
- [ ] When `complete={true}`: update `aria-label` to `"Loaded"` before `onExitComplete` fires — allows AT to announce completion. Implement via state: `const label = complete ? completedLabel : loadingLabel`
- [ ] Export `loadingIndicator.label` and `loadingIndicator.completedLabel` as i18n keys
- [ ] Document `aria-busy` hosting pattern in Storybook (section-level loading with `<LoadingIndicator aria-hidden="true">` inside an `aria-busy` container)

### Tokens (unprefixed CSS custom props)
- [ ] `--primary` → active indicator stroke
- [ ] `--surface-container-high` → contained track background
- [ ] `--corner-full` → contained track border-radius
- [ ] `--duration-long2` → animation cycle period
- [ ] `--easing-emphasized` → morph + breathe easing
- [ ] `--duration-short4` → enter/exit fade
- [ ] `--easing-emphasized-decelerate` → enter
- [ ] `--easing-emphasized-accelerate` → exit + completing

### Tests
- [ ] Unit: renders `role="progressbar"` with no `aria-valuenow`/`aria-valuemin`/`aria-valuemax`
- [ ] Unit: `aria-label` present and non-empty (default "Loading")
- [ ] Unit: `complete={true}` → `.loading-indicator--completing` class applied
- [ ] Unit: `onExitComplete` called after `animationend` fires on completing animation
- [ ] Unit: `complete={true}` → `aria-label` updates to "Loaded"
- [ ] Unit: `size="sm"` → `--_size: 32px` in inline style
- [ ] Unit: `variant="uncontained"` → no `.loading-indicator__track` in DOM
- [ ] Visual regression: contained static; uncontained static; completing state (static snapshot at exit keyframe)
- [ ] A11y: inner SVG has `aria-hidden="true"`

### Decisions resolved
- [x] Animation: CSS `d` property + SMIL fallback (no Lottie in v1)
- [x] Exit strategy: `completing-to-calm` + `exit-fade` chained keyframes; `animationend` triggers `onExitComplete`
- [x] Size aliases: `"sm"=32`, `"md"=48`, `"lg"=64`, `"xl"=80`; uncontained default = 40 px
- [x] No RAC primitive — purely presentational div with manual ARIA

### Open questions
- [ ] **Exact SVG path data** for all 4 morphing phases — must be obtained from M3 Expressive Figma kit or reconstructed from motion spec video. This is the single biggest implementation blocker.
- [ ] **`li-to-calm` approach** assumes the exit animation can override the loop mid-cycle cleanly — validate that `animation-fill-mode: forwards` on the completing animation actually wins over the looping animation in browsers without a race condition.
- [ ] **Lottie opt-in (v2)** — if SVG keyframe fidelity is insufficient, consider a `<LoadingIndicatorLottie>` wrapper component that accepts a Lottie JSON file. Keep the base component dependency-free.

### Storybook
- [ ] Contained (live animation)
- [ ] Uncontained (live animation)
- [ ] All size aliases side-by-side
- [ ] `complete` lifecycle demo (button triggers completion)
- [ ] Reduced motion (static)
- [ ] RTL (confirm rotation direction reversal)
- [ ] Dark mode
- [ ] `aria-busy` hosting pattern example
