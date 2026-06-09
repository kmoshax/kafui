# Toolbar (M3 Expressive) — TODO

## MUI equivalent

`@mui/material/Toolbar` — a `display: flex; align-items: center; padding: 0 16px` container with no ARIA role, no shape, no color semantics, no M3 anything. It is the internal layout helper for `AppBar`, not a standalone component. There is no MUI equivalent of the M3 Expressive Toolbar.

---

## Beat-MUI opportunities

| Dimension | MUI pain | kafUI win |
|---|---|---|
| **Existence** | MUI has nothing resembling M3 Expressive Toolbar — it would require `AppBar` + `Toolbar` + custom `position: fixed` + custom border-radius + custom colors + custom FAB positioning + hand-rolled roving focus | Ship a complete, spec-faithful component with zero assembly required |
| **Shape** | `borderRadius` prop on `Paper` — numeric, not token-based | `--corner-extra-large` (floating) / logical top-only radius (docked); overridable at CSS scope; shape tokens animate with `transition: border-radius` for morphing (Expressive motion) |
| **Vibrant colors** | `color="primary"` on `AppBar` changes entire bar to primary — does not match M3 container roles | `color` prop maps to `--primary-container`/`--on-primary-container` etc.; icon colors auto-correct to on-container roles; dark mode via `light-dark()` — correct contrast without extra config |
| **A11y: roving focus** | MUI Toolbar is a `<div>` with no role; consumer must add `role="toolbar"`, `aria-label`, and implement arrow-key roving from scratch. Common omission in practice. | RAC `Toolbar` provides `role="toolbar"`, `aria-label` forwarded (required type), roving tabindex, `←/→/Home/End`, and disabled-item skipping — correct out of the box |
| **FAB as first-class** | FAB dropped inside `Toolbar` as an arbitrary child; z-index, elevation, and roving focus management are consumer's problem | `fab` prop renders FAB in the same RAC Toolbar focus group with `__divider` separator; FAB elevation independent of container; `fabPosition` API enforces M3 layout |
| **Scroll hide/show** | Consumer writes scroll listener + `Slide` wrapper + `useScrollTrigger` | `isVisible` prop + shared `useScrollSentinel` hook; CSS-only M3 motion tokens; no scroll jank; asymmetric easing (accelerate on hide, decelerate on reveal) built in |
| **Runtime cost** | Emotion CSS-in-JS — style injection per instance | Zero-runtime CSS in `@layer kafui`; `IntersectionObserver` fires only on threshold crossings |
| **RTL** | `paddingLeft`/`paddingRight` in some MUI versions; no arrow-key flip | RAC `Toolbar` flips arrow key direction from `dir` attribute; `flex-direction: row-reverse` on `__actions`; directional icons mirrored via `:dir(rtl)` |
| **Dark mode** | `ThemeProvider mode="dark"` required; every color token must be re-declared | `light-dark()` for every role; `color-scheme` flip — zero ThemeProvider, zero duplication |

---

## Actionable TODO checklist

### Styles (`@kafui/styles`) — all inside `@layer kafui`

- [ ] Define component-internal vars on `.toolbar`: `--toolbar-bg`, `--toolbar-icon`, `--toolbar-gap: 16px`, `--toolbar-pad-x: 16px`, `--toolbar-pad-y: 12px`.
- [ ] `.toolbar` base: `display: inline-flex; align-items: center; background: var(--toolbar-bg); color: var(--toolbar-icon); padding: var(--toolbar-pad-y) var(--toolbar-pad-x); gap: 0; border-radius: var(--corner-extra-large); box-shadow: var(--elevation-2);`
- [ ] `.toolbar--docked`: `--toolbar-bg: var(--surface-container); --toolbar-pad-x: 8px; inline-size: 100%; justify-content: space-around; border-start-start-radius: var(--corner-extra-large); border-start-end-radius: var(--corner-extra-large); border-end-start-radius: var(--corner-none); border-end-end-radius: var(--corner-none); box-shadow: var(--elevation-0);`
- [ ] Color variants: `.toolbar--primary { --toolbar-bg: var(--primary-container); --toolbar-icon: var(--on-primary-container); }` — repeat for `--secondary`, `--tertiary`.
- [ ] `.toolbar__actions`: `display: flex; align-items: center; gap: 4px;`
- [ ] `.toolbar__divider`: `inline-size: 1px; block-size: 32px; background: var(--outline-variant); margin-inline: 4px; flex-shrink: 0;` with `aria-hidden="true"`.
- [ ] `.toolbar__fab` when `fabPosition="end"`: `order: 1;` (last in row, after `__actions` + `__divider`).
- [ ] Hide: `.toolbar--hidden { transform: translateY(calc(100% + var(--toolbar-gap))); transition: transform var(--duration-short4) var(--easing-emphasized-accelerate); }`
- [ ] Docked hide: `.toolbar--docked.toolbar--hidden { transform: translateY(100%); }`
- [ ] Show (not hidden): `transform: translateY(0); transition: transform var(--duration-medium2) var(--easing-emphasized-decelerate);`
- [ ] `@media (prefers-reduced-motion: reduce)`: `.toolbar, .toolbar--hidden { transform: none; transition: opacity var(--duration-short2); } .toolbar--hidden { opacity: 0; pointer-events: none; }`
- [ ] Document that `.toolbar` does NOT set `position: fixed`; consumers apply positioning via `className` or a utility class (e.g. `.toolbar-fixed-center-bottom { position: fixed; bottom: var(--toolbar-gap); left: 50%; transform: translateX(-50%); }`). Note the conflict between `transform: translateX(-50%)` and the hide/show `translateY` — resolve by composing transforms: `.toolbar-fixed-center-bottom.toolbar--hidden { transform: translateX(-50%) translateY(calc(100% + var(--toolbar-gap))); }`.

