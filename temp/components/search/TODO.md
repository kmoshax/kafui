# Search — TODO

## MUI equivalent

No direct M3 Search Bar/View equivalent in MUI. Closest approximations:
- `@mui/material/TextField` with `type="search"` — input field only; no bar shape, no view.
- `@mui/material/Autocomplete` — suggestions + combobox semantics, but not M3-styled; heavy prop surface (`renderOption`, `filterOptions`, `freeSolo`, etc.); requires extensive `sx`/`styled` wiring to approximate bar shape and view expansion.

Neither component models the M3 bar↔view transition, the pill shape, the on-scroll elevation, or the leading-icon swap. kafUI ships the entire pattern as a single compound component.

---

## Beat-MUI opportunities

### 1. The complete M3 Search bar→view pattern — MUI has nothing like it
MUI teams building M3-compliant search compose two unrelated components with extensive customization. kafUI provides `<Search>` which ships the bar, the full-screen view, the expand/collapse animation, and the suggestion list as one designed unit.

**Tasks: Search bar**
- [ ] Create `search.css` inside `@layer kafui { … }`
- [ ] `.search-bar`: `height: 56px; border-radius: var(--corner-full); background: var(--surface-container-high); display: flex; align-items: center; padding-inline: 16px; gap: 16px; position: relative; overflow: hidden`
- [ ] `.search-bar--scrolled`: `box-shadow: var(--elevation-2)`
- [ ] Elevation transition: `transition: box-shadow var(--duration-short4) var(--easing-standard)`
- [ ] `.search-bar__leading` / `__trailing`: `min-width: 48px; min-height: 48px`; icon color `--on-surface-variant`
- [ ] `.search-bar__input`: transparent; `body-large` typescale; `color: var(--on-surface)`; `::placeholder` `--on-surface-variant`
- [ ] State layer on bar: `--state-hover` / `--state-focus` via `data-hovered` / `data-focus-visible`

**Tasks: Search view**
- [ ] `.search-view`: `position: fixed; inset: 0; background: var(--surface); z-index: var(--z-modal, 1300); display: flex; flex-direction: column`
- [ ] `.search-view__header`: `height: 56px; display: flex; align-items: center; padding-inline: 8px; gap: 4px`
- [ ] `.search-view__divider`: `border-block-start: 1px solid var(--outline-variant)`
- [ ] `.search-view__suggestion`: `min-height: 56px; padding-inline: 16px; display: flex; align-items: center; gap: 16px; position: relative`
- [ ] State layer on each suggestion row
- [ ] `.search-view__suggestion-label`: `flex: 1; font-size: var(--body-large-size)`
- [ ] `.search-view__suggestion-sublabel`: `font-size: var(--label-medium-size); color: var(--on-surface-variant)`
- [ ] `.search-view__suggestion-trailing`: fill-in arrow; `color: var(--on-surface-variant)`

### 2. CSS expand/collapse animation — MUI autocomplete has no M3 full-screen expansion
The bar-to-view transition is a signature M3 motion: the bar grows to fill the screen with an emphasized easing. kafUI implements this with `clip-path` animation — no JS, no `height: auto` animation hacks.

**Tasks:**
- [ ] `@media (prefers-reduced-motion: no-preference)` block: `.search-view { clip-path: inset(0 0 100% 0); transition: clip-path var(--duration-medium2) var(--easing-emphasized) }` + `.search-view[data-open] { clip-path: inset(0) }`
- [ ] Close animation: `.search-view[data-closing] { clip-path: inset(0 0 100% 0); transition-duration: var(--duration-short4); transition-timing-function: var(--easing-emphasized-decelerate) }`
- [ ] Leading icon swap: CSS transition on icon opacity/transform `--duration-short2`; or React conditional render with exit animation
- [ ] Suggestion stagger-fade in: `animation: suggestion-fade-in var(--duration-short2)`; delay via inline `--stagger-delay` (set by React: `style={{ '--stagger-delay': `${index * 20}ms` }}`)
- [ ] `@media (prefers-reduced-motion: reduce)`: `transition: none; clip-path: none; animation: none` on all animated elements

### 3. Correct WAI-ARIA combobox pattern — MUI's Autocomplete has `role` inconsistencies
RAC `SearchField` + `ListBox` wires the full WAI-ARIA 1.2 combobox pattern correctly: `role="combobox"` + `aria-expanded` + `aria-controls` + `aria-activedescendant` on the input, `role="listbox"` on the suggestions, `role="option"` per row. MUI's `Autocomplete` has had issues with `role="searchbox"` vs `role="combobox"` depending on version.

