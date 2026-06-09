# List

**Purpose:** Presents a vertically stacked collection of items. Each item is a row containing a combination of leading element, text block, and trailing element. M3 category: **Containment → Lists**.

---

## Anatomy / Parts → BEM Elements

```
.list                      root <ul> or GridList container
.list__subheader           optional sticky/non-sticky group label; label-large

.list-item                 root item element; <li> or interactive GridListItem
.list-item__state-layer    absolutely-positioned hover/focus/pressed overlay
.list-item__leading        leading slot; contains icon, avatar, or image
.list-item__content        text block; flex column
.list-item__headline       primary line; body-large
.list-item__supporting     secondary/tertiary lines; body-medium
.list-item__trailing       trailing slot; icon, text, checkbox, radio, or switch
.list-item__overline       optional overline text above headline; label-small
```

---

## Variants

### Line density (on `List.Item`)

| Variant | `lines` prop | Height | Description |
|---|---|---|---|
| One-line | `1` (default) | 56 dp | Headline only; compact rows |
| Two-line | `2` | 72 dp | Headline + one line of supporting text |
| Three-line | `3` | 88 dp | Headline + two lines supporting text; supporting truncates at 2 lines |

### Leading element types (on `List.Item`)

| Type | `leading` slot content | BEM modifier |
|---|---|---|
| Icon | `<Icon name="..." />` | `.list-item--leading-icon` |
| Avatar | `<Avatar>` | `.list-item--leading-avatar` |
| Image | `<img>` (40×40 or 56×56 dp) | `.list-item--leading-image` |
| Video thumbnail | `<img>` (100×56 dp) | `.list-item--leading-video` |
| None | absent | (no modifier) |

### Trailing element types

| Type | `trailing` slot content | Notes |
|---|---|---|
| Icon | `<Icon>` | Taps fire `onTrailingPress`; must have `aria-label` |
| Text/meta | plain text | e.g. "3 min", "Jan 12" |
| Checkbox | `<Checkbox>` | Controlled; item selectable |
| Radio | `<Radio>` | Controlled within a RadioGroup |
| Switch | `<Switch>` | Controlled |

---

## States

| State | State-layer opacity | Notes |
|---|---|---|
| Enabled | 0% | — |
| Hovered | 8% (`--on-surface`) | Entire row highlights |
| Focused | 10% (`--on-surface`) | Focus ring on item |
| Pressed | 10% (`--on-surface`) | Ripple / state layer |
| Selected / Activated | Container: `--secondary-container`; text/icon: `--on-secondary-container` | For `selectionMode` use cases |
| Disabled | 38% `--on-surface` for text + icon; no state layer | `isDisabled` prop |
| Dragged | 16% state layer + `--elevation-1` | Drag-and-drop reorder |

State-layer color is `--on-surface` for non-selected items; `--on-secondary-container` for selected items.

---

## Design Tokens

### Color
- Background (default): transparent (inherits surface from parent)
- Selected container: `--secondary-container`
- Selected text/icon: `--on-secondary-container`
- Headline text: `--on-surface`
- Supporting text: `--on-surface-variant`
- Trailing icon: `--on-surface-variant`
- Overline: `--on-surface-variant`
- Leading icon: `--on-surface-variant` (active/primary context: `--primary`)
- State layer: `--on-surface` at opacity

### Shape
- Item within a list: `--corner-none` (0) — flush edges
- Standalone item (outside a list): `--corner-extra-small` (4 dp)
- Selected/activated indicator in navigation contexts: `--corner-full` (pill) — not applicable to generic list

### Typography
- Headline: `--body-large-size` / `--body-large-weight` / `--body-large-line-height`
- Supporting text: `--body-medium-size` / `--body-medium-weight`
- Overline: `--label-small-size` / `--label-small-weight`
- Subheader: `--label-large-size` / `--label-large-weight`
- Trailing text (meta): `--label-small-size` / `--label-small-weight`

### State layer
- `--state-hover`: 0.08
- `--state-focus`: 0.10
- `--state-pressed`: 0.10
- `--state-dragged`: 0.16

---

## Interaction & Accessibility

**React Aria primitive choice — GridList vs ListBox:**

Use **`GridList`** (from `react-aria-components`) rather than `ListBox`.