### React / RAC wiring (`@kafui/react`)

- [ ] Outer `.toolbar` renders as plain `<div>`; no role on the container.
- [ ] `__actions` renders as RAC `Toolbar` with `aria-label` forwarded. `aria-label` typed as required (not optional) — `role="toolbar"` without a label is an accessibility violation.
- [ ] `color` prop → adds `.toolbar--primary` / `--secondary` / `--tertiary` modifier; default is `.toolbar--surface` (no modifier needed since base styles cover surface).
- [ ] `fab` prop: render `__fab` before `__actions` (fabPosition `"start"`) or after (fabPosition `"end"`, default). Auto-render `__divider` between FAB and `__actions`.
- [ ] FAB rendered inside RAC `Toolbar` children (inside `__actions` subtree) as the first or last focusable child, so roving tabindex covers it. `__divider` is a non-focusable sibling (`aria-hidden`).
- [ ] `isVisible` prop: `false` → add `.toolbar--hidden`; default `true`.
- [ ] `onVisibleChange` fires when `isVisible` changes (for consumer scroll wiring).
- [ ] Enforce `aria-label` as required TypeScript string (never `string | undefined`).
- [ ] RTL: no JS changes — RAC `Toolbar` handles arrow-key direction from `dir` attribute.
- [ ] Export `useScrollSentinel(scrollTarget?: Element | null, mode?: 'position' | 'direction'): { isAtTop: boolean; isScrollingDown: boolean }` from `@kafui/react`. Shared by TopAppBar, BottomAppBar, Toolbar.
- [ ] Positioning note in component docs: "Do not expect `position: fixed` — apply via `className` or a wrapper."

### `useScrollSentinel` shared hook — design

- [ ] For TopAppBar: `mode: 'position'` — `IntersectionObserver` on a zero-height sentinel at top of `scrollTarget`; `isAtTop: false` when sentinel exits viewport.
- [ ] For BottomAppBar + Toolbar: `mode: 'direction'` — debounced `scroll` event comparing `scrollY` snapshots; `isScrollingDown: boolean`. Alternatively, dual `IntersectionObserver` thresholds (0% and some %) to detect direction without a persistent handler.
- [ ] One hook export; callers pick the returned field they need.
- [ ] Cleanup: `IntersectionObserver.disconnect()` on unmount.

### Testing & QA

- [ ] `Tab` enters toolbar; `←`/`→` rove between all buttons including FAB; `Home`/`End` work.
- [ ] Disabled icon button: `aria-disabled` announced by screen reader; skipped in arrow-key nav.
- [ ] Screen reader: `role="toolbar"` + `aria-label` announced on focus entry (NVDA, VoiceOver).
- [ ] Floating variant: correct pill shape, elevation shadow visible in light mode.
- [ ] Docked variant: flush bottom, top corners rounded only, no shadow.
- [ ] Color variant `"primary"`: background is `--primary-container`; icon color is `--on-primary-container`; dark mode resolves correctly via `light-dark()`.
- [ ] `isVisible={false}` → hide animation plays; buttons not focusable.
- [ ] Reduced motion: no transform; opacity-only hide/show.
- [ ] `fabPosition="start"` and `"end"` — visual positions correct; FAB in correct roving order.
- [ ] `__divider` is `aria-hidden="true"` and not focusable.
- [ ] RTL: icon order reverses; `←`/`→` keys work in mirror direction.
- [ ] Positioning transform composition: `.toolbar-fixed-center-bottom` does not break hide/show transform.
- [ ] FAB maintains own elevation (level 3 resting) inside floating toolbar (level 2); FAB hover lifts to level 4 while toolbar stays at 2.
- [ ] No `onClick` on any child — only `onPress` via RAC `Button`.
