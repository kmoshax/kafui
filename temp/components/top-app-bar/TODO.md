# Top App Bar — TODO

## MUI equivalent

`@mui/material/AppBar` + `@mui/material/Toolbar`. MUI has no `medium`/`large` collapsible variants; no M3 `surface`→`surface-container` scroll-color shift; no M3 typescale mapping; on-scroll behavior requires `useScrollTrigger` + manual elevation prop wiring by the consumer.

---

## Beat-MUI opportunities

| Dimension | MUI pain | kafUI win |
|---|---|---|
| **Variants** | One generic box; consumer manually sets height, typescale, layout for each M3 variant | Ship `small`, `center-aligned`, `medium`, `large` with correct heights, typescales, headline positions out of the box |
| **Scroll behavior** | `useScrollTrigger` + `ElevationScroll`/`HideOnScroll` wrappers, separate import, manual elevation prop; background color never changes | `isScrolled` prop or zero-boilerplate `scrollTarget` sentinel; background shift `surface→surface-container` + shadow baked into `--scrolled` modifier — correct M3 spec, one prop |
| **Medium/Large collapse** | Fully custom implementation required; no helpers | CSS-only collapse via `max-block-size` + `padding` transition on `--scrolled`; collapsed inline headline via `--collapsed` clone; no consumer JS |
| **Token fidelity** | AppBar default color is `primary`, not `surface`; overriding to M3 surface requires `styleOverrides.AppBar`; dark mode needs ThemeProvider re-config | Resting `--surface`, scrolled `--surface-container`, dark via `light-dark()` — zero override needed |
| **A11y: role** | MUI Toolbar applies `role="toolbar"` — **wrong** for a top app bar whose trailing icons are not a tools group | Trailing slot is a plain flex `<div>`, no `role="toolbar"`; each icon button has its own `aria-label`; `<header>` landmark correct |
| **A11y: heading** | No guidance or prop for making the headline a real heading element | `headingLevel` prop; collapsed `__headline--collapsed` clone has `aria-hidden`; screen reader always reads the real heading |
| **RTL** | Some Toolbar versions hardcode `marginLeft`/`marginRight`; icon flip not handled | Logical CSS properties throughout; back-arrow and hamburger mirrored via `:dir(rtl)` |
| **Dark mode** | Requires `<ThemeProvider mode="dark">`; redefine all palette keys | `color-scheme` flip; `light-dark()` handles every role — no ThemeProvider, no duplication |
| **Runtime cost** | CSS-in-JS: Emotion injects styles per instance | Zero-runtime CSS; `@layer kafui` contains everything; `IntersectionObserver` fires only on threshold crossings — no scroll jank |

---

## Actionable TODO checklist

### Styles (`@kafui/styles`) — all inside `@layer kafui`

