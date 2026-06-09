# Loading Indicator ✦ M3 Expressive

**Purpose:** Signals an active short-duration loading state (target < 5 s) through a morphing, animated shape — providing an expressive visual that communicates "something is happening" with more personality than a plain spinner. Distinct from `ProgressIndicator`: no `value` prop, no determinate state, purely a "system is working" signal.
**M3 category:** Communication → Loading Indicator (M3 Expressive 2025).

---

## Anatomy / Parts → BEM Elements

### Contained variant

```
.loading-indicator                            root <div role="progressbar">
.loading-indicator__container                 sized wrapper (48×48 dp default)
.loading-indicator__track                     background surface (circle/squircle)
.loading-indicator__active                    morphing animated shape (SVG)
```

### Uncontained variant (`.loading-indicator--uncontained` on root)

```
.loading-indicator--uncontained
.loading-indicator__container                 sized wrapper (40×40 dp default)
.loading-indicator__active                    morphing animated shape only — no track
```

The `.loading-indicator__active` element is the SVG with `aria-hidden="true"` and `focusable="false"`. All animation lives on this element.

---

## Variants

| Variant | `variant` prop | Default size | Track? | When to use |
|---------|---------------|--------------|--------|-------------|
| Contained | `"contained"` | 48 × 48 dp | Yes — surface-container-high circle | Over content areas, cards, full-page loading |
| Uncontained | `"uncontained"` | 40 × 40 dp | No | Inline in app bars, next to text, over images, in tight spaces |

---

## The Morphing Animation — Phase Breakdown

The M3 Expressive animation cycles through a **4-phase loop** (~1400 ms total, `--easing-emphasized` throughout):

| Phase | Duration | Shape | Stroke behavior |
|-------|----------|-------|-----------------|
| 1 — Arc | ~350 ms | Circular partial arc (270° open) | Stroke-width constant ~4 dp; rotation building |
| 2 — Wavy | ~350 ms | Arc acquires organic waviness along path | Stroke-width begins breathing 4→6 dp |
| 3 — Squircle transition | ~350 ms | Path deforms toward squircle; arc gap widens | Stroke breathing continues 6→3 dp |
| 4 — Contract + expand | ~350 ms | Shape contracts to near-dot then expands back to arc | Stroke snap back to 4 dp; rotation continues |

Each phase transitions into the next via `@keyframes` on the SVG `<path d>` property. The full loop runs on `animation-iteration-count: infinite`.

**Animation implementation:**
- Primary: CSS `d` property keyframes on SVG `<path>` — `@keyframes morph { 0% { d: path("…"); } 25% { d: path("…"); } … }`. Browser support: Chrome 88+, Firefox 112+, Safari 16.4+ (acceptable floor; covers >96% of users as of 2026).
- Fallback (older browsers): SMIL `<animate attributeName="d" values="…" dur="1400ms" repeatCount="indefinite" />` inline in the SVG — no JS needed.
- Stroke-width breathing: separate `@keyframes breathe` targeting `stroke-width` attribute (or `stroke-width` CSS property on the `<path>`).
- Rotation: separate `@keyframes rotate` targeting `transform: rotate()` on the SVG or a `<g>` wrapper.

> **Implementation decision (record here):** CSS `d` property preferred. SMIL fallback via `<animate>` is declarative and requires no JS. A Lottie JSON opt-in is deferred to v2 — animation fidelity should come from well-crafted SVG keyframes first.

---

## Exit Lifecycle

The exit must feel intentional — the shape should *finish* before disappearing, not cut mid-morph.

```
complete={false}  →  loop plays
complete={true}   →  .loading-indicator--completing added
                  →  CSS checks current animation state: wait for
                     next "calm keyframe" (end of Phase 1 — plain arc)
                  →  exit animation: scale(1)→scale(0) + opacity 1→0
                  →  animationend fires → onExitComplete() called
```

**"Calm keyframe" strategy:** Add an intermediate `@keyframes completing` that, when applied, overrides the morph loop and eases the shape back to the plain arc position, then plays exit. This requires the completing animation to pick up naturally — use `animation-fill-mode: forwards` + `animation-delay: 0s` so it starts immediately from whatever visual state the loop is at. It won't be frame-perfect, but the transition to arc is fast (~200 ms) and then exit plays cleanly.

```css
.loading-indicator--completing .loading-indicator__active {
  animation:
    completing-to-arc var(--duration-short4) var(--easing-emphasized-accelerate) forwards,
    exit-fade         var(--duration-short4) var(--easing-emphasized-accelerate) var(--duration-short4) forwards;
}
```

---

## States

| State | Behavior |
|-------|----------|
| Loading (active) | Morphing loop plays continuously |
| Completing | `complete={true}` → `.loading-indicator--completing` → plays to calm arc → exits |
| Hidden | `aria-hidden="true"` set; element may be unmounted by parent via `onExitComplete` |

No hover/press states. Purely presentational.

---

## Design Tokens

