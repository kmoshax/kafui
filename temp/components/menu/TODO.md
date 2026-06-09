# Menu — TODO

## MUI Equivalent

`Menu` + `MenuItem` from `@mui/material`. For grouped items: `MenuList` + `ListSubheader`. Selectable items: `MenuItem` with manual `selected` prop (no built-in `selectionMode`). Submenu: no official support — requires third-party or manual Popper composition. Trigger: `anchorEl` + controlled `open` state wired manually.

---

## kafUI beats MUI — opportunities

| Dimension | kafUI wins | MUI pain |
|---|---|---|
| **Submenu first-class** | RAC `SubmenuTrigger` (≥1.3) wires keyboard (`→`/`←`), RTL-aware direction, `aria-haspopup` + `aria-expanded` — zero boilerplate | No official MUI submenu; every project hand-rolls Popper + custom keyboard handler; a11y invariably broken |
| **Correct selectable ARIA** | `selectionMode` auto-applies `menuitemradio` / `menuitemcheckbox` + `aria-checked`; no manual `role` overrides needed | MUI `MenuItem selected` prop is visual-only; `role` is always `menuitem`; radio/checkbox semantics need manual `role` + `aria-checked` — easy to forget, easy to get wrong |
| **Type-ahead built-in** | RAC `Menu` provides type-ahead (typing "D" jumps to "Delete") automatically | MUI Menu has no type-ahead by default |
| **Zero runtime CSS** | `@layer kafui` + `::before` state layer; all styles static — menu renders to DOM with zero JS style work | Emotion generates class names at runtime; Portal rendering + `MenuItem` each fire Emotion; measurable frame cost on slow devices |
| **CSS-var theming** | Override `--surface-container`, `--corner-extra-small`, `--elevation-2` at any scope; one CSS declaration changes the whole menu appearance | `theme.components.MuiMenu.styleOverrides` + `theme.components.MuiMenuItem.styleOverrides` — separate JS objects, no scope isolation, requires `ThemeProvider` |
| **Compound API DX** | `<Menu trigger={…}>` + `<Menu.Item>` + `<Menu.Section>` — self-contained, no external state wiring | `anchorEl` + `open` + `onClose` on `<Menu>`; each `MenuItem` needs its own `onClick` — 4-6 lines of boilerplate before any items appear |
| **RTL automatic** | RAC `Popover` placement flips; submenu indicator uses `:dir(rtl)` CSS; `padding-inline-*` logical — zero config | MUI RTL needs `rtl: true` in theme + `jss-rtl` plugin configured at app root |
| **Copy/extend** | Single `@layer kafui` block; rename `.menu` → `.context-menu` in a fork; no Emotion class-name archaeology | Overriding MUI requires knowing undocumented internal class names (`MuiMenu-paper`, `MuiMenuItem-root`) that change across minor versions |

---

## Actionable TODO Checklist

### CSS (`@kafui/styles`)

#### Layer & container
- [ ] Create `packages/styles/src/components/_menu.css`
- [ ] Wrap all rules in `@layer kafui { … }`
- [ ] Define `.menu` component-scoped vars: `--radius: var(--corner-extra-small); --min-w: 112px; --max-w: 280px; --item-h: 48px; --item-h-2line: 64px; --icon-size: 24px; --state-color: var(--on-surface)`
- [ ] `.menu__container`: `border-radius: var(--radius); background: var(--surface-container); box-shadow: var(--elevation-2); min-width: var(--min-w); max-width: var(--max-w); overflow: hidden; position: relative`
- [ ] Elevation tint `::before`: `background: var(--surface-tint); opacity: var(--elevation-tint-2); position: absolute; inset: 0; pointer-events: none`
- [ ] `.menu__list`: `padding-block: 8px; overflow-y: auto; max-height: 50dvh`

#### Items
- [ ] `.menu__item`: `min-height: var(--item-h); padding-inline: 12px; display: flex; align-items: center; gap: 12px; font-size: var(--label-large-size); font-weight: var(--label-large-weight); color: var(--on-surface); cursor: pointer; position: relative; overflow: hidden`
- [ ] State-layer `::before` on `.menu__item`: same pattern as chip — `background: var(--state-color); opacity: 0; transition: opacity var(--duration-short3) var(--easing-standard)`
- [ ] `[data-hovered]::before { opacity: var(--state-hover) }` — 0.08
- [ ] `[data-focused]::before { opacity: var(--state-focus) }` — 0.10
- [ ] `[data-pressed]::before { opacity: var(--state-pressed) }` — 0.10
- [ ] `.menu__item--has-supporting-text`: `min-height: var(--item-h-2line); align-items: flex-start; padding-block: 8px`
- [ ] Supporting text: `font-size: var(--label-medium-size); color: var(--on-surface-variant)`
- [ ] Trailing text: `margin-inline-start: auto; color: var(--on-surface-variant); font-size: var(--label-large-size)`
- [ ] `[data-disabled]`: `opacity: 0.38; pointer-events: none; cursor: default`
- [ ] Leading icon: `width: var(--icon-size); height: var(--icon-size); flex-shrink: 0; color: var(--on-surface-variant)`
- [ ] Submenu icon: `margin-inline-start: auto`; `:dir(rtl) .menu__item-submenu-icon { transform: scaleX(-1) }`

#### Layout alignment — mixed icon/no-icon rows
- [ ] `.menu__item--has-leading-icon .menu__item-label`: `padding-inline-start: 0` (icon provides spacing)
- [ ] When `--has-leading-icon` modifier is absent but sibling items have it: add `padding-inline-start: calc(var(--icon-size) + 12px)` for label alignment — apply this at the `.menu__list` level with `:has(.menu__item--has-leading-icon)` selector

