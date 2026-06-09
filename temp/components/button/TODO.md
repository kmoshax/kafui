# Button — TODO

## MUI equivalent
`@mui/material/Button` (also `LoadingButton` from `@mui/lab` for async state).

MUI Button key reference: `variant` in `{text,outlined,contained}` (not M3 names), `color` prop accepting palette keys, `startIcon`/`endIcon` as ReactNode slots, Emotion CSS-in-JS, no M3 shape morph, no Expressive sizes.

---

## Beat-MUI opportunities

### 1. Static CSS = zero runtime style cost
**What we do:** all button styles live in `@layer kafui { … }` in `@kafui/styles` — a static `.css` file. No `<style>` tags injected, no Emotion runtime, no `sx` prop evaluation.

**Why it wins:** MUI's Emotion layer adds ~15 kB gzipped to the JS bundle and forces SSR hydration coordination. Our button costs **0 JS bytes** for styling. Storybook loads instantly; Next.js SSR needs no special MUI adapter.

**Task:** Ship `packages/styles/button.css` as a side-effect import. Benchmark: `size-limit` should show < 2 kB JS per button import.

### 2. Variant = color + shape in one prop (no leaky `color` escape hatch)
**What we do:** `variant` encodes the exact M3 container/label/border roles. There is no `color` prop accepting arbitrary palette keys.

**Why it wins:** MUI's `color="success"` on a Button is off-spec and lets consumers produce non-M3 UIs accidentally. Our type union `'filled' | 'filled-tonal' | 'elevated' | 'outlined' | 'text'` makes the M3 constraint compiler-enforced. Wrong variant = TypeScript error, not a silent style miss.

**Task:** Ensure TypeScript `ButtonVariant` type is exported and used in all consuming components (ButtonGroup, SplitButton). Document the intentional omission of `color` in the Storybook argTypes.

### 3. `icon` string prop beats ReactNode `startIcon`/`endIcon` slots
**What we do:** `icon?: string` accepts an icon sprite name. `trailingIcon?: boolean` flips order via CSS `order: 1` on `.button__icon` — no DOM restructuring, no two-slot API.

**Why it wins:** MUI's `startIcon={<SomeIcon />}` forces consumers to import per-icon JS modules (tree-shaking helps, but the API encourages importing icon packages). Our sprite-based `<Icon name="…" />` is a single HTTP request; order is a CSS concern not a structural one. API is 1 prop (string) vs 2 props (ReactNode each).

**Task:** Implement `<Icon name>` sprite component. Implement `.button--trailing-icon .button__icon { order: 1; }` in CSS. Remove any `startIcon`/`endIcon` leakage from the types.

### 4. Touch target as pure CSS — no wrapper div
**What we do:** `::before` pseudo-element with `inset: min(0px, calc((48px - 100%) / 2))` expands the hit area to 48 dp minimum without adding a wrapper element or changing visual layout.

**Why it wins:** MUI wraps small buttons in a `<span>` with extra padding to meet touch targets, bloating the DOM and complicating layout containment. Our approach is invisible to layout engines and requires zero extra DOM nodes.

**Task:** Implement the `::before` touch-target pseudo on `.button`. Write a Storybook accessibility story that visualises the touch target rectangle for each size.

### 5. Expressive shape morph — M3-native, zero JS
**What we do:** `border-radius` transitions on `[data-pressed]` via CSS transition. Duration and easing come from `--duration-short2` and `--easing-standard`.

**Why it wins:** MUI has no shape morph at all; adding it requires custom `styled()` overrides and JS event handlers. Ours is 3 lines of CSS inside a `@media (prefers-reduced-motion: no-preference)` guard.

**Task:** Implement the transition block. Test in Storybook: confirm the morph is visible at `lg`/`xl` (where radius changes from `--corner-extra-large` to `--corner-full`), and absent at `xs`/`sm`/`md` (always `--corner-full`, so morph is a subtle spring vs flat). Consider: should the morph at `md` animate to a slightly *squished* radius (e.g. `--corner-large`) for visual spring effect? Raise with lead as a design decision.

### 6. `aria-disabled` keeps the button in tab order — MUI removes it
**What we do:** React Aria's `isDisabled` prop sets `aria-disabled="true"` and suppresses press events without removing the element from the tab order.