| Token | CSS custom property (unprefixed) | Usage |
|-------|----------------------------------|-------|
| `md.sys.color.primary` | `--primary` | active indicator stroke color |
| `md.sys.color.surface-container-high` | `--surface-container-high` | contained track background |
| `md.sys.shape.corner.full` | `--corner-full` | contained track border-radius |
| `md.sys.motion.duration.long2` | `--duration-long2` | morphing cycle period (~1200–1400 ms) |
| `md.sys.motion.easing.emphasized` | `--easing-emphasized` | morph transition easing |
| `md.sys.motion.duration.short4` | `--duration-short4` | enter/exit fade |
| `md.sys.motion.easing.emphasized-decelerate` | `--easing-emphasized-decelerate` | enter easing |
| `md.sys.motion.easing.emphasized-accelerate` | `--easing-emphasized-accelerate` | exit easing |

**Component-internal variables** (scoped inside `.loading-indicator { … }`):
```css
.loading-indicator {
  --_size:       48px;   /* overridden by size prop */
  --_size-unc:   40px;   /* uncontained default */
  --_stroke-w:   4px;
  --_stroke-min: 3px;
  --_stroke-max: 6px;
  --_track-bg:   var(--surface-container-high);
  --_active-color: var(--primary);
}
```

---

## Interaction & Accessibility

### ARIA
```html
<div
  role="progressbar"
  aria-label="Loading"
  aria-valuemin="0"
  aria-valuemax="100"
  <!-- aria-valuenow intentionally omitted: indeterminate -->
>
  <div class="loading-indicator__container" aria-hidden="true">
    <!-- SVG shapes; aria-hidden, focusable="false" -->
  </div>
</div>
```

- `role="progressbar"` without `aria-valuenow` = indeterminate (most AT-compatible signal).
- Never set `aria-valuenow`/`aria-valuemin`/`aria-valuemax` — LoadingIndicator is always indeterminate.
- `aria-label` defaults to `"Loading"` (i18n key `loadingIndicator.label`); overridable via prop.
- When `complete={true}`, briefly update `aria-label` to `"Loaded"` (200 ms) before `onExitComplete` — or leave it to the parent context to announce (preferred; component should not linger visible after completion).
- Inner container and SVG: `aria-hidden="true"`.

### Alternative ARIA pattern
For section-level loading (e.g. a card content area loading), the preferred pattern is:
```html
<section aria-busy="true" aria-label="Loading search results">
  <LoadingIndicator aria-hidden="true" /> <!-- suppress duplicate announcement -->
</section>
```
Document this as the recommended hosting pattern in Storybook.

### Keyboard
Non-interactive — no keyboard target, no focus ring.

### Reduced motion
```css
@media (prefers-reduced-motion: reduce) {
  .loading-indicator__active {
    animation: none !important;
    /* Show static partial arc — Phase 1 shape */
  }
}
```
Never suppress entirely. Show a static partial-arc at reduced motion; the user needs to know loading is in progress.

### RTL
Rotation animation direction mirrors in RTL:
```css
:dir(rtl) .loading-indicator__active {
  animation-direction: reverse; /* only for rotation keyframe */
}
```
Shape morphing frames are neutral; only rotation needs reversal. Apply `animation-direction: reverse` only to the `rotate` animation, not the `morph` or `breathe` animations (use separate named animations for composability).

---

## Proposed kafUI React API

```tsx
interface LoadingIndicatorProps {
  /** Visual style. Default: "contained" */
  variant?: "contained" | "uncontained";
  /**
   * Size in px OR named alias.
   * "sm"=32, "md"=48 (contained default), "lg"=64, "xl"=80.
   * Uncontained default: 40px.
   */
  size?: number | "sm" | "md" | "lg" | "xl";
  /** Accessible label. Default: "Loading" (i18n key: loadingIndicator.label) */
  "aria-label"?: string;
  /**
   * When true, plays completing → exit animation, then calls onExitComplete.
   * Parent should unmount after onExitComplete fires.
   * Default: false
   */
  complete?: boolean;
  /** Called when exit animation finishes — use to unmount the component. */
  onExitComplete?: () => void;
  className?: string;
}

// Basic usage
<LoadingIndicator />

// Uncontained, in app bar
<LoadingIndicator variant="uncontained" aria-label="Syncing" />

// Lifecycle-managed
<LoadingIndicator
  complete={!isLoading}
  onExitComplete={() => setShowLoader(false)}
  aria-label="Loading search results"
/>

// Custom size
<LoadingIndicator size="sm" aria-label="Loading" />
```

**React Aria primitive:** None — purely presentational. No RAC primitive wraps a morphing animation. The `role="progressbar"` + ARIA attributes are set directly on the root `<div>`.

**BEM classes emitted:**
```
.loading-indicator  .loading-indicator--contained | .loading-indicator--uncontained
                    .loading-indicator--completing
.loading-indicator__container
.loading-indicator__track         (contained only)
.loading-indicator__active        (SVG element — aria-hidden)
```

**Layer:**
```css
@layer kafui { /* all loading-indicator styles */ }
```

**Why this beats MUI:**
MUI has no equivalent. The closest is `<CircularProgress variant="indeterminate">` — a constant-width rotating arc with no personality. The M3 Expressive loading indicator is a genuine differentiator: it ships what MUI cannot without Lottie or custom animation code. The `complete` + `onExitComplete` lifecycle pattern is cleaner than unmounting immediately (which creates jarring cuts).