#### Divider / Section
- [ ] `.menu__divider`: `block-size: 1px; background: var(--outline-variant); margin-block: 8px; border: none`
- [ ] `.menu__section-header`: `font-size: var(--label-small-size); font-weight: var(--label-small-weight); color: var(--on-surface-variant); padding-inline: 16px; padding-block: 8px; pointer-events: none; user-select: none`

#### Motion
- [ ] `@keyframes menu-open { from { opacity: 0; transform: scale(0.9) } to { opacity: 1; transform: scale(1) } }`
- [ ] `@keyframes menu-close { from { opacity: 1 } to { opacity: 0 } }`
- [ ] `.menu__container[data-entering]`: `animation: menu-open var(--duration-short4) var(--easing-emphasized-decelerate); transform-origin: top left`
- [ ] `:dir(rtl) .menu__container[data-entering]`: `transform-origin: top right`
- [ ] `.menu__container[data-exiting]`: `animation: menu-close var(--duration-short2) var(--easing-emphasized-accelerate)`
- [ ] `@media (prefers-reduced-motion: reduce)` — disable all animations and transitions

### React components (`@kafui/react`)

- [ ] `packages/react/src/components/Menu/Menu.tsx`
  - [ ] Wrap RAC `MenuTrigger` + `Popover` + RAC `Menu`
  - [ ] `placement` prop → RAC `Popover` `placement`
  - [ ] `selectionMode` → RAC `Menu` `selectionMode`; gate ARIA roles
  - [ ] `onAction` → RAC `Menu` `onAction`
  - [ ] `trigger={null}` branch — render as bare `Menu` panel (submenu mode)
- [ ] `packages/react/src/components/Menu/MenuItem.tsx` — RAC `MenuItem`
  - [ ] Leading icon, trailing text, supporting text slots
  - [ ] Auto-apply `--has-leading-icon` modifier when `leadingIcon` present
  - [ ] Auto-apply `--has-supporting-text` when `supportingText` present
- [ ] `packages/react/src/components/Menu/MenuSection.tsx` — RAC `MenuSection` + `Header`
- [ ] `packages/react/src/components/Menu/MenuDivider.tsx` — RAC `Separator`; className `menu__divider`
- [ ] `packages/react/src/components/Menu/MenuSubmenuItem.tsx`
  - [ ] Wrap RAC `SubmenuTrigger` (requires RAC ≥ 1.3 — add version check / peer dep note)
  - [ ] Render chevron icon in `.menu__item-submenu-icon` slot
- [ ] Attach compound namespace: `Menu.Item`, `Menu.Section`, `Menu.Divider`, `Menu.SubmenuItem`
- [ ] Export `Menu` from `packages/react/src/index.ts`

### Testing
- [ ] Unit: menu opens on trigger press; closes on Escape; focus returns to trigger
- [ ] Unit: `↑/↓` navigation cycles items; `Home`/`End` jump to first/last
- [ ] Unit: type-ahead focuses matching item (RAC built-in — smoke test)
- [ ] Unit: `onAction` fires with correct `id`; item-level `onPress` also fires
- [ ] Unit: `selectionMode="single"` — selecting item deselects previous; `onSelectionChange` payload is `Set<string>`
- [ ] Unit: `selectionMode="multiple"` — multiple items independently selectable; payload correct
- [ ] Unit: `role="menuitemradio"` applied for single; `"menuitemcheckbox"` for multiple
- [ ] Unit: disabled item skipped by arrow navigation; `data-disabled` present; `onAction` not fired
- [ ] Unit: submenu opens on `→`; closes on `←`; focus returns to parent item
- [ ] Unit: RTL — submenu opens on `←`; menu placement flips (Popover)
- [ ] Unit: outside click closes menu; focus returns to trigger
- [ ] Unit: `--has-leading-icon` modifier applied when icon present; absent otherwise
- [ ] a11y: axe-core — open menu (standard, selectable single, selectable multiple)
- [ ] a11y: `aria-expanded` on submenu trigger item
- [ ] a11y: icons are `aria-hidden`; shortcut text has `aria-keyshortcuts`
- [ ] Visual regression: standard, selectable, with submenu, dividers, hover/focus, selected; light + dark; LTR + RTL

### Docs / Storybook
- [ ] Story: standard menu with icons, trailing shortcuts, dividers, section headers
- [ ] Story: selectable single (sort order) — persists across open/close
- [ ] Story: selectable multiple (filter labels)
- [ ] Story: menu with submenu (export → PDF/PNG)
- [ ] Story: long item list (scrollable container)
- [ ] Story: RTL layout
- [ ] Story: placement options (`bottom-start`, `bottom-end`, `top`)
- [ ] Story: disabled items mixed into list

### Open Questions
- [ ] RAC `SubmenuTrigger` stability: confirm it is stable in RAC 1.3+ (not alpha). Pin peer dep; add runtime warning if RAC version is too old.
- [ ] `max-height: 50dvh` on `.menu__list` — is `dvh` the right unit? On mobile, `100dvh` accounts for browser chrome; `50dvh` may be too tall. Consider `clamp(200px, 50dvh, 400px)`.
- [ ] Mixed icon/no-icon alignment via `:has()` — check browser support floor; provide a fallback (explicit `--has-leading-icon` on the list container as alternative).
- [ ] Should `Menu` support a `dense` variant (36 dp item height) for information-dense contexts? M3 doesn't spec it, but it is commonly needed. Defer to a `density` prop added later.
