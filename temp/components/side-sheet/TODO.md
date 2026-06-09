# Side Sheet — TODO

## MUI Equivalent

`Drawer` with `anchor="right"`. Modal variant ≈ `variant="temporary"`; standard variant ≈ `variant="persistent"` or `"permanent"`. No M3-native header structure. No `role="complementary"`. Requires manual `Box`/`Typography`/`IconButton` header composition.

---

## How kafUI Beats MUI

| Dimension | kafUI | MUI Drawer (anchor=right) |
|---|---|---|
| **Semantics** | Modal: `role="dialog"` + `aria-modal="true"`. Standard: `role="complementary"`. Semantically distinct per usage. | Always `role="presentation"` wrapping a plain `<div>` — loses both modal and landmark semantics entirely. |
| **Variants** | Two clear M3 variants: `standard` (inline, no scrim) and `modal` (overlay, scrim, focus-trapped). | Three variants (`temporary`/`persistent`/`permanent`) that partially overlap and conflate layout with modal behavior — confusing for new developers. |
| **Focus management** | RAC `Modal + Dialog` provides a spec-correct focus trap with return-focus to trigger. | MUI `Unstable_TrapFocus` has known issues with nested portals and shadow DOM. |
| **Header** | First-class `title`, `closeLabel`, `closeIcon`, `headerActions` props with BEM structure — zero boilerplate. | No built-in header; consumer composes `Box`/`Typography`/`IconButton` manually every time. |
| **CSS-only animation** | `@keyframes` + `translate` CSS property; enter/exit use distinct M3 easing; RAC `[data-entering]`/`[data-exiting]`. | JS `Slide` transition; Emotion runtime cost per drawer. |
| **RTL** | `inset-inline-end: 0` + logical `translate: 100% 0` — auto-correct in LTR and RTL; no JS, no `[dir]` overrides. | Requires `jss-rtl` / Emotion RTL transform plugin at build/runtime. |
| **M3 token fidelity** | `--surface-container-low`, `--corner-extra-large` on inline-start corners only, exact scrim opacity via `color-mix()`. | `background.paper`, no M3 shape, generic scrim. |
| **Anchor restriction** | Trailing-edge only — prevents misuse of side sheet for navigation (leading-edge is NavigationDrawer). | `anchor` allows all 4 sides — flexible but encourages M3 misuse (using Drawer for navigation and for detail panels interchangeably). |
| **Dark mode** | `light-dark()` + `color-scheme` — zero extra config. | Separate `ThemeProvider` dark palette. |
| **Back vs close icon** | `closeIcon='close'|'back'` prop — M3 detail panel pattern handled natively. | No built-in distinction; consumer renders the icon manually. |

**Biggest wins:**

1. **`role="complementary"` on standard variant** — MUI `Drawer` renders `role="presentation"` which is invisible to landmark navigation in AT. kafUI makes the standard side sheet a navigable landmark for free.
2. **Header as first-class anatomy** — every MUI Drawer implementation in the wild re-invents the same `Box + Typography + IconButton` header pattern. kafUI ships it once, correctly, with i18n (`closeLabel`) and icon flexibility (`closeIcon`).

---

## Actionable TODO Checklist

### Styles (`@kafui/styles`)

- [ ] Create `side-sheet.css`; wrap entirely in `@layer kafui { … }`
- [ ] Import / depend on shared `_scrim.css` for `.scrim` — do NOT redefine scrim
- [ ] Declare component-internal vars at top of `.side-sheet { --w: 360px; }` block
- [ ] Base `.side-sheet`: `position: fixed; inset-block: 0; inset-inline-end: 0; width: min(var(--w), 100vw); background: var(--surface-container-low); display: flex; flex-direction: column; outline: none`
- [ ] Inline-start corners only: `border-start-start-radius: var(--corner-extra-large); border-end-start-radius: var(--corner-extra-large); border-start-end-radius: 0; border-end-end-radius: 0`
- [ ] `.side-sheet--standard`: `position: relative; box-shadow: var(--elevation-1); z-index: auto` (layout-dependent; consumer must wrap in grid)
- [ ] `.side-sheet--modal`: `box-shadow: var(--elevation-2); z-index: var(--z-modal, 1300)`
- [ ] Enter: `@keyframes side-sheet-enter { from { translate: 100% 0 } to { translate: 0 0 } }`; bind to `[data-entering]` with `--duration-medium2` + `--easing-emphasized-decelerate`
- [ ] Exit: reverse; bind to `[data-exiting]` with `--duration-short4` + `--easing-emphasized-accelerate`
- [ ] **Do NOT add `[dir="rtl"]` overrides for the translation** — `translate: 100% 0` using the CSS `translate` property is already direction-aware (moves in inline-end direction)
- [ ] `__header`: `display: flex; align-items: center; gap: 4px; padding: 16px; flex-shrink: 0`
- [ ] `__close-btn`: `color: var(--on-surface-variant)`; ensure 48×48 dp minimum touch target via padding
- [ ] `__title`: `flex: 1; color: var(--on-surface); font-size: var(--title-large-size); line-height: var(--title-large-line-height)`
- [ ] `__header-actions`: `display: flex; gap: 4px; margin-inline-start: auto`
- [ ] `__divider`: `border-block-end: 1px solid var(--outline-variant); flex-shrink: 0`
- [ ] `__content`: `flex: 1; overflow-y: auto; overscroll-behavior: contain; padding: 16px`
- [ ] `__actions`: `flex-shrink: 0; padding: 16px; display: flex; gap: 8px; justify-content: flex-end`
- [ ] `prefers-reduced-motion`: `animation-duration: 1ms` on `[data-entering]`/`[data-exiting]`; override keyframes to opacity-only fade
- [ ] Do NOT use `--kafui-side-sheet-width` — use `--w` as the component-internal var name (inside `.side-sheet { }`)
- [ ] Do NOT use `rgba(var(--md-sys-color-scrim-rgb), 0.32)` — use shared `.scrim` with `color-mix(in srgb, var(--scrim) 32%, transparent)`

