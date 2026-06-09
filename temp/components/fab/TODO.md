# FAB — TODO

## MUI equivalent
`@mui/material/Fab`

MUI Fab key reference: `color` in `{default, primary, secondary, inherit}` (not M3 roles), `size` in `{small, medium, large}` (dimensions don't match M3 dp), `variant` in `{circular, extended}` (Extended FAB is the same component with a `variant` switch — no dedicated API), no `lowered` support, `href` allowed (turns FAB into anchor), Emotion CSS-in-JS.

---

## Beat-MUI opportunities

### 1. All four M3 color roles — not MUI's opaque `color="default"`
**What we do:** `color` prop accepts the four M3-specified values: `'surface' | 'primary' | 'secondary' | 'tertiary'`. Each maps directly to a container/icon token pair from the system palette. `surface` is the default.

**Why it wins:** MUI's `color="default"` maps to `theme.palette.grey[300]` — that is not M3 and produces an off-spec appearance. MUI also has no `tertiary` at all. Our four-value union makes every valid M3 FAB expressible and every invalid one a TypeScript error.

**Task:** Implement `.fab--primary`, `.fab--secondary`, `.fab--tertiary` modifier classes, each setting `--_bg` and `--_fg` local variables. The default `.fab` class encodes the `surface` variant (no modifier needed). Add a Storybook story showing all four side by side to visually validate tonal palette generation.

### 2. `aria-label` required at the type level — compile-time a11y
**What we do:** `'aria-label': string` is a non-optional property in `FabProps`. The TypeScript compiler rejects any usage that omits it.

**Why it wins:** MUI's `Fab` has `aria-label` as an optional `string | undefined`. An icon-only FAB without `aria-label` fails WCAG 1.1.1 (Non-text Content) — but MUI will happily compile and render it. We push the error left, to compile time.

**Task:** Verify the TypeScript interface has `'aria-label': string` (not `?: string`). Add an ESLint rule (or Storybook a11y addon check) that catches `aria-label` absence at the story level. Document this in the component's JSDoc: _"aria-label is required because FABs have no visible text label"_.

### 3. `lowered` prop — M3 Expressive feature MUI ignores entirely
**What we do:** `lowered` boolean drops resting elevation from 3 → 1 and hover from 4 → 2. This signals reduced prominence in a layered layout (e.g. FAB over a modal sheet, or in a bottom app bar context).

**Why it wins:** MUI has no concept of `lowered`. Consumers who want this must override box-shadow via `sx` prop or `styleOverrides` — opaque and fragile. Our one-prop approach maps to `.fab--lowered` CSS which resets the elevation custom variables. Zero JS logic.

**Task:** Implement `.fab--lowered` and `.fab--lowered[data-hovered]` elevation rules. Add a Storybook story showing normal vs lowered FAB side-by-side. Document use case: "use `lowered` when the FAB sits inside a container that already carries elevation (e.g. bottom app bar)".

### 4. Correct dp dimensions + corner radii — MUI's are approximate
**What we do:** small = 40×40 dp / `--corner-medium`, medium = 56×56 dp / `--corner-large`, large = 96×96 dp / `--corner-extra-large`. Sizes are CSS custom properties internally, so a design-token update propagates everywhere.

**Why it wins:** MUI's `size="small"` renders at 40 dp but uses `theme.shape.borderRadius * 7` for corner radius — that's not tied to M3 corner roles and will drift from spec when someone customises the base radius. Our sizes are 1:1 M3 spec and use named shape tokens.

**Task:** Verify CSS `--_sz` and `--_radius` values match M3 spec exactly. Write a Playwright visual snapshot test for each size × color combination.

### 5. No `href` support — FABs are actions, not navigation
**What we do:** `FabProps` does not include `href`, `component`, or `to` props. The component is a `<button>`, always.

**Why it wins:** MUI allows `<Fab href="…">` which renders an `<a>` — this conflates navigation and action semantics, produces `role="link"` on what looks like a FAB, and causes AT to announce it incorrectly. M3 spec explicitly categorises FAB under "Actions", not navigation. If a FAB-shaped link is needed, the consumer should compose a `<Link>` with FAB styling at the application layer.

**Task:** Document this decision as a comment in the type definition file. Add a note in Storybook docs: "If you need a FAB-shaped navigation element, wrap an `<a>` in FAB styles — do not add href to Fab."

### 6. Scroll-hide contract documented — `useFabScrollHide()` hook provided
**What we do:** the component documents a `.fab--hidden` CSS class contract (`transform: scale(0)` + `opacity: 0`). A `useFabScrollHide(threshold?)` utility hook in `@kafui/hooks` returns a `hidden` boolean that consumers apply to the FAB.

**Why it wins:** MUI documents nothing about scroll-hide behavior; every MUI codebase reinvents it differently (some use JS `display: none`, some use Transform, some use Intersection Observer). We standardise the CSS contract and provide the hook — consistent motion spec, respects `prefers-reduced-motion`, and works with any scroll container.

**Task:** Implement `.fab--hidden` CSS with `transform: scale(0)` + `opacity: 0` transition. Implement `useFabScrollHide(threshold = 100)` hook using `IntersectionObserver` or `scroll` event. Wrap the transition in `@media (prefers-reduced-motion: no-preference)`. Add a Storybook story with a scrollable container demonstrating hide/show.

### 7. Static CSS elevation — no MUI shadow array
**What we do:** `--elevation-1` through `--elevation-4` are CSS custom properties in `tokens.css`. The exact `box-shadow` values (key + ambient) are defined once, used everywhere.

**Why it wins:** MUI's `theme.shadows[1..24]` is a JS array that gets embedded in the theme context and referenced at runtime. Our elevations are static CSS — no context provider, no JS reference, no SSR mismatch risk.

**Task:** Define `--elevation-1` through `--elevation-4` in `tokens.css` matching M3 spec shadow pairs. Confirm the FAB's elevation transitions are smooth (`transition: box-shadow`). Verify dark-mode: M3 specifies an additional surface tint overlay at elevation in dark scheme — document whether we implement this via a pseudo-element or a background gradient in `tokens.css`.

---

## Checklist

- [ ] **[Beat #1]** Implement four color variant modifier classes; Storybook story with all four.
- [ ] **[Beat #2]** Non-optional `aria-label` in type; ESLint/Storybook a11y check; JSDoc note.
- [ ] **[Beat #3]** Implement `--lowered` elevation rules; side-by-side Storybook story.
- [ ] **[Beat #4]** Verify dp + corner-radius values; Playwright visual snapshots.
- [ ] **[Beat #5]** Document no-`href` decision; Storybook docs note.
- [ ] **[Beat #6]** Implement `.fab--hidden`; implement `useFabScrollHide()` hook; scrollable Storybook story.
- [ ] **[Beat #7]** Define `--elevation-1..4` in `tokens.css`; verify dark-mode tint overlay strategy.
- [ ] Small FAB `::before` pseudo-element for 48 dp touch target.
- [ ] State-layer element driven by `data-hovered`, `data-focused`, `data-pressed`.
- [ ] Elevation transition: `box-shadow: var(--elevation-N)` animated with `transition`.
- [ ] `aria-disabled` via React Aria `isDisabled`; do NOT set HTML `disabled`.
- [ ] Verify M3 shadow key + ambient values per elevation level in `tokens.css`.
