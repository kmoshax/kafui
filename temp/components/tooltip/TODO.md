# Tooltip — TODO

## MUI Equivalent

`@mui/material/Tooltip` — plain tooltips map directly. MUI has **no rich tooltip variant** (closest workaround: custom `Popover` or community packages).

---

## How kafUI Beats MUI

| Dimension | kafUI | MUI Tooltip |
|-----------|-------|-------------|
| **Rich variant** | First-class `variant="rich"` with `title`, `body`, `actions` — zero boilerplate | Non-existent; consumer must compose a `Popover` + manual ARIA wiring |
| **Correct ARIA** | Automatically switches `role="tooltip"` ↔ `role="dialog"` based on `actions` presence; `aria-modal="false"` on interactive variant | Always `role="tooltip"` — semantically wrong when content is interactive |
| **Backing primitive** | Unified `<Tooltip>` API; RAC `TooltipTrigger` or `DialogTrigger + Popover` chosen internally | Single `Popper.js`-backed component — no semantic distinction |
| **CSS-first theming** | Two distinct token sets (plain: `--inverse-surface`; rich: `--surface`, `--outline-variant`) as CSS vars; one CSS file override | Single `.MuiTooltip-tooltip` class; dark background regardless of theme unless sx/styled override |
| **Performance** | RAC lazy-mounts in portal; CSS opacity/visibility transitions; zero JS animation cost | Popper.js + Emotion runtime; non-trivial bundle delta per tooltip |
| **RTL** | Logical placement (`start`/`end`) via RAC Popover; no JS direction logic | Requires `jss-rtl` transform or manual `placement` swap |
| **Motion** | `@media (prefers-reduced-motion)` built into CSS layer; no config needed | Requires manual `TransitionComponent` or theme override |
| **Delay control** | Per-variant sensible defaults (plain: 300 ms, rich: 500 ms); single `delay` prop | `enterDelay` + `leaveDelay` (two props); no variant-aware defaults |

**Biggest single win:** MUI users composing a "rich tooltip" write hundreds of lines of `Popover` + ARIA boilerplate. kafUI ships it in one prop.

---

## Actionable TODO Checklist

### Core implementation

- [ ] Create `packages/react/src/tooltip/Tooltip.tsx` — unified component; dispatches to `PlainTooltip` or `RichTooltip` based on `variant` and `actions` presence
- [ ] Create `packages/react/src/tooltip/PlainTooltip.tsx` — wraps RAC `TooltipTrigger` + `Tooltip`; emits `.tooltip .tooltip--plain`
- [ ] Create `packages/react/src/tooltip/RichTooltip.tsx`:
  - No `actions`: wraps RAC `TooltipTrigger` + `Tooltip` with extra content; `role="tooltip"`
  - With `actions`: wraps RAC `DialogTrigger` + `Popover` + `Dialog`; `role="dialog"` + `aria-modal="false"`
- [ ] Create `packages/styles/src/tooltip/tooltip.css` inside `@layer kafui { … }`

### Positioning

- [ ] Use React Aria's built-in `Popover` placement engine (no extra dependency); evaluate flip/collision behavior in first pass
- [ ] Add `@floating-ui/react` only if RAC Popover collision handling proves insufficient (document decision)
- [ ] Map `placement="start"` → RAC `"left"` in LTR / `"right"` in RTL via React Aria's `placement` prop (RAC handles logical→physical mapping internally)
- [ ] Verify tooltip dismisses on scroll (RAC `TooltipTrigger` does this by default; confirm for `DialogTrigger + Popover` path)

### Styles

- [ ] Wrap all rules in `@layer kafui { … }` — no `kafui-` prefix on class names
- [ ] Declare component-internal vars at `.tooltip { --max-w: 200px; --pad-block: 4px; --pad-inline: 8px; … }`
- [ ] Plain: `background: var(--inverse-surface)`, `color: var(--inverse-on-surface)`, `border-radius: var(--corner-extra-small)`, `box-shadow: var(--elevation-2)`
- [ ] Plain: `visibility: hidden; opacity: 0` default; `[data-entering]`/`[data-open]` → visible + opacity 1; transition on `opacity` + delayed `visibility` so `aria-describedby` resolves even when hidden
- [ ] Plain: `max-width: 200px; white-space: nowrap` (single line per M3); `overflow: hidden; text-overflow: ellipsis` fallback
- [ ] Rich: override `--max-w: 320px`; `background: var(--surface)`; `border: 1px solid var(--outline-variant)` (M3 rich tooltip has a subtle border); `border-radius: var(--corner-medium)`
- [ ] Rich `.tooltip__title`: `color: var(--on-surface)`; `font-size: var(--title-small-size)`
- [ ] Rich `.tooltip__body`: `color: var(--on-surface-variant)`; `font-size: var(--body-medium-size)`
- [ ] Rich `.tooltip__actions`: `display: flex; gap: 8px; justify-content: flex-end; margin-block-start: 8px`
- [ ] `@media (prefers-reduced-motion: reduce)` block: `transition: none` on `.tooltip`

### Accessibility

- [ ] Plain: confirm RAC `TooltipTrigger` auto-wires `aria-describedby` — verify in integration test
- [ ] Plain: ensure tooltip stays in DOM (via portal) with `visibility: hidden` when closed — NOT `display: none` — so `aria-describedby` always resolves
- [ ] Rich non-interactive: same `role="tooltip"` + `aria-describedby` flow; test with VoiceOver
- [ ] Rich interactive: `role="dialog"` + `aria-label={label}` + `aria-modal="false"` on the Popover content; `aria-controls` + `aria-expanded` on trigger
- [ ] Rich interactive: `Escape` closes and returns focus to trigger — RAC `DialogTrigger` handles this; verify
- [ ] Rich interactive: focus enters tooltip on `Tab` from trigger; `Tab` past last action moves focus to next document element (NOT a focus trap)
- [ ] Test: icon-button trigger with tooltip — `aria-label` on icon button is the accessible name; tooltip is the accessible description

### Tests

- [ ] Unit: plain tooltip shows on trigger focus; hides on blur
- [ ] Unit: plain tooltip shows on hover after 300 ms delay
- [ ] Unit: rich tooltip shows after 500 ms hover delay
- [ ] Unit: `Escape` closes rich interactive tooltip; focus returns to trigger
- [ ] Unit: rich tooltip with `actions` emits `role="dialog"`, not `role="tooltip"`
- [ ] Unit: rich tooltip without `actions` emits `role="tooltip"`
- [ ] A11y (axe): plain tooltip on icon button — no violations
- [ ] A11y (axe): rich interactive tooltip — no violations (esp. `aria-modal="false"`)
- [ ] A11y: `aria-describedby` on trigger references tooltip id in both plain and rich non-interactive

### Stories

- [ ] Plain tooltip on icon button
- [ ] Plain tooltip on text trigger; custom `placement`
- [ ] Rich tooltip — title + body only (no actions)
- [ ] Rich tooltip — title + body + actions
- [ ] Delay customization (`delay` prop)
- [ ] RTL placement
- [ ] Dark mode (automatic via `color-scheme`)
- [ ] Reduced motion

### Open decisions

- [ ] **Positioning library:** Prefer RAC built-in Popover; document if `@floating-ui/react` is added
- [ ] **Rich no-actions backing:** `TooltipTrigger` (simpler) vs `DialogTrigger` (consistent with actions path) — recommend splitting on `actions` presence only
- [ ] **`label` as title on rich:** current API uses `label` for both the plain label and the rich title. Consider `title` prop alias for rich variant for clarity, while keeping `label` for backward compat
