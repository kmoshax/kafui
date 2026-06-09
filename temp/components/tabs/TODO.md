# Tabs — TODO

## MUI Equivalent

**`Tabs`** + **`Tab`** (`@mui/material`). `TabPanel` requires `@mui/lab` or manual implementation.

MUI Tabs are CSS-in-JS (Emotion), use physical `marginLeft`/`marginRight` for some icon spacing, do not ship `primary`/`secondary` M3 variants, have no `label-large` vs `title-small` typescale distinction, and don't support `keyboardActivation="manual"`. Every gap is a kafUI win.

---

## Beat-MUI Opportunities

### 1. True M3 `primary`/`secondary` variants — MUI ships one tab style

MUI Tabs have a single style with `indicatorColor` / `textColor` props as escape hatches. kafUI ships two semantically distinct M3 variants with correct typescales, indicator dimensions, and color roles per variant.

**Tasks:**
- [ ] `.tabs--primary .tabs__tab`: `font: var(--label-large-font); font-size: var(--label-large-size); font-weight: var(--label-large-weight)`; active color `--primary`; indicator `3px` full-width, `border-start-start-radius: var(--corner-full); border-start-end-radius: var(--corner-full)`
- [ ] `.tabs--secondary .tabs__tab`: `font: var(--title-small-font); font-size: var(--title-small-size); font-weight: var(--title-small-weight)`; active color `--on-surface`; indicator `2px` content-width, `border-radius: var(--corner-full)`
- [ ] Primary tab bar bottom border: `border-block-end: 1px solid var(--surface-variant)` — note: use `--surface-variant` (the outline-variant equivalent for tab dividers per M3)
- [ ] Secondary tab bar: no bottom border

### 2. Sliding indicator with no layout thrash — MUI's implementation uses inline styles via Emotion

MUI computes the indicator's width and offset in a `useEffect` and sets inline `style` — this works but requires Emotion wrapper overhead. kafUI sets `--indicator-x` and `--indicator-w` CSS custom properties directly on the `.tabs__list` DOM element in a single `useLayoutEffect`, then drives the indicator with pure CSS `transform` and `width`.

**Tasks:**
- [ ] `useTabIndicator(listRef, selectedKey)` hook:
  - `useLayoutEffect`: read `getBoundingClientRect()` of active `.tabs__tab` and `.tabs__list`
  - Compute `offsetX = tab.left - list.left` (LTR) or `offsetX = list.right - tab.right` (RTL, via `useLocale()`)
  - Set `listEl.style.setProperty('--indicator-x', offsetX + 'px')` and `'--indicator-w', tabWidth + 'px'`
  - For secondary variant: measure inner `.tabs__tab-label` (+ `.tabs__tab-icon` if present) width instead of full tab width
  - Attach `ResizeObserver` on the list; recompute on resize
- [ ] `.tabs__indicator`: `position: absolute; bottom: 0; left: 0; height: var(--indicator-h-primary); background: var(--primary); transform: translateX(var(--indicator-x, 0)); width: var(--indicator-w, 0)`
- [ ] Secondary: `height: var(--indicator-h-secondary); background: var(--on-surface)`
- [ ] Transition: `transform var(--duration-medium2) var(--easing-emphasized), width var(--duration-medium2) var(--easing-emphasized)` — wrapped in `@media (prefers-reduced-motion: no-preference)`
- [ ] `@media (prefers-reduced-motion: reduce)`: no transition; indicator jumps instantly

### 3. `keyboardActivation="manual"` — MUI always activates on arrow key

MUI's tabs activate immediately on arrow key focus, which triggers data fetches for async panels. RAC's `keyboardActivation` prop decouples focus from activation. kafUI exposes this directly.

