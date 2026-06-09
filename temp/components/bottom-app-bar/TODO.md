# Bottom App Bar — TODO

## MUI equivalent

No direct MUI core equivalent. Closest approximations — all inadequate:
- `AppBar` with `position="fixed" sx={{ top: "auto", bottom: 0 }}` — uses `banner` landmark (wrong), default color is `primary` (not M3 `surface-container`), no FAB notch/overlap, no scroll-hide.
- `BottomNavigation` — wrong component class; models navigation destinations, not actions.
- `Toolbar` inside a bottom `AppBar` — functionally close but wrong semantics, no M3 tokens, no roving focus.

---

## Beat-MUI opportunities

| Dimension | MUI pain | kafUI win |
|---|---|---|
| **Existence** | No M3 Bottom App Bar in MUI core at all | First-class component with correct `surface-container` background, `elevation-2`, FAB overlap, and M3 scroll-hide motion |
| **FAB integration** | Consumer manually positions FAB relative to AppBar with `Box`/`Fab` and z-index guesswork | `fab` slot prop — bar automatically switches to `--with-fab` layout; FAB overlap (`--fab-overlap: 8px`) baked into CSS; consumer just passes `<Fab>` |
| **Scroll hide** | No built-in; consumer writes scroll listener + `useScrollTrigger` + `Slide` wrapper | `isHidden` prop + `useScrollSentinel` hook (same hook as TopAppBar and Toolbar); CSS-only transition; no scroll jank |
| **A11y: role** | AppBar renders `<header role="banner">` — **wrong** for an action bar | `role="toolbar"` on a plain `<div>`; `aria-label` required at TypeScript level; no incorrect landmark |
| **A11y: roving focus** | No roving focus on AppBar contents; consumer must implement | Built-in `useRovingTabIndex` across actions AND FAB; `Home`/`End` work; disabled items skipped automatically |
| **Token fidelity** | AppBar default color is `primary`; overriding requires `styleOverrides.MuiAppBar` and palette reconfiguration | `--surface-container` out of the box; dark mode via `light-dark()` — zero config |
| **Motion spec** | `Slide`/`Fade` from MUI Transitions; not M3 motion tokens | `--easing-emphasized-accelerate` on hide; `--easing-emphasized-decelerate` on reveal; durations from M3 motion scale |
| **Migration to Toolbar** | No concept of migration; no Expressive successor documented | Explicit deprecation note + link to Toolbar spec; `migration` doc entry planned |

---

## Actionable TODO checklist

### Styles (`@kafui/styles`) — all inside `@layer kafui`

- [ ] `.bottom-app-bar` base: `position: fixed; inset-block-end: 0; inset-inline: 0; background: var(--surface-container); box-shadow: var(--elevation-2); transition: transform var(--duration-short3) var(--easing-emphasized-accelerate);`
- [ ] Define component-internal vars: `--bar-h: 80px; --fab-overlap: 8px;`
- [ ] `__content`: `display: flex; align-items: center; block-size: var(--bar-h); padding-inline: 4px;`
- [ ] `__actions`: `display: flex; align-items: center; gap: 0; color: var(--on-surface-variant);`
- [ ] `__fab`: `margin-inline-start: auto; margin-block-start: calc(-1 * var(--fab-overlap)); align-self: flex-start;`
- [ ] `--hidden` modifier: `transform: translateY(100%); transition-timing-function: var(--easing-emphasized-decelerate); transition-duration: var(--duration-medium2);` (reveal uses decelerate + longer duration)
- [ ] `@media (prefers-reduced-motion: reduce)`: replace transform with `opacity: 0; pointer-events: none; transform: none;` on `--hidden`; use `transition: opacity var(--duration-short2)`.
- [ ] Touch targets: `__action::before { content: ""; position: absolute; inset: -4px; }` to expand 48×48 dp minimum.
- [ ] State layer: `.state-layer::after` driven by RAC `data-hovered`/`data-focused`/`data-pressed`; colors `--on-surface-variant`.
- [ ] Verify `color-scheme: light dark` propagates; `--surface-container` resolves correctly in dark.

### React (`@kafui/react`)

- [ ] Root: `<div role="toolbar">` with `aria-label` prop (required in TypeScript — non-optional string, not `string | undefined`).
- [ ] `BottomAppBar.Action` uses RAC `<Button>`; emits `.bottom-app-bar__action`; passes `aria-label` (required), `isDisabled`, `onPress`.
- [ ] `isSelected` → `aria-pressed` on `BottomAppBar.Action`.
- [ ] Implement `useRovingTabIndex(items: RefObject[], options: { wrap: boolean })` utility — manages `tabIndex` across both actions and the FAB as a single focus group; handles `ArrowLeft`/`ArrowRight`/`Home`/`End` and disabled skipping.
- [ ] `rovingFocus` prop (default `true`): when `false`, skip roving and use natural tab order.
- [ ] Wire `isHidden` prop → `.bottom-app-bar--hidden` class.
- [ ] `fab` prop truthy → add `.bottom-app-bar--with-fab`; render `__fab` wrapper.
- [ ] Export `useScrollSentinel` hook from `@kafui/react` (shared with TopAppBar and Toolbar) — returns `{ isAtTop: boolean }` based on `IntersectionObserver` on a zero-height sentinel at top of `scrollTarget` (default: `document.documentElement`). Note: for BottomAppBar, scroll-DOWN should hide — requires scroll direction detection, not just position. Implement direction tracking (compare `scrollY` on each `IntersectionObserver` or `scroll` event).
- [ ] Rename `hidden` → `isHidden` (aligns with RAC naming convention: `isDisabled`, `isSelected`).
- [ ] Export `BottomAppBar` and `BottomAppBar.Action` from package index.

### Scroll direction detection (BottomAppBar differs from TopAppBar)

- [ ] TopAppBar uses position-based detection (sentinel at top of content, fires when out of viewport).
- [ ] BottomAppBar uses **direction-based** detection: hide on scroll down, reveal on scroll up. The `useScrollSentinel` hook must accept a `mode: 'position' | 'direction'` option to support both.
- [ ] Document this distinction clearly in the hook's JSDoc.

### Testing & QA

- [ ] `role="toolbar"` + `aria-label` announced on focus entry (NVDA/VoiceOver).
- [ ] 3 actions: `ArrowRight` moves focus sequentially; wraps from last to first.
- [ ] `Home` focuses first action; `End` focuses last (or FAB if present).
- [ ] `isDisabled` on action: `aria-disabled="true"`, `tabindex="-1"`, skipped in arrow-key navigation.
- [ ] `isHidden={true}`: `.bottom-app-bar--hidden` present; bar off-screen; focus inaccessible.
- [ ] `fab` present: `.bottom-app-bar--with-fab` applied; FAB in roving focus order after last action.
- [ ] FAB participates in arrow-key navigation from last action button.
- [ ] Touch target on each action meets 48×48 dp (visual + `::before`).
- [ ] RTL (`dir="rtl"`): actions at `inline-start`, FAB at `inline-end`.
- [ ] `prefers-reduced-motion: reduce`: no transform transition; opacity-only.
- [ ] Dark mode: `surface-container` resolves to correct dark tonal value.

### Documentation / migration

- [ ] Component-level doc note: "Bottom App Bar is superseded by Docked Toolbar in M3 Expressive — prefer `<Toolbar variant="docked">` for new screens. This component is maintained for M3 baseline conformance."
- [ ] Migration guide entry: `<BottomAppBar>` → `<Toolbar variant="docked">` — map `fab` slot, `isHidden`, action children.
