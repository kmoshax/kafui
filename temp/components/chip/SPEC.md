# Chip

M3 category: **Action / Selection** (cross-cutting — chips trigger actions, filter content, and represent input tokens). M3 defines four distinct chip types that differ in purpose, structure, and interaction model. All share the same compact pill shape.

---

## Anatomy / Parts

All chip types share a common container structure; icons and trailing elements are type-specific.

```
[chip-root]
  ├─ [state-layer]
  ├─ [leading-icon]?        — assist: optional icon; filter: checkmark when selected; input: avatar/icon
  ├─ [label]
  └─ [trailing-icon]?       — input chip: remove ✕; suggestion: none; filter: none; assist: none
```

| Part | BEM element |
|---|---|
| Root (button/div) | `.chip` |
| State-layer | `.chip__state-layer` |
| Leading icon | `.chip__leading-icon` |
| Label | `.chip__label` |
| Trailing remove icon | `.chip__trailing-icon` |
| Chip set container | `.chip-set` |

BEM modifiers on `.chip`:
- `--assist`, `--filter`, `--input`, `--suggestion` (type)
- `--selected` (filter chips when active)
- `--elevated` (assist + suggestion can optionally be elevated)
- `--disabled`

BEM modifiers on `.chip-set`:
- `--wrap` (default: wraps), `--scroll` (single-row horizontal scroll)

---

## Variants

### 1. Assist Chip
- Triggers an action relevant to primary content on screen (e.g., "Share", "Add to cart").
- No toggle/select; stateless from selection perspective.
- Optional leading icon; no trailing icon.
- Two surfaces: **flat** (outlined, `surface-variant` fill) and **elevated** (`surface` fill + elevation-1 shadow).

### 2. Filter Chip
- Toggleable; represents a filter category. Selected state shows checkmark as leading icon.
- Can appear in a group (multi-select allowed).
- Optional leading icon in unselected state (replaced by checkmark when selected).
- Elevated variant available.

### 3. Input Chip
- Represents a discrete piece of user input (e.g., a tag, email address, search token).
- Removable via trailing ✕ icon button.
- Optional leading avatar/icon.
- Lives inside a chip set that manages the set of tokens.
- **Not toggleable**; tapping the label may open an edit affordance (app-defined).

### 4. Suggestion Chip
- Presents AI/dynamic suggestions for user to accept (e.g., suggested replies).
- Stateless (no toggle); triggers action on press.
- No leading icon; no trailing icon.
- Elevated variant available.

---

## States

All chip types:

| State | Token applied |
|---|---|
| Hover | State-layer 8% `on-surface-variant` (unselected) / `on-secondary-container` (selected) |
| Focus | State-layer 10%; `outline` focus ring |
| Pressed | State-layer 10% + ripple |
| Disabled | Container + label at 38% opacity; `on-surface` outline; no state layers |
| Selected (filter) | Fill: `secondary-container`; label+icon: `on-secondary-container`; checkmark appears |
| Dragged | State-layer 16%; elevation-4 shadow |

Elevated variants use `--elevation-1` at rest, `--elevation-2` on hover, `--elevation-0` on disabled.

---

## Design Tokens

### Color

| Role | Token |
|---|---|
| Unselected container (flat) | transparent / `--surface` border: `--outline` |
| Unselected container (elevated) | `--surface-container-low` |
| Selected container (filter) | `--secondary-container` |
| Unselected label/icon | `--on-surface-variant` |
| Selected label/icon (filter) | `--on-secondary-container` |
| Disabled (all) | `--on-surface` @ 38% |
| State layer (unselected) | `--on-surface-variant` |
| State layer (selected) | `--on-secondary-container` |

### Shape
| Token | Value |
|---|---|
| Chip container | `--corner-full` (fully rounded pill) |

M3 spec: chip uses `shape.corner.full`. Use `border-radius: var(--corner-full)`.

### Typography
| Role | Token |
|---|---|
| Label | `--label-large-size` / `--label-large-weight` / `--label-large-line-height` |

### Elevation
| State | Token |
|---|---|
| Elevated at rest | `--elevation-1` |
| Elevated hover | `--elevation-2` |
| Flat at rest | `--elevation-0` |

### Motion
| Token | Usage |
|---|---|
| `--duration-short3` | State-layer transition |
| `--easing-standard` | Checkmark fade-in on filter select |
| `--duration-short2` | Chip remove exit animation (scale + fade) |

---

## Interaction & Accessibility

### React Aria Primitive Decision per Type

| Chip type | Primitive | Justification |
|---|---|---|
| **Assist** | `Button` (RAC) | Pure action; no toggle state; `role="button"` |
| **Filter** | `ToggleButton` (RAC) within `ToggleButtonGroup` | Toggle semantics (`aria-pressed`); `ToggleButtonGroup` manages single/multi-select; avoids `role="option"` which implies listbox ownership. `TagGroup` would give `role="row"/"gridcell"` which is wrong for filter chips — they are not "tags" (removable tokens). |
| **Input** | `TagGroup` / `Tag` (RAC) | Input chips are removable tokens — exactly the semantic of `role="row"/"gridcell"` within a grid, with built-in `allowsRemoving` + `onRemove`. The remove ✕ button is wired automatically. |
| **Suggestion** | `Button` (RAC) | Same as assist — pure action, no persisted state. |