Justification:
- `ListBox` maps to `role="listbox"` with `role="option"` children — semantically correct only when every item is a selectable option (e.g. a dropdown). Most M3 list items are interactive rows with complex content (icons, trailing actions, switches) that are not purely "options".
- `GridList` maps to `role="grid"` with `role="row"` children and supports interactive cells within a row (trailing checkbox, switch, trailing icon button) — correct semantic for complex interactive lists.
- When the list is purely non-interactive or read-only (no `onAction`/`selectionMode`), render a plain `<ul>` with `<li>` — no RAC wrapper needed; screen readers announce a plain list.
- When items are purely selectable with no sub-actions, `ListBox` is acceptable and semantically lighter; expose via `asListBox` prop for opt-in.

**ARIA:**
- Interactive list: `role="grid"` on `.list`, `role="row"` on `.list-item`, `role="gridcell"` on `.list-item__content` and `.list-item__trailing`
- Non-interactive list: `<ul>`/`<li>` — implicit `role="list"`/`"listitem"`
- `aria-selected` on selected rows
- `aria-disabled` on disabled rows
- `aria-label` or `aria-labelledby` required on the grid container (enforce in dev mode)
- Trailing interactive elements (checkbox, switch, icon button) are focusable cells within the row

**Keyboard navigation:**
- `Arrow Up`/`Arrow Down`: move focus between rows (GridList handles)
- `Space`: toggle selection of focused row
- `Enter`: trigger primary action (`onAction`) on focused row
- `Tab`: moves into/out of the list; within a row, Tab navigates between interactive cells (trailing action)
- `Home`/`End`: jump to first/last row

**Focus:**
- Item focus ring on the row boundary; visible `outline` using `--outline`
- Trailing interactive elements have their own focus rings within the row

**RTL:** Leading and trailing slots swap sides via `dir` inheritance. All CSS uses logical properties. Icon transforms (chevrons) flip via `:dir(rtl) .list-item__trailing { transform: scaleX(-1) }`.

**Reduced motion:** No enter animations on list items by default. Reorder drag animations skipped under `prefers-reduced-motion`.

---

## CSS Architecture

```css
@layer kafui {
  .list {
    /* Component-scoped vars */
    --item-h: 56px;
    --pad-inline: 16px;
    --leading-icon-size: 24px;
    --state-color: var(--on-surface);

    list-style: none;
    margin: 0;
    padding: 0;
    display: flex;
    flex-direction: column;
  }

  .list__subheader {
    font-size: var(--label-large-size);
    font-weight: var(--label-large-weight);
    color: var(--on-surface-variant);
    padding-inline: var(--pad-inline);
    padding-block: 8px;
    position: sticky;
    top: 0;
    background: inherit;
    pointer-events: none;
  }

  .list-item {
    position: relative;
    display: flex;
    align-items: center;
    gap: 16px;
    min-height: var(--item-h);
    padding-inline: var(--pad-inline);
    overflow: hidden;
  }

  .list-item__state-layer {
    position: absolute;
    inset: 0;
    pointer-events: none;
    background: var(--state-color);
    opacity: 0;
    transition: opacity var(--duration-short3) var(--easing-standard);
  }

  .list-item[data-hovered]  .list-item__state-layer { opacity: var(--state-hover); }
  .list-item[data-focused]  .list-item__state-layer { opacity: var(--state-focus); }
  .list-item[data-pressed]  .list-item__state-layer { opacity: var(--state-pressed); }

  /* Lines variants */
  .list-item--lines-2 { min-height: 72px; }
  .list-item--lines-3 {
    min-height: 88px;
    align-items: flex-start;
    padding-block: 12px;
  }

  /* Selected state */
  .list-item[data-selected] {
    --state-color: var(--on-secondary-container);
    background: var(--secondary-container);
  }
  .list-item[data-selected] .list-item__headline,
  .list-item[data-selected] .list-item__leading { color: var(--on-secondary-container); }

  /* Disabled */
  .list-item[data-disabled] {
    opacity: 0.38;
    pointer-events: none;
    cursor: default;
  }
  .list-item[data-disabled] .list-item__state-layer { display: none; }

  /* Leading slot */
  .list-item__leading {
    flex-shrink: 0;
    width: 40px;
    display: flex;
    align-items: center;
    justify-content: center;
    color: var(--on-surface-variant);
  }
  .list-item--leading-image  .list-item__leading { width: 56px; height: 56px; }
  .list-item--leading-video  .list-item__leading { width: 100px; height: 56px; }

  /* Content block */
  .list-item__content {
    flex: 1;
    min-width: 0;
    display: flex;
    flex-direction: column;
    justify-content: center;
  }

  .list-item__headline {
    font-size: var(--body-large-size);
    font-weight: var(--body-large-weight);
    line-height: var(--body-large-line-height);
    color: var(--on-surface);
    white-space: nowrap;
    overflow: hidden;
    text-overflow: ellipsis;
  }

  .list-item__supporting {
    font-size: var(--body-medium-size);
    font-weight: var(--body-medium-weight);
    color: var(--on-surface-variant);
  }
  .list-item--lines-3 .list-item__supporting {
    display: -webkit-box;
    -webkit-line-clamp: 2;
    -webkit-box-orient: vertical;
    overflow: hidden;
  }

  .list-item__overline {
    font-size: var(--label-small-size);
    font-weight: var(--label-small-weight);
    color: var(--on-surface-variant);
  }

  /* Trailing slot */
  .list-item__trailing {
    flex-shrink: 0;
    color: var(--on-surface-variant);
    font-size: var(--label-small-size);
  }

  /* Dragged */
  .list-item--dragged .list-item__state-layer { opacity: var(--state-dragged); }
  .list-item--dragged { box-shadow: var(--elevation-1); }

  @media (prefers-reduced-motion: reduce) {
    .list-item__state-layer { transition: none; }
  }
}
```

