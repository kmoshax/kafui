# Dialog — TODO

## MUI Equivalent

`Dialog` (with `DialogTitle`, `DialogContent`, `DialogContentText`, `DialogActions`) + `Backdrop`. Full-screen: `Dialog fullScreen` prop. Typical usage requires 5+ imports and `open`/`onClose` wired at the top level.

---

## How kafUI Beats MUI

| Dimension | kafUI | MUI Dialog |
|---|---|---|
| **Boilerplate** | `<Dialog.Trigger>` + `<Dialog headline="…">` — 2 components, zero `useState` needed for uncontrolled | Always controlled: `useState(false)` + `open={open}` + `onClose={() => setOpen(false)}` — 3 extra lines minimum |
| **Token fidelity** | Exact M3: `--surface-container-high`, `--corner-extra-large`, elevation 3, precise enter/exit easing, 32% scrim via `color-mix()` | Loosely M3-inspired. Container is `background.paper`. Radius is configurable but not M3-defaulted. No hero icon slot. No exact scrim opacity. |
| **Variants** | `basic` vs `full-screen` as first-class `variant` prop with distinct token sets | Single `<Dialog fullScreen>` boolean; no token differentiation; `fullWidth`/`maxWidth` props add complexity that M3 replaces with a single sizing rule |
| **CSS-only animation** | `@keyframes` bound to RAC `[data-entering]`/`[data-exiting]`; two distinct easing curves; zero JS animation overhead | JS-driven `Fade`/`Grow` via `TransitionComponent`; Emotion runtime cost per dialog |
| **Focus management** | RAC `Modal + Dialog` handles trap, restore, `aria-modal`, and nested portal edge cases correctly | MUI `Unstable_TrapFocus` has documented issues with nested portals and shadow DOM |
| **No keepMounted footgun** | DOM is unmounted on close by default (RAC); `[data-exiting]` keeps it alive during exit animation only | `keepMounted` prop risks hidden dialog DOM accumulating in the tree |
| **Dark mode** | Single `--surface-container-high` CSS var; `light-dark()` in token layer flips automatically | Separate `ThemeProvider` palette override required for dark mode |
| **RTL** | CSS logical properties throughout; zero JS | MUI RTL requires `jss-rtl` / `stylis-plugin-rtl` transform |
| **Compound DX** | `Dialog.Trigger` alias co-located on export; trigger and dialog are siblings in JSX — context is clear | `open`/`onClose` on Dialog root; trigger is a separate unrelated element; wiring is implicit |

**Biggest wins:**

1. **Uncontrolled trigger pattern** — kafUI's `Dialog.Trigger` + `Dialog` eliminates the `useState` boilerplate that every MUI dialog forces. This is the same win shadcn/ui and Radix deliver vs MUI.
2. **CSS-only transitions** — MUI's `TransitionComponent` prop is a footgun (consumers replace the entire transition instead of just tweaking duration). kafUI exposes CSS vars for duration/easing; the animation itself is untouchable from JS.

---

## Actionable TODO Checklist

### Styles (`@kafui/styles`)

- [ ] Wrap entire stylesheet in `@layer kafui { … }` — no `kafui-` prefix on any class name
- [ ] `.scrim` (shared class): `position: fixed; inset: 0; background: color-mix(in srgb, var(--scrim) 32%, transparent); z-index: var(--z-modal-scrim, 1200)` — define once in a shared `_scrim.css` partial, import in dialog + bottom-sheet + side-sheet
- [ ] `.dialog` base: declare internal vars `--max-w: 560px; --min-w: 280px; --pad: 24px; --radius: var(--corner-extra-large)` then apply them; `background: var(--surface-container-high); border-radius: var(--radius); box-shadow: var(--elevation-3)`
- [ ] `.dialog--full-screen`: `--radius: var(--corner-none); max-width: 100dvw; width: 100dvw; height: 100dvh; box-shadow: var(--elevation-0); padding: 0`
- [ ] `.dialog--has-icon`: increase `padding-block-start` on `__headline` region to accommodate icon
- [ ] `__icon`, `__headline`, `__supporting-text`, `__divider`, `__actions` element styles per M3 spacing spec
- [ ] `__header` and `__content` for full-screen variant
- [ ] `@keyframes dialog-enter` (scale 0.8→1 + opacity 0→1); `@keyframes dialog-exit` (reverse); bind via `[data-entering]`/`[data-exiting]`
- [ ] `prefers-reduced-motion`: remove `transform` from keyframes; halve duration via `animation-duration: calc(var(--duration-short4) / 2)` override
- [ ] `__divider` hidden by default; `.dialog--scrolled .dialog__divider { visibility: visible }`
- [ ] `__supporting-text`: `overflow-y: auto; overscroll-behavior: contain`
- [ ] Do NOT use `rgba(var(--scrim-rgb), 0.32)` — use `color-mix(in srgb, var(--scrim) 32%, transparent)` which works with any color type and requires no `-rgb` variant