**Tasks:**
- [ ] Forward `keyboardActivation` prop to RAC `<Tabs>` (maps directly to RAC's same-named prop)
- [ ] Default: `"automatic"` for standard in-page content
- [ ] Document: use `keyboardActivation="manual"` for panels with async data (avoids fetching on arrow key navigation)
- [ ] Test: in manual mode, arrow keys move focus without changing `selectedKey`; Enter/Space activates

### 4. Zero-JS CSS theming — MUI requires `styleOverrides` + `sx` for color customization

MUI color customization for tabs requires `MuiTabs.styleOverrides.indicator`, `MuiTab.styleOverrides.root`, or `sx` props — all Emotion, all JS. kafUI: override `--primary` or `--on-surface` at any scope in plain CSS.

**Tasks:**
- [ ] Verify all colors reference unprefixed tokens: `--primary`, `--on-surface`, `--on-surface-variant`, `--surface-variant`
- [ ] No hardcoded hex or `rgba()` values
- [ ] Dark mode: `color-scheme: dark` on `:root` — tabs look correct with zero component-level changes
- [ ] Theme test: changing `--source` at `:root` re-derives `--primary` → indicator + active tab color update automatically

### 5. RTL arrow key + indicator direction — MUI has known RTL gaps

Some MUI versions don't flip arrow key direction for RTL; icon `margin-left` values aren't logical. RAC handles arrow key RTL automatically. kafUI uses logical properties and direction-aware indicator offset.

**Tasks:**
- [ ] Icon spacing: `margin-block-end: 4px` when `--icon-top`; `margin-inline-end: 8px` when `--icon-inline` (NOT `margin-left`)
- [ ] `useTabIndicator`: use `useLocale()` from RAC; when `dir === 'rtl'`, compute `offsetX = list.right - tab.right` instead of `tab.left - list.left`
- [ ] Test: `dir="rtl"` — indicator slides correctly; arrow keys (`←` = next, `→` = previous) correct via RAC

### 6. Full compound API with RAC base — MUI requires `@mui/lab` for TabPanel

MUI separates `TabPanel` into `@mui/lab` and requires manual `id`/`aria-controls`/`aria-labelledby` wiring. RAC `Tabs` + `TabPanel` handle ARIA linkage automatically from `id` prop pairing. kafUI wraps this as a single `Tabs` compound with `.List`, `.Tab`, `.Panel`.

**Tasks:**
- [ ] Wrap RAC `Tabs` → `<Tabs>` forwarding `selectedKey`, `defaultSelectedKey`, `onSelectionChange`, `keyboardActivation`; apply `.tabs`, `.tabs--{variant}`, `.tabs--{layout}` classes
- [ ] Wrap RAC `TabList` → `<Tabs.List>` adding `__indicator` child; call `useTabIndicator`; apply `.tabs__list`; if `layout="scrollable"` wrap in `__scroller`
- [ ] Wrap RAC `Tab` → `<Tabs.Tab>` rendering `__tab-icon`, `__tab-label`, `__tab-badge` children; apply BEM class modifiers; `::after` state layer via CSS
- [ ] Wrap RAC `TabPanel` → `<Tabs.Panel>` applying `.tabs__panel`; ensure `tabindex="0"` (RAC sets this)
- [ ] `Tabs.List` + `Tabs.Tab` + `Tabs.Panel` as static properties on `Tabs` component (NOT as separate package-level exports)

### 7. Scrollable layout with accessible scroll buttons — MUI's `scrollButtons="auto"` uses JS visibility

**Tasks:**
- [ ] `layout="scrollable"`: wrap `.tabs__list` in `.tabs__scroller` (`overflow-x: auto; scrollbar-width: none; scroll-snap-type: none`)
- [ ] `.tabs__scroller::-webkit-scrollbar { display: none }` for Webkit
- [ ] Leading/trailing `.tabs__scroll-button`: RAC `<Button onPress={…}>` with `aria-label="Scroll tabs left"` / `"Scroll tabs right"` (localized via `useTranslation` or pass-through prop)
- [ ] Show/hide scroll buttons: `IntersectionObserver` on first/last tab; set `aria-hidden="true"` + `visibility: hidden` on button when no overflow in that direction
- [ ] Scroll buttons: 48 dp wide, full tab-list height; icon color `--on-surface-variant`

---

## Styles (`@kafui/styles`)

- [ ] Create `src/components/tabs/_tabs.css` inside `@layer kafui { }`
- [ ] `.tabs`: `display: flex; flex-direction: column`
- [ ] `.tabs__list`: `display: flex; flex-direction: row; position: relative` (contains absolutely-positioned indicator)
- [ ] `.tabs__tab`: `min-height: var(--tab-min-h); padding-inline: var(--tab-px); position: relative; overflow: hidden; cursor: pointer; display: flex; align-items: center; justify-content: center`
- [ ] Fixed layout: `.tabs--fixed .tabs__tab { flex: 1 }`
- [ ] `.tabs__panel`: `padding-block-start: 16px; outline: none` (RAC sets `tabindex="0"`)
- [ ] Disabled tab: `color: color-mix(in oklch, var(--on-surface) 38%, transparent); pointer-events: none`

## React (`@kafui/react`)

- [ ] Create `src/components/Tabs/Tabs.tsx`
- [ ] Export `Tabs` (with `.List`, `.Tab`, `.Panel` as static properties)
- [ ] Do NOT export `TabList`, `Tab`, `TabPanel` as top-level package exports — use dot notation exclusively

## Tests

- [ ] Arrow key roving tabindex: `←`/`→` move focus between tabs
- [ ] Manual mode: `←`/`→` focus without activating; Enter activates
- [ ] `Home`/`End` jump to first/last enabled tab
- [ ] Disabled tabs skipped in arrow navigation; `aria-disabled="true"`
- [ ] `aria-controls`/`aria-labelledby` pairing: each tab points to its panel; each panel points to its tab
- [ ] Inactive panels have `hidden` attribute
- [ ] Screen reader: tab announced as "tab, N of M, selected/not selected"
- [ ] Scrollable: scroll buttons appear/disappear correctly; keyboard-operable
- [ ] Indicator slides on tab change; no layout shift (use `getBoundingClientRect` timing test)
- [ ] Reduced motion: indicator jumps, no transition
- [ ] RTL: arrow key direction flips; indicator offset correct

## QA

- [ ] Primary indicator: 3 dp high, full tab width, `--corner-full` top corners; `--primary` color
- [ ] Secondary indicator: 2 dp high, content-width pill, `--corner-full` all corners; `--on-surface` color
- [ ] `label-large` vs `title-small` typography per variant — verify with design tokens
- [ ] Indicator slides at 60 fps; no layout shift; `transform` only (no `left` transition)
- [ ] Dark mode: `color-scheme: dark` — correct colors for both variants
- [ ] `dir="rtl"`: indicator slides correct direction; arrow keys flipped