**Filter chip group — ToggleButtonGroup vs TagGroup justification:**
`TagGroup` (`role="grid"`) is for collections of removable items (input chips). `ToggleButtonGroup` gives each chip a `role="button"` with `aria-pressed`, which is the correct semantic for filter toggle actions. Screen reader announcement: "Cheese toggle button, pressed" vs. "Cheese, row 1 of 3, grid" — the former is clearer for filter chips.

### ARIA
- Assist/Suggestion chips: `role="button"`, `aria-label` if icon-only.
- Filter chips: `role="button"` + `aria-pressed="true|false"` (from `ToggleButton`).
- Input chip set: `role="grid"` (from `TagGroup`); each chip `role="row"` > `role="gridcell"`; remove button `role="button"` `aria-label="Remove {label}"`.
- Chip set has `aria-label` or `aria-labelledby`.

### Keyboard Navigation
| Key | Assist/Suggestion | Filter (ToggleButtonGroup) | Input (TagGroup) |
|---|---|---|---|
| `Tab` | Focus chip | Focus first chip in group | Focus chip set |
| `Space`/`Enter` | Trigger action | Toggle selected | Activate (open edit) |
| `Arrow ←/→` | — | Move between chips in group | Move between chips |
| `Delete`/`Backspace` | — | — | Remove focused chip |
| `Tab` on remove icon | — | — | Focus remove icon within chip |

### RTL
- Leading icon `margin-inline-end`; trailing icon `margin-inline-start` — auto-flip in RTL.
- Chip set wraps naturally with logical layout.

### Reduced Motion
```css
@layer kafui {
  @media (prefers-reduced-motion: reduce) {
    .chip__state-layer { transition: none; }
    .chip--removing { animation: none; opacity: 0; }
  }
}
```

---

## CSS Architecture

All chip styles live inside `@layer kafui { … }` to avoid specificity collisions.

```css
@layer kafui {
  .chip {
    /* Component-scoped design tokens */
    --radius: var(--corner-full);
    --h: 32px;
    --pad-inline: 16px;
    --icon-size: 18px;
    --state-color: var(--on-surface-variant);

    position: relative;
    display: inline-flex;
    align-items: center;
    gap: 8px;
    height: var(--h);
    padding-inline: var(--pad-inline);
    border-radius: var(--radius);
    font-size: var(--label-large-size);
    font-weight: var(--label-large-weight);
    line-height: var(--label-large-line-height);
    color: var(--on-surface-variant);
    cursor: pointer;
    overflow: hidden;
  }

  /* State-layer via ::before pseudo */
  .chip::before {
    content: "";
    position: absolute;
    inset: 0;
    border-radius: inherit;
    background: var(--state-color);
    opacity: 0;
    transition: opacity var(--duration-short3) var(--easing-standard);
    pointer-events: none;
  }

  .chip[data-hovered]::before  { opacity: var(--state-hover); }
  .chip[data-focused]::before  { opacity: var(--state-focus); }
  .chip[data-pressed]::before  { opacity: var(--state-pressed); }

  /* Flat (outlined) — assist, filter, suggestion default */
  .chip--assist,
  .chip--filter,
  .chip--suggestion {
    background: transparent;
    outline: 1px solid var(--outline);
    outline-offset: -1px;
  }

  /* Elevated — assist + suggestion optional */
  .chip--elevated {
    background: var(--surface-container-low);
    outline: none;
    box-shadow: var(--elevation-1);
  }
  .chip--elevated[data-hovered] {
    box-shadow: var(--elevation-2);
  }

  /* Filter selected */
  .chip--filter.chip--selected {
    --state-color: var(--on-secondary-container);
    background: var(--secondary-container);
    outline: none;
    color: var(--on-secondary-container);
  }

  /* Input chip */
  .chip--input {
    background: transparent;
    outline: 1px solid var(--outline);
    outline-offset: -1px;
    --pad-inline: 12px; /* tighter because of trailing icon */
  }

  /* Disabled — all types */
  .chip[data-disabled],
  .chip--disabled {
    opacity: 0.38;
    pointer-events: none;
    outline-color: var(--on-surface);
  }

  /* Leading/trailing icons */
  .chip__leading-icon,
  .chip__trailing-icon {
    width: var(--icon-size);
    height: var(--icon-size);
    flex-shrink: 0;
    display: flex;
    align-items: center;
    justify-content: center;
  }

  /* Trailing icon is an independent interactive button */
  .chip__trailing-icon {
    border-radius: var(--corner-full);
    margin-inline-start: -4px;
  }

  /* Chip set */
  .chip-set {
    display: flex;
    flex-wrap: wrap;
    gap: 8px;
    align-items: center;
  }
  .chip-set--scroll {
    flex-wrap: nowrap;
    overflow-x: auto;
    scrollbar-width: none;
  }
  .chip-set--scroll::-webkit-scrollbar { display: none; }

  /* Filter checkmark animation */
  .chip__leading-icon--checkmark {
    opacity: 0;
    transform: scale(0.5);
    transition:
      opacity var(--duration-short3) var(--easing-standard),
      transform var(--duration-short3) var(--easing-standard);
  }
  .chip--selected .chip__leading-icon--checkmark {
    opacity: 1;
    transform: scale(1);
  }

  /* Input chip remove animation */
  @keyframes chip-exit {
    from { transform: scale(1); opacity: 1; }
    to   { transform: scale(0); opacity: 0; }
  }
  .chip--removing {
    animation: chip-exit var(--duration-short2) var(--easing-standard) forwards;
  }

  @media (prefers-reduced-motion: reduce) {
    .chip::before { transition: none; }
    .chip__leading-icon--checkmark { transition: none; }
    .chip--removing { animation: none; opacity: 0; }
  }
}
```