### React Component (`@kafui/react`)

- [ ] `Dialog.tsx`: wrap RAC `<Dialog>` with BEM class injection; receive `headline`, `icon`, `children`, `actions`, `isDismissable`, `variant` props
- [ ] `DialogTrigger.tsx`: wrap RAC `<DialogTrigger>` + `<ModalOverlay>` + `<Modal>`; expose as `Dialog.Trigger`
- [ ] Auto-generate stable ids for headline and supporting-text via `React.useId()`; pass to `aria-labelledby` / `aria-describedby`
- [ ] **Omit `aria-describedby` entirely** when no `children` (supporting text) is passed — don't point to an empty id
- [ ] Scroll listener on `__supporting-text` → toggle `dialog--scrolled` modifier (use `IntersectionObserver` on a sentinel element at the top of content for performance)
- [ ] Full-screen: render `__header` with close `<IconButton>` (leading, `aria-label={closeLabel}`) + headline + confirm `<Button>` (trailing); `closeLabel` prop defaults to `"Close"`
- [ ] Wire `isDismissable` → RAC `ModalOverlay isDismissable` prop
- [ ] Export `Dialog` (with `.Trigger` and `.Actions` on the namespace) as named exports; `DialogTrigger` also as standalone named export for consumers who prefer it

### Accessibility

- [ ] Verify focus restore to trigger on close — test with keyboard-only + VoiceOver + NVDA
- [ ] Test Esc close in nested dialog scenario — should close only innermost (RAC handles this; verify)
- [ ] Ensure scroll lock does not break iOS Safari momentum scrolling inside dialog (test on real device; `overscroll-behavior: contain` on `__supporting-text` is critical)
- [ ] Confirm `aria-describedby` is absent when no supporting text prop passed
- [ ] axe-core: no violations for basic + full-screen variants, light + dark, LTR + RTL

### Testing

- [ ] Unit: renders `.dialog--basic` vs `.dialog--full-screen` per `variant`
- [ ] Unit: `isDismissable={false}` — scrim click does not call `onOpenChange`
- [ ] Unit: focus trap — `Tab` cannot leave dialog; `Shift+Tab` from first element wraps
- [ ] Unit: `Esc` fires `onOpenChange(false)`
- [ ] Unit: scroll listener adds `.dialog--scrolled` correctly
- [ ] Unit: `aria-labelledby` id matches headline element id
- [ ] Unit: `aria-describedby` absent when no supporting text
- [ ] A11y: axe-core audit, both variants
- [ ] Visual regression: basic + full-screen, light + dark, LTR + RTL, has-icon vs no-icon

### Open Questions

- [ ] **`actions` prop vs `Dialog.Actions` compound child:** compound child is more DX-forward (can pass arbitrary layout); prop is simpler for the basic 2-button case. Recommendation: ship both — `actions` prop as shorthand, `Dialog.Actions` as first-class compound for advanced use.
- [ ] **Full-screen breakpoint:** auto-switch from basic → full-screen below `--breakpoint-compact`? Make opt-in via `responsive` prop; never automatic (breaks predictability).
- [ ] **Fully-controlled without trigger:** add `<DialogPortal>` that renders `ModalOverlay` + `Modal` + `Dialog` directly, without `DialogTrigger`, for programmatic open/close from outside React tree.
- [ ] **`keepMounted` equivalent:** RAC keeps DOM during exit via `[data-exiting]` — confirm no additional work needed; document this behavior explicitly.
