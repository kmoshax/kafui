# List — TODO

## MUI Equivalent

`List`, `ListItem`, `ListItemButton`, `ListItemIcon`, `ListItemAvatar`, `ListItemText`, `ListItemSecondaryAction`, `ListSubheader` — seven components to compose one list item.

---

## kafUI beats MUI — opportunities

| Dimension | kafUI wins | MUI pain |
|---|---|---|
| **Flat API — 2 components instead of 7** | `<List>` + `<List.Item leading={…} trailing={…}>` covers every combination — icon, avatar, image, trailing checkbox, trailing switch, secondary text, overline | MUI requires stacking `ListItem`, `ListItemButton`, `ListItemIcon`, `ListItemAvatar`, `ListItemText`, `ListItemSecondaryAction` — ~7 lines of JSX for a single two-line item with icon |
| **Correct grid semantics for complex rows** | `GridList` + `GridListItem` gives `role="grid"` + `role="row"` — trailing interactive elements (checkbox, switch) are independently focusable `gridcell`s, keyboard-accessible without hacks | MUI `ListItemButton` is `<div role="button">` inside `<li>` — nested interactive element; `ListItemSecondaryAction` positioned absolutely; Tab to trailing control is an ad-hoc consumer concern |
| **Arrow key nav built-in** | RAC `GridList` provides Up/Down arrow navigation across rows, Space-to-select, Enter-to-action automatically | MUI has no built-in arrow key navigation in lists — each project reinvents or skips it |
| **`divided` prop** | Single prop auto-injects `<Divider variant="inset">` between every item — correct M3 inset alignment with zero boilerplate | Consumer must manually place `<Divider>` between every `<ListItem>` pair — repetitive and error-prone |
| **Virtualization seam** | `List` API accepts `items` array + render function (like RAC `GridList`) — TanStack Virtual can be dropped in later without breaking call sites | MUI `List` renders children directly; adding windowing requires replacing the wrapper component and re-mapping children |
| **Zero runtime CSS** | `@layer kafui` BEM classes; state-layer DOM node toggled by `data-*`; no JS on hot path | MUI `sx`, Emotion class generation, `ListItem` internal `styled(…)` chain — every render touches Emotion |
| **Single token flip for dark** | `--on-surface`, `--secondary-container`, etc. use `light-dark()` — automatic dark mode with zero duplication | MUI dark mode needs `createTheme({ palette: { mode: 'dark' } })` and a separate `ThemeProvider` |

---

## Actionable TODO Checklist

### CSS (`@kafui/styles`)

#### Layer setup
- [ ] Create `packages/styles/src/components/_list.css`
- [ ] Wrap all rules in `@layer kafui { … }`
- [ ] Define `.list` component-scoped vars: `--item-h: 56px; --pad-inline: 16px; --leading-icon-size: 24px; --state-color: var(--on-surface)`

#### List container
- [ ] `.list`: `list-style: none; margin: 0; padding: 0; display: flex; flex-direction: column`
- [ ] `.list__subheader`: `font-size: var(--label-large-size); font-weight: var(--label-large-weight); color: var(--on-surface-variant); padding-inline: var(--pad-inline); padding-block: 8px; position: sticky; top: 0; background: inherit; pointer-events: none; user-select: none`

#### List item base
- [ ] `.list-item`: `position: relative; display: flex; align-items: center; gap: 16px; min-height: var(--item-h); padding-inline: var(--pad-inline); overflow: hidden; cursor: default`
- [ ] `.list-item--interactive`: `cursor: pointer`

#### State layer (DOM child — required because row contains sub-interactive elements)
- [ ] `.list-item__state-layer`: `position: absolute; inset: 0; pointer-events: none; background: var(--state-color); opacity: 0; transition: opacity var(--duration-short3) var(--easing-standard); z-index: 0`
- [ ] `.list-item[data-hovered] .list-item__state-layer { opacity: var(--state-hover) }` — 0.08
- [ ] `.list-item[data-focused] .list-item__state-layer { opacity: var(--state-focus) }` — 0.10
- [ ] `.list-item[data-pressed] .list-item__state-layer { opacity: var(--state-pressed) }` — 0.10
- [ ] Note: trailing interactive children need `position: relative; z-index: 1` to remain clickable above the state layer

#### Lines variants
- [ ] `.list-item--lines-2`: `min-height: 72px`; supporting text single-line
- [ ] `.list-item--lines-3`: `min-height: 88px; align-items: flex-start; padding-block: 12px`

#### Selected / disabled
- [ ] `[data-selected]`: `--state-color: var(--on-secondary-container); background: var(--secondary-container)`
- [ ] `[data-selected] .list-item__headline, [data-selected] .list-item__leading { color: var(--on-secondary-container) }`
- [ ] `[data-disabled]`: `opacity: 0.38; pointer-events: none; cursor: default`
- [ ] `[data-disabled] .list-item__state-layer { display: none }` — no state layer on disabled

#### Leading slot
- [ ] `.list-item__leading`: `flex-shrink: 0; width: 40px; display: flex; align-items: center; justify-content: center; color: var(--on-surface-variant); position: relative; z-index: 1`
- [ ] `.list-item--leading-image .list-item__leading { width: 56px; height: 56px; overflow: hidden; border-radius: var(--corner-extra-small) }`
- [ ] `.list-item--leading-video .list-item__leading { width: 100px; height: 56px; overflow: hidden }`