---

## Proposed kafUI React API

```tsx
// ── Assist Chip ───────────────────────────────────────────────
interface AssistChipProps {
  /** Leading icon name (sprite) */
  icon?: string;
  elevated?: boolean;
  isDisabled?: boolean;
  onPress?: () => void;
  children: React.ReactNode; // label
}
// Renders RAC <Button> with className="chip chip--assist [chip--elevated]"

// ── Suggestion Chip ───────────────────────────────────────────
interface SuggestionChipProps {
  elevated?: boolean;
  isDisabled?: boolean;
  onPress?: () => void;
  children: React.ReactNode;
}
// Renders RAC <Button> with className="chip chip--suggestion [chip--elevated]"

// ── Filter Chip + Group ───────────────────────────────────────
interface FilterChipProps {
  value: string; // identity within group
  icon?: string; // shown when unselected; replaced by checkmark when selected
  isDisabled?: boolean;
  elevated?: boolean;
  children: React.ReactNode;
}

interface FilterChipGroupProps {
  /** "single" = radio-like; "multiple" = multi-select. Default: "multiple" */
  selectionMode?: "single" | "multiple";
  selectedKeys?: Set<string>;
  defaultSelectedKeys?: Set<string>;
  onSelectionChange?: (keys: Set<string>) => void;
  isDisabled?: boolean;
  "aria-label": string;
  children: React.ReactNode; // FilterChip elements
}
// Renders RAC <ToggleButtonGroup> (selectionMode mapped to RAC prop)
// Each FilterChip renders RAC <ToggleButton id={value}>

// ── Input Chip Set ────────────────────────────────────────────
interface InputChipItem {
  id: string;
  label: string;
  icon?: string; // leading icon or avatar
}

interface InputChipSetProps {
  items: InputChipItem[];
  onRemove: (keys: Set<string>) => void;
  isDisabled?: boolean;
  /** "wrap" (default) | "scroll" */
  layout?: "wrap" | "scroll";
  "aria-label": string;
}
// Renders RAC <TagGroup allowsRemoving onRemove={onRemove}>
//   <TagList className="chip-set [chip-set--scroll]">
//     {items.map(item => (
//       <Tag key={item.id} className="chip chip--input">
//         {item.icon && <Icon name={item.icon} className="chip__leading-icon" />}
//         <span className="chip__label">{item.label}</span>
//         <Button slot="remove" className="chip__trailing-icon" aria-label={`Remove ${item.label}`}>
//           <Icon name="close" />
//         </Button>
//       </Tag>
//     ))}
//   </TagList>
// </TagGroup>

// ── Usage Examples ────────────────────────────────────────────
// Assist:
<AssistChip icon="directions" onPress={() => navigate()}>
  Get directions
</AssistChip>

// Filter group:
<FilterChipGroup
  aria-label="Filter by cuisine"
  selectionMode="multiple"
  selectedKeys={filters}
  onSelectionChange={setFilters}
>
  <FilterChip value="italian">Italian</FilterChip>
  <FilterChip value="thai">Thai</FilterChip>
</FilterChipGroup>

// Input chip set:
<InputChipSet
  aria-label="Recipients"
  items={recipients}
  onRemove={(keys) => removeRecipients(keys)}
/>
```

**Deviations / justifications:**
- `FilterChipGroup` uses RAC `ToggleButtonGroup` not `TagGroup` — see ARIA decision table above.
- `selectionMode` on `FilterChipGroup` maps directly to RAC `ToggleButtonGroup` `selectionMode`; M3 doesn't name this prop but both single and multi selection are spec-valid.
- Input chip remove label (`aria-label="Remove {label}"`) is auto-generated from the item label to avoid requiring consumers to wire it manually.
- Chip set `layout="scroll"` applies `overflow-x: auto; white-space: nowrap; flex-wrap: nowrap` via BEM modifier rather than injecting inline style.
- The `chip__state-layer` DOM element noted in the anatomy table is realized as `.chip::before` in CSS — no extra DOM node needed, reducing rendered tree depth vs. MUI's overlay `<span>`.