### React Component (`@kafui/react`)

- [ ] Create `SideSheet.tsx`:
  - Modal path: wrap `ModalOverlay` + `Modal` + `Dialog` (RAC); inject `.side-sheet .side-sheet--modal`; pass `aria-labelledby` to `Dialog`
  - Standard path: render `<section role="complementary">` with `aria-labelledby`; no RAC wrapper; animate via `.side-sheet--open` modifier + CSS transition on `translate`
- [ ] Create `SideSheetTrigger.tsx` (modal): thin wrapper around RAC `DialogTrigger`; expose as `SideSheet.Trigger`
- [ ] Auto-generate stable id for `__title` via `React.useId()`; wire to `aria-labelledby` on root
- [ ] Render `__close-btn`: `<IconButton>` with `aria-label={closeLabel}` (default `"Close"`); calls `onOpenChange(false)` on press; icon determined by `closeIcon` prop (`'close'` | `'back'`; default `'close'`)
- [ ] `closeIcon` suggestion: auto-default `'back'` when `variant="standard"` and `'close'` when `variant="modal"` — matches M3 UX pattern; override always available
- [ ] Standard variant: `onOpenChange` called only when close button pressed; no Esc or scrim handling (by design)
- [ ] Wire `isDismissable` → RAC `ModalOverlay isDismissable`
- [ ] Document `--w` as the CSS custom property consumers override (in the component file's JSDoc and in Storybook)
- [ ] Export: `SideSheet` (with `.Trigger` on namespace); `SideSheetTrigger` also as standalone named export

### Accessibility

- [ ] Modal: verify focus moves to close button on open; verify return-focus to trigger on close — keyboard + VoiceOver + NVDA
- [ ] Modal: verify `Esc` closes and focus returns to trigger
- [ ] Modal: verify `Tab` cannot leave the sheet
- [ ] Standard: verify `role="complementary"` announced as landmark by VoiceOver and NVDA
- [ ] Standard: verify close button reachable by keyboard and activates `onOpenChange(false)`
- [ ] `aria-labelledby` must reference a unique id per page instance — confirm `useId` produces unique ids for multiple simultaneous sheets
- [ ] axe-core: no violations for both variants, light + dark

### Testing

- [ ] Unit: `.side-sheet--standard` vs `.side-sheet--modal` BEM class per `variant`
- [ ] Unit: close button fires `onOpenChange(false)` in both variants
- [ ] Unit: `isDismissable={false}` — scrim click does not call `onOpenChange`
- [ ] Unit: `closeLabel` sets `aria-label` on close button
- [ ] Unit: `headerActions` slot renders inside `__header`
- [ ] Unit: `closeIcon="back"` renders back-arrow icon
- [ ] A11y: axe audit — both variants, light + dark, LTR + RTL
- [ ] Visual regression: open state, both variants, with/without `headerActions`, with/without `actions` bar, light + dark, LTR + RTL

### Open Questions

- [ ] **`SideSheetLayout` helper:** provide a CSS grid wrapper (`main + sheet` columns) to reduce layout boilerplate for standard variant; consider `<SideSheetLayout>` component that applies `display: grid; grid-template-columns: 1fr auto` (or `minmax(0,1fr) var(--w)`). Would dramatically improve standard variant DX.
- [ ] **Responsive `promoteTo="bottom-sheet"`:** modal side sheet auto-converts to bottom sheet on compact viewports — mirror of bottom sheet's `promoteTo="side-sheet"`. Use `ResizeObserver` (not `window.innerWidth`) for container-aware layouts.
- [ ] **Standard animation:** `translate` (correct M3 motion, needs `position: absolute` or layout wrapper to avoid reflow) vs `width` transition (avoids reflow, visible collapse). Recommend `translate` inside a `SideSheetLayout` grid — grid handles the layout; sheet translates without reflow.
- [ ] **`closeIcon` auto-default:** auto-pick `'back'` for standard and `'close'` for modal unless overridden — matches M3 UX without extra consumer config.
- [ ] **Scrim shared partial:** confirm `_scrim.css` is imported by side-sheet, dialog, and bottom-sheet CSS — single source, no duplication.
