# Search

M3 Search provides a persistent, pill-shaped search bar (resting state) that expands into a full-screen search view with a suggestion/autocomplete list. M3 category: **Navigation → Search**. The bar and view are two states of one compound component, not two separate components.

---

## Anatomy / parts

### Search bar (resting)

| BEM element | Description |
|---|---|
| `.search-bar` | Root; pill surface, always visible; press opens view |
| `.search-bar__leading` | Leading icon slot — search/menu icon resting; back-arrow when view is open |
| `.search-bar__input` | `<input type="search">`; `role="searchbox"` implicit; becomes `role="combobox"` when view expands |
| `.search-bar__trailing` | Trailing icon slot(s) — mic, avatar, clear (×) when value is non-empty |
| `.state-layer` | Hover/focus/pressed overlay on bar root |

### Search view (expanded)

| BEM element | Description |
|---|---|
| `.search-view` | Full-screen overlay; `position: fixed; inset: 0`; contains header + suggestion list |
| `.search-view__header` | Back icon + input field + trailing icon; same height as bar (56 dp) |
| `.search-view__input` | Active input inside the view; same RAC `SearchField` as bar — same element, repositioned by CSS |
| `.search-view__divider` | 1 dp `--outline-variant` line separating header from suggestions |
| `.search-view__list` | Suggestions container; `role="listbox"` |
| `.search-view__suggestion` | Individual suggestion row; `role="option"` |
| `.search-view__suggestion-icon` | Leading icon (history, search, or custom) |
| `.search-view__suggestion-label` | Primary text |
| `.search-view__suggestion-sublabel` | Optional secondary text or metadata |
| `.search-view__suggestion-trailing` | Optional fill-in arrow icon — populates query without submitting |

---

## Variants / states

### Component-level states

| State | Trigger | Visual |
|---|---|---|
| Bar — default | Resting | `--surface-container-high` pill; elevation 0 |
| Bar — hovered | Pointer over bar | `--state-hover` (8%) state layer |
| Bar — scrolled | Page scrolled past top | Elevation `--elevation-2` (shadow appears) |
| Bar — expanded | Tapped / pressed | View opens; leading icon swaps to back-arrow; input focused |
| View — empty | No query value | Placeholder; recent/default suggestions list |
| View — typing | Value non-empty | Live-filtered suggestions; clear (×) icon in trailing |
| View — loading | `isLoading` prop | Spinner in trailing; `aria-busy="true"` on listbox |
| View — suggestion hovered | Pointer over suggestion | `--state-hover` on row |
| View — suggestion focused | Keyboard navigation | `--state-focus` on row; `aria-activedescendant` pattern |
| View — suggestion pressed | Tap/click on row | `--state-pressed` on row |

---

## Design tokens

All names are unprefixed system roles from `@kafui/styles`. Component-internal vars declared on `.search-bar` / `.search-view`.

### Color
| Role | Token |
|---|---|
| Bar background | `--surface-container-high` |
| Bar input text | `--on-surface` |
| Bar placeholder | `--on-surface-variant` |
| Bar leading/trailing icon | `--on-surface-variant` |
| View background | `--surface` |
| Suggestion hover state layer color | `--on-surface` |
| Divider | `--outline-variant` |

### Shape
- Bar: `--corner-full` (fully rounded pill; bar height 56 dp → radius ≥ 28 dp)
- View header area: `--corner-none` (flush with viewport edges)
- View container top corners (when not full-width): `--corner-extra-large` (28 dp top corners only)

### Elevation
- Bar resting: `--elevation-0` (no shadow; blends with surface)
- Bar scrolled: `--elevation-2`
- View: `--elevation-0` (full-screen; no shadow needed)

### Typography
- Input text: `--body-large-size`, `--body-large-weight`, `--body-large-line-height`
- Suggestion primary label: `--body-large-size`, `--body-large-weight`
- Suggestion secondary label: `--label-medium-size`, `--label-medium-weight`

### Motion
- Bar → view expand: `--easing-emphasized` + `--duration-medium2` (~300 ms); `clip-path` or `height` expansion
- View → bar collapse: `--easing-emphasized-decelerate` + `--duration-short4`
- Leading icon swap: `--duration-short2`
- Suggestion stagger-fade in: per-item delay, `--duration-short2` each
- `@media (prefers-reduced-motion: no-preference)`: all transitions opt-in (default off, add on no-preference match)
- `@media (prefers-reduced-motion: reduce)`: instant state switch; no expand/collapse animation

### State layer
- `--state-hover`: 0.08
- `--state-focus`: 0.10
- `--state-pressed`: 0.10

### CSS structure (illustrative)

