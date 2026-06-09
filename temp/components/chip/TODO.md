# Chip — TODO

## MUI Equivalent

`Chip` from `@mui/material`. Single component covers all four M3 types via `variant` ("filled" / "outlined") and props (`onClick`, `onDelete`, `icon`, `avatar`, `clickable`, `color`). No built-in filter-group or input chip set container; consumers compose those themselves with MUI Box/Stack.

---

## kafUI beats MUI — opportunities

| Dimension | kafUI wins | MUI pain |
|---|---|---|
| **Correct semantics per type** | Four purpose-driven APIs (`AssistChip`, `FilterChipGroup`, `InputChipSet`, `SuggestionChip`) — intent is self-documenting at the call site, impossible to accidentally use wrong ARIA role | Single `Chip` with guesswork props; filter behavior requires manual `onClick` + uncontrolled CSS class; no `role="menuitemradio"` or `aria-pressed` by default |
| **Filter group built-in** | `FilterChipGroup` wraps RAC `ToggleButtonGroup`; keyboard arrow-nav, single/multi-select, `onSelectionChange` — zero boilerplate | Consumer must build their own group container, manage `selectedKeys`, handle keyboard manually — or reach for a third-party lib |
| **Input chip set built-in** | `InputChipSet` wraps RAC `TagGroup`; Delete/Backspace removes, arrow navigation, `onRemove` with key set — ready to use | MUI `Chip` + `onDelete` puts a delete button on the chip, but consumers must write their own chip-set container, keyboard nav, and focus management — every project reinvents this |
| **Zero runtime CSS** | `@layer kafui` BEM classes; state-layer is a `::before` pseudo-element toggled by RAC `data-*` attributes — no JS on hot path | Emotion runtime generates class names on mount; ~8 kB gz overhead even for a single chip; `sx` prop re-executes on every render |
| **CSS-var theming at any scope** | Override `--secondary-container`, `--corner-full`, or component-scoped `--radius`/`--h`/`--pad-inline` from any selector — one-line brand themes, scoped to a subtree | `theme.components.MuiChip.styleOverrides` requires a full JS theme rebuild; impossible to scope to a subtree without a nested `ThemeProvider` |
| **Checkmark animation** | Pure CSS transition on `.chip__leading-icon--checkmark` driven by `.chip--selected`; no JS state for visual | MUI checks a boolean prop and renders/unmounts the icon — causes layout shift; transition requires `TransitionComponent` |
| **Copy/extend friendly** | Single CSS block per variant type; no Emotion internals to fight; fork and rename BEM classes | Overriding MUI chip requires `styleOverrides.root`, `styleOverrides.label`, `styleOverrides.deleteIcon` — undocumented class names that change across minor versions |

---

## Actionable TODO Checklist

### CSS (`@kafui/styles`)

#### Layer setup
- [ ] Create `packages/styles/src/components/_chip.css`
- [ ] Wrap all rules in `@layer kafui { … }` — guarantees collision safety against app styles

#### Base chip
- [ ] Define `.chip` with component-scoped vars: `--radius: var(--corner-full); --h: 32px; --pad-inline: 16px; --icon-size: 18px; --state-color: var(--on-surface-variant)`
- [ ] `border-radius: var(--radius); height: var(--h); padding-inline: var(--pad-inline)` — use logical properties throughout
- [ ] `font-size: var(--label-large-size); font-weight: var(--label-large-weight); line-height: var(--label-large-line-height)`
- [ ] State-layer `::before` pseudo: `position: absolute; inset: 0; border-radius: inherit; background: var(--state-color); opacity: 0; transition: opacity var(--duration-short3) var(--easing-standard)`
- [ ] `[data-hovered]::before { opacity: var(--state-hover) }` — 0.08
- [ ] `[data-focused]::before { opacity: var(--state-focus) }` — 0.10
- [ ] `[data-pressed]::before { opacity: var(--state-pressed) }` — 0.10
- [ ] `[data-disabled], .chip--disabled`: `opacity: 0.38; pointer-events: none; outline-color: var(--on-surface)`
- [ ] Dragged: `.chip--dragged::before { opacity: var(--state-dragged) }` + `box-shadow: var(--elevation-4)`

#### Type modifiers
- [ ] `.chip--assist, .chip--filter, .chip--suggestion`: `background: transparent; outline: 1px solid var(--outline); outline-offset: -1px; color: var(--on-surface-variant)`
- [ ] `.chip--elevated`: `background: var(--surface-container-low); outline: none; box-shadow: var(--elevation-1)`
- [ ] `.chip--elevated[data-hovered]`: `box-shadow: var(--elevation-2)`
- [ ] `.chip--filter.chip--selected`: `--state-color: var(--on-secondary-container); background: var(--secondary-container); outline: none; color: var(--on-secondary-container)`
- [ ] `.chip--input`: `outline: 1px solid var(--outline); --pad-inline: 12px`

