# Snackbar — TODO

## MUI Equivalents
`@mui/material/Snackbar` (controlled `open` per instance) + `@mui/material/Alert` (for severity colors) + `notistack` (third-party queue library MUI recommends). kafUI beats all three by rolling a first-class imperative queue into the component itself, with zero severity-color theater and correct M3 tokens.

---

## Beat-MUI Opportunities

| # | kafUI wins by… | vs MUI |
|---|----------------|--------|
| 1 | **First-class imperative queue via `useSnackbar()` + `add()`** — no `open` boolean, no manual queue state, no notistack dependency | MUI requires either `open` prop per instance (manual queue) or installing notistack (different API, different tokens, 15 kB) |
| 2 | **`add()` returns a key; `close(key)` for targeted dismissal** — predictable, testable, TypeScript-native | notistack `enqueueSnackbar` returns a key but the API is loosely typed and not integrated into design tokens |
| 3 | **`priority="assertive"` maps to ARIA role swap** — error handling is an a11y concern, not a color concern | MUI/Alert conflate visual severity (error/warning/success) with ARIA urgency; wrong in M3, wrong in general |
| 4 | **Auto `"two-line"` variant detection from message length** — zero config for the common case | MUI has no automatic layout switch; developer manually checks + composes |
| 5 | **`duration=0` + no `showClose` → dev warning** — enforces accessible persistent snackbars | MUI allows persistent snackbar with no dismiss mechanism silently; a11y trap |
| 6 | **Explicit `queue` prop on `<SnackbarRegion>`** — SSR-safe, testable, multiple independent regions | MUI/notistack use implicit globals that break SSR and make isolated testing hard |

---

## Actionable TODO Checklist

### Architecture
- [ ] `packages/react/src/snackbar/types.ts` — export `SnackbarOptions` interface (canonical source of truth)
- [ ] `packages/react/src/snackbar/snackbarQueue.ts` — export `createSnackbarQueue()` factory (returns `ToastQueue<SnackbarOptions>` with `maxVisibleToasts: 1`)
- [ ] `packages/react/src/snackbar/useSnackbar.ts` — context-aware hook; `add()` returns key; `close(key)`; `closeAll()`
- [ ] `packages/react/src/snackbar/SnackbarContext.tsx` — `SnackbarProvider` wraps `SnackbarRegion` + context for `useSnackbar()`
- [ ] `packages/react/src/snackbar/SnackbarRegion.tsx` — wraps RAC `ToastRegion`; accepts explicit `queue` prop
- [ ] `packages/react/src/snackbar/SnackbarItem.tsx` — wraps RAC `Toast`; renders `.snackbar` BEM structure
- [ ] `packages/styles/src/snackbar/snackbar.css` — all styles in `@layer kafui { … }`

### Queue logic
- [ ] `ToastQueue` `maxVisibleToasts: 1` — M3 spec; not negotiable in v1
- [ ] Map `priority="assertive"` → `{ priority: 1 }` in `ToastQueue.add()` options
- [ ] Map `duration` → `{ timeout: duration }` (RAC `timeout` param)
- [ ] Pause timer on hover: listen `data-hovered` from RAC `Toast` render prop OR use `onMouseEnter`/`onMouseLeave` on `.snackbar` and call `toast.pauseTimer()` / `toast.resumeTimer()` (RAC API)
- [ ] Pause timer on focus-within: `onFocus` on `.snackbar__actions` → pause; `onBlur` → resume
- [ ] Ensure next item shows only *after exit animation finishes*: use RAC `Toast` `state.exiting` + `onAnimationEnd` to call `state.remove()`
- [ ] Dev guard: `duration=0` && `!showClose` → `console.warn("Persistent snackbar requires showClose=true")`
- [ ] Auto-detect `variant`: if not specified, `message.length > 40` → `"two-line"`, else `"single-line"`

### `SnackbarItem` render
- [ ] `priority === "assertive"` → swap root element's role/aria attributes to `role="alertdialog"` + `aria-live="assertive"` (coordinate with RAC Toast assertive channel)
- [ ] Render `.snackbar__action` only when `actionLabel` + `onAction` present; `onAction` calls provided callback then `toast.close()`
- [ ] Render `.snackbar__close` only when `showClose=true`; `aria-label` from i18n key `snackbar.dismiss` (default `"Dismiss"`)
- [ ] Apply `.snackbar--entering` on mount → CSS handles enter animation; remove after animation ends
- [ ] Apply `.snackbar--exiting` when RAC `state === "exiting"`; call `state.remove()` on `animationend`