- [ ] Define component-internal vars on `.top-app-bar`: `--h: 64px`, `--content-h: 0px`, `--bg: var(--surface)`.
- [ ] `.top-app-bar--medium` sets `--content-h: 48px`; `.top-app-bar--large` sets `--content-h: 88px`.
- [ ] Base: `display: flex; flex-direction: column; width: 100%; background: var(--bg); box-shadow: var(--elevation-0); transition: background-color var(--duration-short4) var(--easing-standard), box-shadow var(--duration-short4) var(--easing-standard);`
- [ ] `__row`: `display: flex; align-items: center; height: var(--h); padding-inline: 4px; flex-shrink: 0;`
- [ ] `__leading`: `width: 48px; height: 48px; flex-shrink: 0; display: flex; align-items: center; justify-content: center; color: var(--on-surface);`
- [ ] `__headline` (inline): `flex: 1; color: var(--on-surface); font: var(--title-large-weight) var(--title-large-size)/var(--title-large-line-height) var(--title-large-font); padding-inline-start: 4px; overflow: hidden; text-overflow: ellipsis; white-space: nowrap;`
- [ ] `--center-aligned` headline: `text-align: center; padding-inline: 0;`
- [ ] `__trailing`: `display: flex; align-items: center; margin-inline-start: auto; color: var(--on-surface-variant);`
- [ ] `__content` (medium/large): `overflow: hidden; max-block-size: var(--content-h); padding-inline: 16px; padding-block-end: 28px; transition: max-block-size var(--duration-medium2) var(--easing-emphasized), padding var(--duration-medium2) var(--easing-emphasized);`
- [ ] `--medium .top-app-bar__content .top-app-bar__headline`: `headline-small` typescale; `white-space: normal;`
- [ ] `--large .top-app-bar__content .top-app-bar__headline`: `headline-medium` typescale; `white-space: normal;`
- [ ] `--scrolled` modifier: `--bg: var(--surface-container); box-shadow: var(--elevation-2);`
- [ ] `--scrolled .top-app-bar__content`: `max-block-size: 0; padding-block-end: 0;`
- [ ] `__headline--collapsed` (inside `__row` for medium/large): `opacity: 0; pointer-events: none; transition: opacity var(--duration-medium2) var(--easing-emphasized);`
- [ ] `--scrolled .top-app-bar__headline--collapsed`: `opacity: 1;`
- [ ] `[dir="rtl"]` overrides for `__leading`/`__trailing` logical swap.
- [ ] `@media (prefers-reduced-motion: reduce)`: remove all transitions from `.top-app-bar`, `__content`, `__headline--collapsed`.

### React wiring (`@kafui/react`)

- [ ] Render root as `<header>`; forward `aria-label`.
- [ ] `headingLevel` prop: render headline as `<h1>`…`<h6>` or `<span>` (default). Apply same CSS class; font override via BEM modifier `.top-app-bar__headline--h1` etc. if needed, or just the heading element's default styles are suppressed.
- [ ] For medium/large: render `__content > __headline` (real, accessible) AND `__row > __headline--collapsed` with `aria-hidden="true"`.
- [ ] Export `useScrollSentinel(scrollTarget: Element | null): boolean` hook — shared with BottomAppBar and Toolbar. Creates a zero-height `<div>` at top of `scrollTarget`, attaches `IntersectionObserver`, returns `isScrolled`.
- [ ] `isScrolled` prop (controlled) takes precedence over sentinel state.
- [ ] `scrollTarget` prop triggers `useScrollSentinel`; fires `onScrolledChange` on change.
- [ ] Do NOT wrap trailing slot in `role="toolbar"`.
- [ ] Apply `data-scrolled` attribute mirroring `isScrolled` for consumer CSS hooks if needed (bonus).

### Scroll behavior edge cases

- [ ] Page loads already scrolled (not just scroll events) — `IntersectionObserver` fires synchronously on connect when sentinel is already out of viewport; verify `--scrolled` is set on first render.
- [ ] Snap-to-top re-expansion: removing `--scrolled` must trigger smooth re-expansion of `__content`. Test that `max-block-size` transition runs in reverse.
- [ ] Medium/large: confirm collapsed inline headline (`title-large`) does not clash with expanded headline node in DOM during transition. Verify no duplicate screen reader announcement.
- [ ] Test `scrollTarget` cleanup on unmount (disconnect observer).

### Testing & QA

- [ ] `<header>` landmark announced by screen reader (NVDA, VoiceOver, JAWS).
- [ ] All four variants at 360 dp and 1440 dp widths.
- [ ] Trailing icon buttons have individual `aria-label`; none default to "button".
- [ ] `--scrolled` modifier present on initial render when page already scrolled.
- [ ] Dark mode: `surface-container` is the correct dark-scheme tonal value via `light-dark()`.
- [ ] All interactive children use `onPress` (RAC `Button`), not `onClick`.
- [ ] RTL: leading/trailing visually flip; back-arrow icon mirrors.
- [ ] `prefers-reduced-motion: reduce` — no animated collapse, no background transition.
- [ ] `headingLevel={1}` renders `<h1>` with no default browser margin/font override bleed.