**Why it wins:** MUI's `disabled` HTML attr causes the button to vanish from keyboard navigation — users can't discover that an action exists but is unavailable. This matters for form wizards (step 2 button is disabled until step 1 is complete — users should be able to find it and receive contextual feedback via tooltip).

**Task:** Confirm RAC `Button` with `isDisabled` emits `aria-disabled` not `disabled`. Add a Storybook story that renders a disabled button and asserts it appears in the tab sequence. Document the behaviour in the component's JSDoc.

### 7. One source color → all button color tokens via OKLCH — no manual palette config
**What we do:** `--primary`, `--on-primary`, `--secondary-container`, etc. all derive from `--source` via OKLCH relative-color syntax in `@kafui/styles/tokens.css`. Changing the brand color is `--source: oklch(0.55 0.18 145)` on any scope.

**Why it wins:** MUI requires `createTheme({ palette: { primary: { main: '#...' } } })` in JS, which means re-rendering the theme context, SSR serialisation of the theme, and manual dark-mode palette definition. Ours is a single CSS custom property — works in static HTML, no JS context needed.

**Task:** Wire up button color token variables to the OKLCH pipeline in `tokens.css`. Provide a Storybook "theme switcher" knob that sets `--source` on the preview root and demonstrates all five button variants updating live.

### 8. Correct `outlined` border color — `outline`, not `primary`
**What we do:** `.button--outlined` sets `border: 1px solid var(--outline)`.

**Why it wins:** MUI's outlined button defaults its border to `theme.palette.primary.main` (the primary brand color), which is off-M3 spec. M3 specifies `outline` system role — a neutral boundary color. This matters for accessibility (contrast with surface) and for design coherence when `--primary` is very saturated.

**Task:** Confirm CSS uses `var(--outline)` for the border. Add a visual regression test at the `outlined` variant level. Note: `outlined` border in disabled state must use `color-mix(in srgb, var(--on-surface) 12%, transparent)` (not `--outline`).

### 9. Disabled state via `color-mix()` — no separate token values needed
**What we do:** disabled container = `color-mix(in srgb, var(--on-surface) 12%, transparent)`, disabled label = `color-mix(in srgb, var(--on-surface) 38%, transparent)`.

**Why it wins:** MUI requires consumers to override `Mui-disabled` class with theme `styleOverrides`, or define explicit disabled palette colors. Our approach derives disabled appearance from one role (`--on-surface`) using native CSS — no extra tokens, no overrides, correct in both light and dark via `light-dark()`.

**Task:** Implement `color-mix()` expressions for disabled. Verify in dark mode: `--on-surface` in dark scheme is light-toned, so 12%/38% alpha will produce the correct translucent light-on-dark appearance.

---

## Checklist

- [ ] **[Beat #1]** Ship `button.css` as static side-effect import; add `size-limit` check for < 2 kB JS.
- [ ] **[Beat #2]** Export `ButtonVariant` type; document no-`color`-prop decision in Storybook docs.
- [ ] **[Beat #3]** Implement `<Icon name>` sprite component; implement `--trailing-icon` via CSS `order`.
- [ ] **[Beat #4]** `::before` touch target on `.button`; visual Storybook story per size.
- [ ] **[Beat #5]** Shape morph CSS transition; design decision: morph direction at `md` size (raise with lead).
- [ ] **[Beat #6]** Confirm `aria-disabled` output; tab-order Storybook story; JSDoc note.
- [ ] **[Beat #7]** Wire button tokens to OKLCH pipeline; Storybook theme-switcher knob.
- [ ] **[Beat #8]** Verify `var(--outline)` for outlined border; visual regression test.
- [ ] **[Beat #9]** Implement `color-mix()` disabled expressions; dark-mode visual test.
- [ ] Implement all five variant modifier classes in CSS.
- [ ] State-layer element driven by `data-hovered`, `data-focused`, `data-pressed`.
- [ ] Ripple keyframe animation (CSS clip-circle expand from press origin) — no JS ripple library.
- [ ] Elevation transitions for `elevated` variant using `--elevation-{n}` tokens.
- [ ] Expressive size modifiers `--size-xs` through `--size-xl`.
- [ ] RTL: verify icon appears correct side under `dir="rtl"` without JS.
- [ ] Focus ring appears only on keyboard navigation (`:focus-visible`), not mouse click.
