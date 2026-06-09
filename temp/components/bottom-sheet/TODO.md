# Bottom Sheet — TODO

## MUI Equivalent

**MUI `SwipeableDrawer`** (`anchor="bottom"`) + **`Drawer`** (`anchor="bottom" variant="persistent"`).

- `SwipeableDrawer`: swipe gestures, modal behavior, single open/closed toggle. No snap points.
- `Drawer variant="persistent"`: non-modal, stays open. No snap points.
- MUI has no detent/snap-point system for bottom sheets in any variant.

---

## How kafUI Beats MUI

| Dimension | kafUI | MUI SwipeableDrawer |
|---|---|---|
| **Detents** | First-class `peek`/`half`/`expanded` snap points with `detent`/`onDetentChange` — matches M3 spec; controlled + uncontrolled | Single open/closed toggle; no snap points; no corner morph; consumer must build entirely from scratch |
| **Corner morph** | `border-start-start/end-radius` transitions from `--corner-extra-large` → 0 at full expansion; purely CSS, driven by `.bottom-sheet--detent-expanded` | No corner morph; generic `border-radius` |
| **Drag semantics** | `role="slider"` on handle with `aria-valuenow/valuetext` announces detent to AT | No ARIA values on drag handle; typically a decorative div |
| **CSS-only animation** | `@keyframes` on `[data-entering]`/`[data-exiting]`; emphasized easing; zero JS animation cost | JS-driven `Slide` transition; Emotion runtime; `prefers-reduced-motion` requires manual config |
| **Zero reflow drag** | `transform: translateY(var(--translate-y))` via CSS custom property update — GPU-composited layer; zero layout reflow during gesture | CSS transform too, but `keepMounted`/`onOpen` lifecycle adds React render overhead |
| **Responsive promotion** | `promoteTo="side-sheet"` opt-in — correct M3 breakpoint behavior | No promotion; manual consumer wiring |
| **Token fidelity** | `--surface-container-low`, emphasized easing for enter, emphasized-accelerate for exit, `color-mix()` scrim | MUI theme colors; generic easing |
| **Standard variant** | `role="region"` landmark; no focus trap — correct M3 behavior | `Drawer variant="persistent"` uses `role="presentation"` — loses landmark semantics |
| **Safe area** | `padding-block-end: env(safe-area-inset-bottom)` on `__content` — home bar handled correctly | Manual consumer addition |

**Biggest single win:** MUI offers zero detent/snap-point API. kafUI's `detent` + `onDetentChange` system is a category-level feature gap — any team that needs a bottom sheet with snap points currently has to build from scratch or add a separate library (`@gorhom/bottom-sheet` equivalent for web).

---

## Actionable TODO Checklist

### Styles (`@kafui/styles`)

- [ ] Create `bottom-sheet.css`; wrap entirely in `@layer kafui { … }`
- [ ] Import / depend on shared `_scrim.css` for `.scrim` — do NOT redefine scrim here
- [ ] Declare component-internal vars at top of `.bottom-sheet { --translate-y: 0px; }` block
- [ ] Sheet surface: `position: fixed; inset-inline: 0; inset-block-end: 0; background: var(--surface-container-low); box-shadow: var(--elevation-1)`
- [ ] Top corners only: `border-start-start-radius: var(--corner-extra-large); border-start-end-radius: var(--corner-extra-large); border-end-start-radius: 0; border-end-end-radius: 0`
- [ ] `.bottom-sheet--detent-expanded`: `border-start-start-radius: 0; border-start-end-radius: 0`; add `transition` on those two properties with `--duration-short4` + `--easing-standard`
- [ ] `.bottom-sheet--dragging`: `transition: none` — JS controls position during gesture
- [ ] Enter: `@keyframes sheet-enter { from { transform: translateY(100%) } to { transform: translateY(0) } }`; bind to `[data-entering]` with `--duration-long2` + `--easing-emphasized`
- [ ] Exit: `@keyframes sheet-exit` (reverse); bind to `[data-exiting]` with `--duration-medium4` + `--easing-emphasized-accelerate`
- [ ] Drag handle: `width: 32px; height: 4px; border-radius: var(--corner-full); background: var(--on-surface-variant); opacity: 0.4; margin-inline: auto; margin-block: 22px 0; cursor: grab`
- [ ] State layer on drag handle: `::after` pseudo with `background: currentColor` driven by `[data-hovered]` / `[data-pressed]` / `[data-focus-visible]` → opacity `--state-hover` / `--state-pressed` / `--state-focus`
- [ ] `__content`: `overflow-y: auto; overscroll-behavior: contain; padding-block-end: env(safe-area-inset-bottom)`
- [ ] `__header`: `position: sticky; top: 0; padding: 16px; flex-shrink: 0; font-size: var(--title-medium-size)`
- [ ] `@media (prefers-reduced-motion: reduce)`: `transition: none; animation: none` on `.bottom-sheet`
- [ ] Do NOT use `--kafui-bottom-sheet-*` local var names — use short unprefixed names: `--translate-y`, `--h`, etc. (inside `.bottom-sheet { }` block only)
- [ ] Do NOT use `rgba(var(--md-sys-color-scrim-rgb), 0.32)` — use shared `.scrim` class with `color-mix(in srgb, var(--scrim) 32%, transparent)`