### Styles — inside `@layer kafui`
```css
@layer kafui {
  .snackbar-region {
    position: fixed;
    inset-inline: 0;
    inset-block-end: 16px;
    display: flex;
    flex-direction: column;
    align-items: center;
    pointer-events: none;
    z-index: 1400; /* above dialogs: adjust per app z-index scale */
  }

  .snackbar {
    --_bg:        var(--inverse-surface);
    --_fg:        var(--inverse-on-surface);
    --_action-fg: var(--inverse-primary);
    --_r:         var(--corner-extra-small);
    --_min-w:     288px;
    --_max-w:     568px;

    display: flex;
    align-items: center;
    gap: 8px;
    padding-block: 14px;
    padding-inline: 16px;
    background: var(--_bg);
    color: var(--_fg);
    border-radius: var(--_r);
    box-shadow: var(--elevation-3);
    min-width: var(--_min-w);
    max-width: var(--_max-w);
    width: fit-content;
    pointer-events: auto;

    animation: snackbar-enter var(--duration-short4) var(--easing-emphasized-decelerate) forwards;
  }

  .snackbar--exiting {
    animation: snackbar-exit var(--duration-short4) var(--easing-emphasized-accelerate) forwards;
  }

  .snackbar--two-line {
    flex-direction: column;
    align-items: flex-start;
  }

  .snackbar__text {
    flex: 1;
    font: var(--body-medium-font);
    color: var(--_fg);
  }

  .snackbar__actions {
    display: flex;
    align-items: center;
    gap: 8px;
    flex-shrink: 0;
  }

  .snackbar--two-line .snackbar__actions {
    align-self: flex-end;
  }

  .snackbar__action {
    color: var(--_action-fg);
    font: var(--label-large-font);
    background: none;
    border: none;
    cursor: pointer;
    padding: 0 8px;
    position: relative; /* state layer */
  }

  .snackbar__action .state-layer::after {
    background: var(--_action-fg);
  }

  /* Responsive: full-width on narrow screens */
  @media (max-width: 599px) {
    .snackbar { min-width: 0; width: calc(100vw - 32px); }
  }

  @keyframes snackbar-enter {
    from { opacity: 0; transform: translateY(100%); }
    to   { opacity: 1; transform: translateY(0); }
  }

  @keyframes snackbar-exit {
    from { opacity: 1; transform: translateY(0); }
    to   { opacity: 0; transform: translateY(100%); }
  }

  @media (prefers-reduced-motion: reduce) {
    .snackbar, .snackbar--exiting { animation: none; }
  }
}
```
- [ ] Implement above
- [ ] `.state-layer` pseudo-element on `.snackbar__action` and `.snackbar__close` using shared mechanism from `_TOKENS.md`
- [ ] Verify `z-index` value is documented as a token or layering convention (not hardcoded 1400)

### Accessibility
- [ ] `SnackbarRegion` root: `role="status"`, `aria-live="polite"`, `aria-relevant="additions"`, `aria-atomic="true"`
- [ ] `priority="assertive"` item: swap to `role="alert"`, `aria-live="assertive"` (coordinate with RAC)
- [ ] Close button `aria-label` via i18n — export `snackbar.dismiss` translation key
- [ ] Action button label is user-supplied; document that it must be self-describing ("Undo" not "OK")
- [ ] Focus return after dismissal: RAC `Toast` restores focus — verify in integration test

### Tokens
- [ ] `--inverse-surface` → `light-dark(var(--ref-neutral-20), var(--ref-neutral-90))`
- [ ] `--inverse-on-surface` → `light-dark(var(--ref-neutral-90), var(--ref-neutral-20))`
- [ ] `--inverse-primary` → `light-dark(var(--ref-primary-80), var(--ref-primary-40))`
- [ ] `--corner-extra-small` → `4px`
- [ ] `--elevation-3` → appropriate box-shadow from elevation scale
- [ ] `--duration-short4`, `--easing-emphasized-decelerate`, `--easing-emphasized-accelerate`

### Tests
- [ ] Unit: `add({ message: "Saved" })` → message rendered in region
- [ ] Unit: `duration=0` + no `showClose` → dev console.warn fires
- [ ] Unit: two consecutive `add()` → second queued; shown after first exits
- [ ] Unit: `actionLabel` + `onAction` → action button present; fires callback + dismisses
- [ ] Unit: `add()` returns key; `close(key)` removes correct item
- [ ] Unit: hover over snackbar → timer paused (mock timer)
- [ ] A11y: region has `aria-live="polite"` and `role="status"`
- [ ] A11y: `priority="assertive"` → region/item has `aria-live="assertive"`
- [ ] Integration: focus returns to trigger element after dismiss

### Decisions resolved (documented here for record)
- [x] `SnackbarRegion` takes explicit `queue` prop — not implicit singleton (SSR-safe)
- [x] `SnackbarProvider` is optional convenience wrapper for simpler apps
- [x] No `severity` prop — `priority` is ARIA-only, not visual
- [x] No `anchorOrigin` — M3 bottom-center only; RTL via logical CSS
- [x] Auto two-line detection: `message.length > 40` threshold (reviewable)

### Open question
- [ ] z-index layering: should `--z-snackbar` be a library token (e.g. `--z-snackbar: 1400`) or left to consumers? Recommend: export as a CSS var default that consumers can override at `:root`.

### Storybook
- [ ] Basic message (auto-dismiss)
- [ ] With action button (Undo)
- [ ] With close button
- [ ] Two-line variant
- [ ] `priority="assertive"` (error-level)
- [ ] Queue demo (add 3 messages rapidly; observe queueing)
- [ ] Persistent (`duration=0`, `showClose=true`)
- [ ] RTL
- [ ] Reduced motion
- [ ] Dark mode