```css
@layer kafui {
  .search-bar {
    --height: 56px;
    --radius: var(--corner-full);
    --pad-inline: 16px;

    display: flex;
    align-items: center;
    height: var(--height);
    border-radius: var(--radius);
    background: var(--surface-container-high);
    padding-inline: var(--pad-inline);
    gap: 16px;
    position: relative;
    overflow: hidden;
    box-shadow: var(--elevation-0);
    transition: box-shadow var(--duration-short4) var(--easing-standard);
    color: var(--on-surface);
  }

  .search-bar--scrolled {
    box-shadow: var(--elevation-2);
  }

  .search-bar .state-layer {
    color: var(--on-surface);
  }
  .search-bar[data-hovered] .state-layer { opacity: var(--state-hover); }
  .search-bar[data-focus-visible] .state-layer { opacity: var(--state-focus); }

  .search-bar__leading,
  .search-bar__trailing {
    flex: 0 0 auto;
    color: var(--on-surface-variant);
    min-width: 48px;
    min-height: 48px;
    display: flex;
    align-items: center;
    justify-content: center;
  }

  .search-bar__input {
    flex: 1;
    background: transparent;
    border: none;
    outline: none;
    font-size: var(--body-large-size);
    font-weight: var(--body-large-weight);
    line-height: var(--body-large-line-height);
    color: var(--on-surface);
  }
  .search-bar__input::placeholder { color: var(--on-surface-variant); }

  /* View */
  .search-view {
    position: fixed;
    inset: 0;
    background: var(--surface);
    z-index: var(--z-modal, 1300);
    display: flex;
    flex-direction: column;
  }

  .search-view__header {
    display: flex;
    align-items: center;
    height: 56px;
    padding-inline: 8px;
    gap: 4px;
  }

  .search-view__divider {
    border: none;
    border-block-start: 1px solid var(--outline-variant);
  }

  .search-view__list {
    flex: 1;
    overflow-y: auto;
  }

  .search-view__suggestion {
    display: flex;
    align-items: center;
    min-height: 56px;
    padding-inline: 16px;
    gap: 16px;
    position: relative;
    cursor: pointer;
    color: var(--on-surface);
  }
  .search-view__suggestion[data-hovered] .state-layer { opacity: var(--state-hover); }
  .search-view__suggestion[data-focus-visible] .state-layer { opacity: var(--state-focus); }
  .search-view__suggestion[aria-selected="true"] .state-layer { opacity: var(--state-focus); }

  .search-view__suggestion-icon { color: var(--on-surface-variant); flex: 0 0 24px; }
  .search-view__suggestion-label { flex: 1; font-size: var(--body-large-size); }
  .search-view__suggestion-sublabel { font-size: var(--label-medium-size); color: var(--on-surface-variant); }
  .search-view__suggestion-trailing { color: var(--on-surface-variant); flex: 0 0 24px; }

  /* Expand/collapse animation (opt-in under no-preference) */
  @media (prefers-reduced-motion: no-preference) {
    .search-view {
      clip-path: inset(0 0 100% 0);
      transition: clip-path var(--duration-medium2) var(--easing-emphasized);
    }
    .search-view[data-open] {
      clip-path: inset(0);
    }
    .search-view[data-closing] {
      clip-path: inset(0 0 100% 0);
      transition: clip-path var(--duration-short4) var(--easing-emphasized-decelerate);
    }

    .search-view__suggestion {
      opacity: 0;
      animation: suggestion-fade-in var(--duration-short2) var(--easing-standard) both;
    }
    @keyframes suggestion-fade-in {
      from { opacity: 0; transform: translateY(4px); }
      to   { opacity: 1; transform: translateY(0); }
    }
    /* stagger via custom property set inline by React */
    .search-view__suggestion { animation-delay: var(--stagger-delay, 0ms); }
  }

  /* RTL */
  [dir="rtl"] .search-bar__leading .icon--directional { transform: scaleX(-1); }
  [dir="rtl"] .search-view__suggestion-trailing .icon--directional { transform: scaleX(-1); }

  @media (prefers-reduced-motion: reduce) {
    .search-bar { transition: none; }
    .search-view { transition: none; clip-path: none; }
    .search-view__suggestion { animation: none; opacity: 1; }
  }
}
```

---

## Interaction & accessibility

### ARIA / WCAG

The search component spans two ARIA patterns depending on state:

**Bar resting (no suggestions visible):**
- `<input type="search">` has implicit `role="searchbox"`.
- No `aria-expanded` needed while view is closed.

**View expanded (suggestions visible):**
- Input: `role="combobox"` + `aria-expanded="true"` + `aria-controls="<listbox-id>"` + `aria-autocomplete="list"` + `aria-activedescendant="<active-option-id>"`.
- Suggestions: `role="listbox"` + `id` referenced by `aria-controls`.
- Each suggestion: `role="option"` + `aria-selected` (reflects keyboard-active state, not committed selection).

This is the WAI-ARIA 1.2 combobox pattern. RAC's `Autocomplete`/`ComboBox` primitive handles all wiring automatically.

**Note on `SearchField` vs `ComboBox`/`Autocomplete`:**
- Use RAC `SearchField` for the input/label/clear-button infrastructure.
- Use RAC `Autocomplete` (if available in RAC version) or manually wire `ListBox` + `aria-activedescendant` for the suggestion list. RAC `ComboBox` is an option but its popup pattern may conflict with the full-screen view — prefer `Autocomplete` + `ListBox` for maximum control.