#### Content block
- [ ] `.list-item__content`: `flex: 1; min-width: 0; display: flex; flex-direction: column; justify-content: center; position: relative; z-index: 1`
- [ ] `.list-item__overline`: `font-size: var(--label-small-size); font-weight: var(--label-small-weight); color: var(--on-surface-variant)`
- [ ] `.list-item__headline`: `font-size: var(--body-large-size); font-weight: var(--body-large-weight); line-height: var(--body-large-line-height); color: var(--on-surface); white-space: nowrap; overflow: hidden; text-overflow: ellipsis`
- [ ] `.list-item__supporting`: `font-size: var(--body-medium-size); color: var(--on-surface-variant)`
- [ ] `.list-item--lines-3 .list-item__supporting`: `-webkit-line-clamp: 2; display: -webkit-box; -webkit-box-orient: vertical; overflow: hidden; white-space: normal`

#### Trailing slot
- [ ] `.list-item__trailing`: `flex-shrink: 0; color: var(--on-surface-variant); font-size: var(--label-small-size); position: relative; z-index: 1`

#### Dragged state
- [ ] `.list-item--dragged .list-item__state-layer { opacity: var(--state-dragged) }` — 0.16
- [ ] `.list-item--dragged { box-shadow: var(--elevation-1) }`

#### Reduced motion
- [ ] `@media (prefers-reduced-motion: reduce) { .list-item__state-layer { transition: none } }`

### React Component (`@kafui/react`)

- [ ] `List.tsx`: renders `<GridList>` (RAC) when `selectionMode !== 'none'` or `onAction` is set; falls back to plain `<ul>` otherwise
  - [ ] Require `aria-label` or `aria-labelledby` when using `GridList`; emit `console.error` in dev if missing
  - [ ] `divided` prop: wrap items and inject `<Divider variant="inset">` between each pair
  - [ ] `subheader` prop: render `.list__subheader` before items
- [ ] `ListItem.tsx` (`List.Item`): renders `<GridListItem>` (RAC) for interactive; `<li>` for non-interactive
  - [ ] Auto-detect `leading` element type (Icon, Avatar, img) to set BEM modifier — use `displayName` convention or an explicit `leadingType` prop for image/video disambiguation
  - [ ] Render `.list-item__state-layer` as first child (DOM order: state layer → leading → content → trailing)
  - [ ] When `trailing` contains a `<Checkbox>`, `<Switch>`, or `<Radio>`, wrap it in a `<GridListCell>` to make it independently focusable
  - [ ] Apply `list-item--interactive` modifier when `onPress` or `selectionMode !== 'none'`
- [ ] Expose `List.Item` as both compound (`List.Item`) and named export (`ListItem`)
- [ ] Export `List`, `ListItem` from `packages/react/src/index.ts`

### Accessibility
- [ ] Test arrow key navigation (Up/Down) in GridList — confirm focus moves correctly
- [ ] Test Tab to trailing checkbox independently from row activation — confirm separate focus
- [ ] Test `selectionMode="multiple"` with checkboxes — confirm `aria-selected` on row and `aria-checked` on checkbox don't conflict (different roles)
- [ ] Test non-interactive `<ul>` — confirm VoiceOver/NVDA announces "list, N items"
- [ ] Test `selectionMode="none"` with `onAction` — GridList still renders; confirm `Enter` fires `onAction`
- [ ] Test disabled item — not focusable via keyboard; announced as unavailable
- [ ] axe-core: `role="grid"` requires `aria-label` or `aria-labelledby` — confirm warning + no axe violation

### Testing
- [ ] Unit: correct BEM classes for all `lines` values (1, 2, 3)
- [ ] Unit: `leading` type auto-detection → correct BEM modifier
- [ ] Unit: `divided={true}` renders `<Divider>` between items (n-1 dividers for n items)
- [ ] Unit: `selectionMode="none"` + `onAction` → `<GridList>` (interactive path)
- [ ] Unit: `selectionMode="none"` and no `onAction` → `<ul>` (non-interactive path)
- [ ] Unit: selected item has `data-selected`; `background: --secondary-container`
- [ ] Unit: disabled item has `data-disabled`; `onPress` not fired; not reachable via keyboard
- [ ] a11y: axe-core on interactive and non-interactive variants
- [ ] Visual regression: 1/2/3-line, all leading types, trailing types (icon, text, checkbox, switch), selected, disabled, divided, light + dark, LTR + RTL

### Open Questions
- [ ] **Virtualization:** Plan the API seam now — `List` should accept `items` array + render function (matching RAC `GridList` `items` + renderItem) so TanStack Virtual can be swapped in without a breaking change. Add a test for this integration path.
- [ ] **`asListBox` escape hatch:** For purely selectable lists (dropdown contexts), expose `asListBox` prop that switches the RAC primitive to `ListBox` (`role="listbox"` + `role="option"`). Alternatively, create a separate `ListBox`-backed component to avoid API confusion.
- [ ] **Drag-and-drop reorder:** RAC `GridList` supports `dragAndDropHooks`. Plan the API (a `reorderable` prop + `onReorder` callback) so enabling it later isn't breaking. Add to SPEC when designed.
- [ ] **Subheader sticky behavior:** Document that sticky subheaders require the `<List>` to sit inside a bounded `overflow-y: auto` ancestor; the component itself doesn't create that ancestor.
- [ ] **Leading type auto-detection:** Distinguishing `--leading-image` from `--leading-video` by inspecting React element type is fragile. Consider an explicit `leadingType?: 'icon' | 'avatar' | 'image' | 'video'` prop instead.