### React Component (`@kafui/react`)

- [ ] `BottomSheet.tsx`: root component; dispatches to modal or standard rendering path
- [ ] Modal path: `ModalOverlay` + `Modal` + `Dialog` (RAC); inject `.bottom-sheet .bottom-sheet--modal` + `data-entering`/`data-exiting`
- [ ] Standard path: plain positioned element; `role="region"`; `aria-labelledby` from header title
- [ ] `BottomSheet.Header`: sticky header; auto-generates id via `useId`; wires `aria-labelledby` on root
- [ ] `BottomSheet.Content`: scrollable region; applies `__content` class
- [ ] `BottomSheet.Trigger`: thin wrapper around RAC `DialogTrigger` (modal) or plain state setter (standard)
- [ ] `useBottomSheet` hook:
  - Detent state machine: `hidden ↔ peek ↔ half ↔ expanded` (configurable set via `detents` prop)
  - Controlled + uncontrolled via `useControlledState`
  - Returns `{ detent, setDetent, sheetProps, handleProps }`
  - Handle pointer events: `pointerdown/pointermove/pointerup` on handle (+ optional body for coarse pointer)
  - Velocity-based snap on `pointerup`: compute `vy = deltaY / deltaTime`; threshold ~0.3 px/ms for a "flick"
  - Update `--translate-y` CSS custom property directly on the element ref during drag (`element.style.setProperty`) — no React state update during gesture = no re-renders while dragging
  - On snap: set detent state (triggers React re-render once); CSS transition handles the snap motion
- [ ] Keyboard on drag handle (`role="slider"`): `ArrowUp` → expand one detent; `ArrowDown` → collapse one; `Home` → `expanded`; `End` → `hidden` or `peek`
- [ ] `Escape` (modal): call `onOpenChange(false)`; (standard): collapse to `peek`
- [ ] `dragHandleLabel` prop → `aria-label` on handle element; default `"Sheet size"`
- [ ] `aria-valuenow` = current detent index; `aria-valuemin` = 0; `aria-valuemax` = `detents.length - 1`; `aria-valuetext` = detent name
- [ ] `@media (pointer: coarse)`: enable full-surface drag; `@media (pointer: fine)`: drag handle only
- [ ] `touch-action: none` on drag handle to prevent browser scroll interference
- [ ] `promoteTo="side-sheet"` prop: if `window.innerWidth >= 600` (or container query), render as `<SideSheet>` instead — import SideSheet; document the contract
- [ ] Export: `BottomSheet` (with `.Header`, `.Content`, `.Trigger` on namespace); `useBottomSheet` as public hook

### Accessibility

- [ ] `aria-modal="true"` on modal root; RAC `ModalOverlay` applies `aria-hidden` to app root — verify
- [ ] `role="slider"` + valuenow/valuetext on drag handle — test with VoiceOver and NVDA
- [ ] `Escape` closes modal and returns focus to trigger — test keyboard-only
- [ ] Standard: `role="region"` announced as landmark by VoiceOver — test
- [ ] Scroll + drag conflict: when `__content` is scrolled to top, downward swipe should dismiss; when scrolled mid-way, swipe scrolls (not dismisses). Implement via `scrollTop === 0` check before initiating drag.

### Testing

- [ ] Unit: detent state machine (hidden↔peek↔half↔expanded)
- [ ] Unit: velocity snap — flick up → expand, flick down → collapse/dismiss
- [ ] Unit: `--translate-y` updates during drag without React re-renders (spy on `setProperty`)
- [ ] Unit: keyboard on handle — ArrowUp/Down cycle detents
- [ ] A11y: `aria-modal`, focus trap, Escape, return-focus (modal variant)
- [ ] A11y: `role="slider"` valuenow/valuetext updates as detent changes
- [ ] Visual: enter/exit animation classes; corner morph on expanded
- [ ] Responsive: `promoteTo="side-sheet"` renders `<SideSheet>` above 600 dp
- [ ] iOS: momentum scroll inside `__content` is not blocked; home-bar padding correct

### Open Questions

- [ ] **Scroll-vs-drag conflict at top of content:** use `scrollTop === 0` gate on `pointerdown` inside `__content`; on drag handle always initiate drag regardless of scroll position
- [ ] **`promoteTo="side-sheet"` responsiveness:** `ResizeObserver` on a container element preferred over `window.innerWidth` for container-query-aware layouts — decide and document
- [ ] **`useBottomSheet` public API:** expose as `@kafui/react/use-bottom-sheet` for headless usage (custom trigger UIs, native-like sheets)
- [ ] **`__header` close button:** M3 bottom sheet header sometimes shows a close icon (×). Should `BottomSheet.Header` accept a `showCloseButton` prop analogous to SideSheet's `closeLabel`?