#### Icons
- [ ] `.chip__leading-icon, .chip__trailing-icon`: `width: var(--icon-size); height: var(--icon-size); flex-shrink: 0; display: flex; align-items: center`
- [ ] `.chip__leading-icon`: `margin-inline-end: -4px` (optical tighten — leading icon sits at padding edge)
- [ ] `.chip__trailing-icon`: `border-radius: var(--corner-full); margin-inline-start: -4px`

#### Chip set
- [ ] `.chip-set`: `display: flex; flex-wrap: wrap; gap: 8px; align-items: center`
- [ ] `.chip-set--scroll`: `flex-wrap: nowrap; overflow-x: auto; scrollbar-width: none`
- [ ] `.chip-set--scroll::-webkit-scrollbar { display: none }`

#### Motion
- [ ] Filter checkmark: `.chip__leading-icon--checkmark { opacity: 0; transform: scale(0.5); transition: opacity var(--duration-short3) var(--easing-standard), transform var(--duration-short3) var(--easing-standard) }`
- [ ] `.chip--selected .chip__leading-icon--checkmark { opacity: 1; transform: scale(1) }`
- [ ] `@keyframes chip-exit { from { transform: scale(1); opacity: 1 } to { transform: scale(0); opacity: 0 } }`
- [ ] `.chip--removing { animation: chip-exit var(--duration-short2) var(--easing-standard) forwards }`
- [ ] `@media (prefers-reduced-motion: reduce)` — disable all transitions and animations

#### Dark mode / RTL
- [ ] All color values use `--on-surface-variant`, `--secondary-container`, etc. — automatically flip via `light-dark()` in the token layer; no extra rules needed
- [ ] Confirm all directional properties use logical form (`padding-inline`, `margin-inline-end`, `margin-inline-start`)

### React components (`@kafui/react`)

- [ ] `packages/react/src/components/Chip/AssistChip.tsx` — RAC `<Button>`; className `chip chip--assist [chip--elevated]`
- [ ] `packages/react/src/components/Chip/SuggestionChip.tsx` — RAC `<Button>`; className `chip chip--suggestion [chip--elevated]`
- [ ] `packages/react/src/components/Chip/FilterChip.tsx` — RAC `<ToggleButton>`; className `chip chip--filter [chip--elevated] [chip--selected]`; render checkmark icon when `data-selected`
- [ ] `packages/react/src/components/Chip/FilterChipGroup.tsx` — RAC `<ToggleButtonGroup>`; map `selectionMode` prop
- [ ] `packages/react/src/components/Chip/InputChipSet.tsx` — RAC `<TagGroup allowsRemoving>` + `<TagList>` + `<Tag>`; auto-generate remove `aria-label` from item label
- [ ] Wire `Icon` component for leading/trailing icons using sprite system
- [ ] Add `chip--removing` class on remove trigger and remove from DOM after animation (use `onAnimationEnd`)
- [ ] Export all from `packages/react/src/index.ts`
- [ ] Expose compound namespace: `Chip.Assist`, `Chip.Filter`, `Chip.FilterGroup`, `Chip.InputSet`, `Chip.Suggestion`

### Testing
- [ ] Unit: `AssistChip` fires `onPress`; does not fire when `isDisabled`
- [ ] Unit: `FilterChip` toggles `chip--selected`; `aria-pressed` updates correctly
- [ ] Unit: `FilterChipGroup` single/multi selection modes; `onSelectionChange` payload shape
- [ ] Unit: `InputChipSet` keyboard remove (Delete/Backspace); `onRemove` fires with correct key set
- [ ] Unit: `InputChipSet` arrow key navigation between chips
- [ ] Unit: disabled state blocks interaction on all types; `data-disabled` present
- [ ] Unit: `chip--removing` class applied → `onRemove` fires after `animationend`
- [ ] a11y: axe-core scan — all four types, selected + unselected states
- [ ] a11y: `FilterChipGroup` — confirm `aria-pressed` on each chip; confirm group `aria-label`
- [ ] a11y: `InputChipSet` — confirm `role="grid"` on container; `aria-label="Remove {label}"` on remove buttons
- [ ] Visual regression: all four types, selected, disabled, elevated, light + dark, LTR + RTL

### Docs / Storybook
- [ ] Story: all four chip types side-by-side
- [ ] Story: `FilterChipGroup` — single-select vs multi-select
- [ ] Story: `InputChipSet` with add + remove interaction
- [ ] Story: chip-set wrap vs scroll layout
- [ ] Story: elevated variants (assist + suggestion)
- [ ] Story: RTL chip sets
- [ ] Story: custom theme — override `--corner-full` to `--corner-small` for squared chips