**Tasks:**
- [ ] Wire RAC `SearchField` for input plumbing (label, clear button, `type="search"`)
- [ ] When view is open: set `role="combobox"` + `aria-expanded="true"` + `aria-controls={listboxId}` + `aria-autocomplete="list"` on the input (override SearchField's default if needed)
- [ ] When view is closed: input back to implicit `role="searchbox"` (type="search" suffices)
- [ ] `aria-activedescendant` managed by RAC `ListBox` keyboard navigation
- [ ] Each suggestion `role="option"` + `aria-selected` (active = true; not committed selection)
- [ ] `aria-busy="true"` on listbox when `isLoading`; show spinner in trailing slot
- [ ] `isLoading` spinner: inline SVG or icon; no JS animation library

### 4. On-scroll bar elevation — no component in MUI does this
M3 specifies the search bar lifts to elevation 2 when the page is scrolled past the top, signaling it is a persistent floating element. kafUI models this via a scroll observer hook setting `data-scrolled` on the bar, with a pure CSS elevation change.

**Tasks:**
- [ ] `useScrollElevation` hook: observes `scrollTarget` (defaults to `window`); sets `data-scrolled` boolean attribute on bar element when scroll position > 0
- [ ] `.search-bar[data-scrolled]`: `box-shadow: var(--elevation-2)`
- [ ] Remove scrolled elevation when view is open (search view is full-screen, no bar visible)
- [ ] `scrollTarget` prop accepts `React.RefObject<HTMLElement>` for SPA custom scroll containers

### 5. Clean single-component API — MUI needs two wired components, still gets it wrong
MUI requires `TextField` + `Autocomplete` for the nearest equivalent, and neither matches M3. kafUI's `<Search>` does it all: bar, view, suggestions, scroll elevation, icon swap — one import, one component.

**Tasks:**
- [ ] Remove `variant` prop (was misleading as bar/view state); use `isExpanded` / `onExpandedChange` for state management
- [ ] Infer leading icon: `'search'` at rest; `'arrow_back'` when `isExpanded`; consumer can still override via `leadingIcon` prop
- [ ] Clear (×) button: appears in trailing when `value` is non-empty; handled automatically by RAC `SearchField`'s built-in clear button; style as `__trailing` slot
- [ ] `onSubmit` prop: fires on Enter with no active suggestion
- [ ] `onSuggestionSelect`: fires when user selects a row; commits `suggestion.label` to input value
- [ ] Trailing fill-in action on suggestion: pressing the trailing arrow populates the input with `suggestion.label` but does NOT submit; `onValueChange` fires, `onSubmit` does not
- [ ] `Escape` closes view: call `onExpandedChange(false)`; return focus to bar root via `ref.focus()`

### 6. RTL and dark mode zero-config
**Tasks:**
- [ ] Logical properties throughout: `padding-inline`, `inset-inline-start/end`, `border-block-start`
- [ ] `[dir="rtl"] .icon--directional { transform: scaleX(-1) }` for back-arrow and fill-in arrow
- [ ] Verify `--surface-container-high`, `--surface`, `--outline-variant`, `--on-surface`, `--on-surface-variant` all flip under `color-scheme: dark`
- [ ] Write dark mode visual test

---

## Styles (`@kafui/styles`) summary checklist
- [ ] `search.css` — all rules in `@layer kafui { … }`
- [ ] Component-internal vars on `.search-bar`: `--height`, `--radius`, `--pad-inline`
- [ ] All system token names unprefixed: `--surface-container-high`, `--surface`, `--on-surface`, `--on-surface-variant`, `--outline-variant`, `--corner-full`, `--corner-extra-large`, `--corner-none`, `--elevation-0`, `--elevation-2`, `--z-modal`, `--body-large-size/weight/line-height`, `--label-medium-size/weight`, `--easing-emphasized`, `--easing-emphasized-decelerate`, `--easing-standard`, `--duration-medium2`, `--duration-short4`, `--duration-short2`, `--state-hover`, `--state-focus`, `--state-pressed`
- [ ] No `kafui-` prefix on any BEM class
- [ ] `@layer kafui` provides collision safety
- [ ] `@media (prefers-reduced-motion: no-preference)` wraps ALL animations (opt-in, not opt-out)

## React (`@kafui/react`) summary checklist
- [ ] Single `<Search>` component; no `variant` prop for bar/view — use `isExpanded`
- [ ] RAC `SearchField` for input plumbing
- [ ] RAC `ListBox` + `ListBoxItem` for suggestions with combobox ARIA wiring
- [ ] `useScrollElevation` hook for on-scroll bar elevation
- [ ] Leading icon auto-swap (search ↔ arrow_back) based on `isExpanded`
- [ ] Focus management: open → focus input; Escape → focus bar root
- [ ] Inline `--stagger-delay` style per suggestion for stagger animation
- [ ] `data-open` / `data-closing` attributes on `.search-view` to drive CSS transitions

## Testing
- [ ] Screen-reader walkthrough: VoiceOver + NVDA — confirm `searchbox` announcement, suggestion count (`N results available`), `aria-activedescendant` updates
- [ ] Keyboard-only: open view (Enter/Space on bar), navigate suggestions (↓↑), select (Enter), close (Escape)
- [ ] On-scroll elevation: scrolling page sets `data-scrolled`; `box-shadow` changes; scrolling back removes it
- [ ] Leading icon swap: transitions from search to arrow_back on expand and back on close
- [ ] Trailing fill-in: pressing fills input without submitting; `onSubmit` NOT called
- [ ] `isLoading`: spinner shows; `aria-busy="true"` on listbox
- [ ] RTL: icon positions and directional icons correct
- [ ] Dark mode: all surface/text colors correct
- [ ] Reduced motion: instant open/close; no animation; `clip-path` removed
- [ ] `scrollTarget` ref: scroll observer works on a custom scroll container, not just window
- [ ] No `onClick` anywhere — only `onPress` via RAC