### Keyboard navigation
- `↓` / `↑`: move `aria-activedescendant` through suggestion list; input retains DOM focus; active option highlighted via CSS `.search-view__suggestion[aria-selected="true"]`.
- `Enter`: confirm highlighted suggestion OR submit current query if no active suggestion.
- `Escape`: close view; return focus to bar trigger.
- `Tab`: close view; move focus out of component.
- Typing any character: re-filters suggestions; resets active descendant.

### Focus management
- Opening view: focus moves to the input immediately.
- Closing view (Escape): focus returns to the bar root element.
- Do not trap focus in the view — `Tab` should escape naturally.

### RTL
- Leading and trailing icon positions swap via `flex-direction` + logical padding.
- Back-arrow and fill-in arrow are directional icons; flip via `transform: scaleX(-1)` on `.icon--directional` under `[dir="rtl"]`.

### Touch target
- Each suggestion row: `min-height: 56px` (48 dp with 4 dp margin each side = 56 dp total row).
- Leading/trailing icons: `min-width: 48px; min-height: 48px` with invisible padding inside flex layout.

### Reduced motion
- Expand/collapse animation replaced by instant `display` switch.
- Suggestion stagger animation disabled.

---

## Proposed kafUI React API

```tsx
// RAC base: SearchField (input plumbing) + Autocomplete/ListBox (suggestion list)
// Single <Search> component manages both bar and view states

interface SearchSuggestion {
  key: string;
  label: string;
  sublabel?: string;
  icon?: string;            // sprite name: 'history', 'search', custom
  trailingAction?: string;  // sprite name for fill-in arrow
}

interface SearchProps {
  // Input value
  value?: string;
  defaultValue?: string;
  onValueChange?: (value: string) => void;

  // View open/close
  isExpanded?: boolean;
  defaultExpanded?: boolean;
  onExpandedChange?: (isExpanded: boolean) => void;

  // Content
  placeholder?: string;         // default: 'Search'
  suggestions?: SearchSuggestion[];
  onSuggestionSelect?: (suggestion: SearchSuggestion) => void;
  onSubmit?: (value: string) => void;

  // Slots
  leadingIcon?: string;         // sprite name; default: 'search'; auto-becomes 'arrow_back' when expanded
  trailingIcon?: string;        // e.g. 'mic', 'account_circle'

  // State
  isLoading?: boolean;          // shows spinner in trailing; sets aria-busy on listbox

  // Scroll-driven bar elevation
  scrollTarget?: React.RefObject<HTMLElement>; // element to observe for scroll; default: window

  className?: string;
}

// Usage
<Search
  placeholder="Search"
  suggestions={suggestions}
  onSuggestionSelect={handleSelect}
  onSubmit={handleSubmit}
  trailingIcon="mic"
/>

// Controlled expansion
<Search
  isExpanded={searchOpen}
  onExpandedChange={setSearchOpen}
  value={query}
  onValueChange={setQuery}
  suggestions={filteredSuggestions}
/>
```

**BEM classes emitted:**
- `.search-bar` (resting state root) / `.search-view` (expanded state root)
- `.search-bar--scrolled` driven by scroll observer (set via `data-scrolled` attribute which CSS targets)
- `.search-bar__leading`, `.search-bar__input`, `.search-bar__trailing`
- `.search-view__header`, `.search-view__input`, `.search-view__divider`, `.search-view__list`
- `.search-view__suggestion`, `.search-view__suggestion-icon`, `.search-view__suggestion-label`, `.search-view__suggestion-sublabel`, `.search-view__suggestion-trailing`
- `.state-layer` on bar root and on each suggestion row
- RAC data attributes: `data-hovered`, `data-focus-visible`, `data-pressed` on bar; `aria-selected`, `aria-activedescendant` on suggestions (set by RAC)

**CSS layer:** all rules in `@layer kafui { … }`. No `kafui-` prefix.

**React Aria base:**
- `SearchField` from `react-aria-components`: input, label, clear button.
- `Autocomplete` (RAC) or `ListBox` + `ListBoxItem` for suggestions with `aria-activedescendant` pattern.
- `aria-activedescendant` managed by RAC's listbox keyboard navigation.

**Deviations / notes:**
- No `freeSolo`, `multiple`, `filterOptions` — kafUI Search matches M3 Search use case, not a general autocomplete.
- `onPress` everywhere (RAC), never `onClick`.
- The bar and view share the same underlying `SearchField` input — the field is rendered once; CSS repositions it between bar and view contexts. This avoids double-focus management.
- `variant` prop from the original draft is removed: the bar/view duality is managed by `isExpanded`, not a static `variant`. Using `variant` for open state was misleading.
- `scrollTarget` lets consumers pass a ref to a scrollable container (e.g. a page scroll div) rather than relying on `window` scroll — necessary for SPAs with custom scroll containers.