---

## Proposed kafUI React API

```tsx
// React Aria primitives:
//   Interactive: GridList, GridListItem (react-aria-components)
//   Non-interactive: plain <ul>/<li>

type ListLines = 1 | 2 | 3;
type ListSelectionMode = 'none' | 'single' | 'multiple'; // maps to RAC SelectionMode

interface ListProps {
  selectionMode?: ListSelectionMode;  // default: 'none'
  selectedKeys?: Iterable<Key>;       // controlled selection
  defaultSelectedKeys?: Iterable<Key>;
  onSelectionChange?: (keys: Selection) => void;
  onAction?: (key: Key) => void;      // row primary tap
  disabledKeys?: Iterable<Key>;
  divided?: boolean;                  // inject <Divider> between items; default false
  subheader?: string;                 // rendered as .list__subheader
  'aria-label'?: string;
  'aria-labelledby'?: string;
  children: React.ReactNode;
  className?: string;
}

interface ListItemProps {
  id: Key;                            // required for RAC GridListItem
  lines?: ListLines;                  // default: 1
  leading?: React.ReactNode;         // icon, avatar, image
  trailing?: React.ReactNode;        // icon, text, checkbox, switch
  overline?: string;
  supportingText?: string;
  isDisabled?: boolean;
  onPress?: (e: PressEvent) => void; // primary action; RAC handles keyboard
  children: React.ReactNode;         // headline text
  className?: string;
}

// Compound pattern:
<List selectionMode="single" onAction={(key) => navigate(key)}>
  <List.Item id="inbox" leading={<Icon name="inbox" />} trailing={<Icon name="chevron_right" />}>
    Inbox
  </List.Item>
  <List.Item id="sent" lines={2} leading={<Icon name="send" />} supportingText="3 items">
    Sent
  </List.Item>
</List>
```

**BEM classes emitted on `List.Item`:**
- `.list-item` (always)
- `.list-item--lines-{1|2|3}`
- `.list-item--leading-icon` / `--leading-avatar` / `--leading-image` / `--leading-video` (auto-detected from `leading` element type)
- `.list-item--interactive` when `onPress` or `selectionMode !== 'none'`

**RAC data attributes** (`data-hovered`, `data-focused`, `data-pressed`, `data-selected`, `data-disabled`) drive state-layer CSS — no className toggling in JS.

**Justifications vs MUI:**
- No `ListItemButton` separate component — M3 doesn't have a separate button variant; interactivity is driven by `onPress` / `selectionMode`, not a wrapper component.
- No `dense` prop — M3 doesn't spec a dense list; density is controlled by the `lines` prop.
- `leading`/`trailing` slots replace MUI's `ListItemAvatar`/`ListItemIcon`/`ListItemSecondaryAction` — flatter, 2-slot API.
- GridList over ListBox is a semantic correctness decision absent from MUI.
- State-layer is a DOM child element (`.list-item__state-layer`) rather than `::before` to support independently interactive cells (trailing checkbox, switch) without z-index conflicts. This differs from chip and menu which use `::before` because those have no interior interactive children.
