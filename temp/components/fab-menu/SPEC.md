# FAB Menu ✦ M3 Expressive

## Purpose & M3 Category

FAB Menu is an **M3 Expressive (2025) Actions** component. It is a Floating Action Button that,
on press, **expands into a vertical stack of labeled action items** positioned above the FAB.
The FAB morphs during expansion: its border-radius broadens and its icon cross-fades to a
close/dismiss icon. Each action item slides up and fades in with staggered motion. On dismissal,
the stack collapses in reverse stagger and the FAB returns to its resting shape.

FAB Menu is used when a surface has multiple important persistent-access actions that do not
warrant a full bottom navigation bar.

> **Expand direction is vertical upward only** per M3 spec. Do not add a `direction` prop —
> supporting four directions (like MUI SpeedDial) would be a spec deviation. Position the FAB
> at bottom-inline-end; items stack above it.

---

## Anatomy / Parts

```
[FabMenu]   (position context)
  ├── .fab-menu__menu            (overlay panel; above FAB; role="menu")
  │     └── .fab-menu__item × n  (RAC Button with role="menuitem")
  │           ├── .fab-menu__item-label      (visible text label)
  │           ├── .fab-menu__item-pill       (circular icon container)
  │           │     └── .fab-menu__item-icon
  │           └── .state-layer
  └── .fab-menu__fab             (RAC Button; always visible)
        ├── .fab-menu__fab-close-icon    (icon when open; cross-fades)
        ├── .fab-menu__fab-open-icon     (icon when closed; cross-fades)
        ├── .fab-menu__fab-label         (Extended FAB text; hidden when open)
        └── .state-layer
```

| Part | BEM class | Notes |
|---|---|---|
| Root | `.fab-menu` | Position context; `position: relative` or `fixed` by consumer |
| Menu panel | `.fab-menu__menu` | `role="menu"`; above FAB; `pointer-events: none` when closed |
| Action item | `.fab-menu__item` | RAC `Button`; `role="menuitem"` |
| Item label | `.fab-menu__item-label` | `label-large`; always visible when menu open |
| Item icon pill | `.fab-menu__item-pill` | Circular container; 56dp; interactive hit target |
| Item icon | `.fab-menu__item-icon` | 24dp sprite |
| Item state layer | `.state-layer` | On `__item-pill` |
| FAB button | `.fab-menu__fab` | RAC `Button` — trigger |
| Close icon | `.fab-menu__fab-close-icon` | Shown when open; cross-fades via opacity |
| Open icon | `.fab-menu__fab-open-icon` | Shown when closed; cross-fades via opacity |
| FAB label | `.fab-menu__fab-label` | Extended FAB variant only; hidden when open |
| FAB state layer | `.state-layer` | On FAB |

Modifiers on `.fab-menu`:
- `[data-open]` — driven by React state; CSS uses this for ALL open-state styling
- `--size-small` / `--size-medium` / `--size-large` — FAB size
- `--variant-primary` / `--variant-secondary` / `--variant-tertiary` / `--variant-surface`
- `--extended` — Extended FAB at rest
- `--label-start` — item labels at inline-start (left in LTR); default is inline-end

> **`[data-open]` not `.--open` modifier class** — the open state is dynamic; data attributes are
> idiomatic for dynamic component state in RAC-based components. All CSS animations and morph
> transitions key off `[data-open]`.

---

## Variants

### FAB size

| Size | `fabSize` | FAB diameter | Icon size |
|---|---|---|---|
| `"small"` | 40dp | 24dp | (transparent padding to 48dp touch target) |
| `"medium"` (default) | 56dp | 24dp | |
| `"large"` | 96dp | 36dp | |

### FAB color variant

| Variant | `fabVariant` | Container | Icon |
|---|---|---|---|
| `"primary"` (default) | `--primary-container` | `--on-primary-container` |
| `"secondary"` | `--secondary-container` | `--on-secondary-container` |
| `"tertiary"` | `--tertiary-container` | `--on-tertiary-container` |
| `"surface"` | `--surface-container-high` | `--primary` |

### Extended FAB at rest

When `extended={true}`, the FAB resting state is an Extended FAB (pill shape with icon + label).
On open, it morphs to a square-ish shape housing only the close icon. Requires `fabLabel`.

### Label side

`labelSide="end"` (default): labels appear to the inline-end of the icon pill (right in LTR).
`labelSide="start"`: labels appear to the inline-start (left in LTR) — for FABs positioned at
the inline-start of the screen.

---

## States

### FAB trigger

| State | Treatment |
|---|---|
| Enabled (closed) | Elevation 3; per-variant container fill; open icon visible |
| Hover (closed) | Elevation 4; state layer 8% |
| Focus (closed) | Elevation 3; state layer 10%; focus ring |
| Pressed (closed) | Opens menu; state layer 10% |
| Enabled (open) | Morphed shape; close icon visible; elevation maintained |
| Hover (open) | State layer 8% |
| Pressed (open) | Closes menu |
| Disabled | 38% opacity; elevation 0 |

### Action items

| State | Treatment |
|---|---|
| Entering | Slides up (`translateY(16px) → 0`) + fade in; staggered by item index |
| Visible | Pill background `--surface-container`; icon `--on-surface` |
| Hover | State layer 8% on pill |
| Focus | State layer 10% on pill; 3dp focus ring on pill |
| Pressed | State layer 10%; fires action; menu closes |
| Exiting | Slides down (reverse stagger); fade out |
| Disabled | 38% opacity; `pointer-events: none` |

### Open/close motion

**Open sequence:**
1. FAB border-radius morphs (300ms `emphasized-decelerate`): `--corner-large` → `--corner-extra-large`.
2. Close icon fades in; open icon fades out (200ms, starts at 100ms offset).
3. Menu panel becomes visible.
4. Items slide up + fade in with stagger: each item `animation-delay: calc(var(--_i) * 40ms)`.

**Close sequence:**
1. Items slide down + fade out in reverse stagger (outermost first, closest to FAB last).
2. Menu panel hidden after last item exits.
3. FAB morphs back to `--corner-large`; icon cross-fades back.

All transitions use M3 motion tokens. Item stagger via CSS `animation-delay` keyed on
`--_i` (CSS custom property set by React as inline style on each item).

```css
@layer kafui {
  /* FAB morph */
  .fab-menu__fab {
    border-radius: var(--corner-large);
    transition: border-radius var(--duration-medium4, 400ms) var(--easing-emphasized-decelerate);
  }
  .fab-menu[data-open] .fab-menu__fab {
    border-radius: var(--corner-extra-large);
    transition-timing-function: var(--easing-emphasized-decelerate);
  }

  /* Icon cross-fade */
  .fab-menu__fab-open-icon  { opacity: 1; transition: opacity 200ms; }
  .fab-menu__fab-close-icon { opacity: 0; transition: opacity 200ms; position: absolute; }
  .fab-menu[data-open] .fab-menu__fab-open-icon  { opacity: 0; }
  .fab-menu[data-open] .fab-menu__fab-close-icon { opacity: 1; }

  /* Item entrance */
  @keyframes fab-item-in {
    from { opacity: 0; transform: translateY(16px); }
    to   { opacity: 1; transform: translateY(0); }
  }
  @keyframes fab-item-out {
    from { opacity: 1; transform: translateY(0); }
    to   { opacity: 0; transform: translateY(16px); }
  }

  .fab-menu[data-open] .fab-menu__item {
    animation: fab-item-in 200ms var(--easing-emphasized-decelerate) both;
    animation-delay: calc(var(--_i) * 40ms);
  }
  .fab-menu[data-closing] .fab-menu__item {
    animation: fab-item-out 200ms var(--easing-emphasized-accelerate) both;
    /* reverse stagger: items closer to FAB exit first */
    animation-delay: calc(var(--_i-reverse) * 40ms);
  }

  @media (prefers-reduced-motion: reduce) {
    .fab-menu__fab,
    .fab-menu__fab-open-icon,
    .fab-menu__fab-close-icon,
    .fab-menu__item { animation: none; transition: none; }
  }
}
```

> **`data-closing`**: React adds this attribute briefly when closing (duration = total close
> animation: `n_items × 40ms + 200ms`), then removes it. This enables reverse-stagger animations
> without JS per-item style manipulation.

---

## Design Tokens

```css
@layer kafui {
  .fab-menu {
    /* component-local */
    --_fab-container: var(--primary-container);
    --_fab-icon-color: var(--on-primary-container);
    --_fab-size: 56px;
    --_icon-size: 24px;

    display: inline-flex;
    flex-direction: column;
    align-items: flex-end;   /* default: label-end layout */
    gap: 8px;
  }

  /* Variant token maps */
  .fab-menu--variant-primary   { --_fab-container: var(--primary-container);   --_fab-icon-color: var(--on-primary-container); }
  .fab-menu--variant-secondary { --_fab-container: var(--secondary-container); --_fab-icon-color: var(--on-secondary-container); }
  .fab-menu--variant-tertiary  { --_fab-container: var(--tertiary-container);  --_fab-icon-color: var(--on-tertiary-container); }
  .fab-menu--variant-surface   { --_fab-container: var(--surface-container-high); --_fab-icon-color: var(--primary); }

  /* Size */
  .fab-menu--size-small  { --_fab-size: 40px;  --_icon-size: 24px; }
  .fab-menu--size-medium { --_fab-size: 56px;  --_icon-size: 24px; }
  .fab-menu--size-large  { --_fab-size: 96px;  --_icon-size: 36px; }

  /* Label side */
  .fab-menu--label-start { align-items: flex-start; }
}
```

| Role | Usage |
|---|---|
| `--primary-container` | FAB fill (primary variant) |
| `--on-primary-container` | FAB icon (primary variant) |
| `--secondary-container` | Secondary variant fill |
| `--on-secondary-container` | Secondary icon |
| `--tertiary-container` | Tertiary variant fill |
| `--on-tertiary-container` | Tertiary icon |
| `--surface-container-high` | Surface variant fill |
| `--primary` | Surface variant icon |
| `--surface-container` | Item icon pill background |
| `--on-surface` | Item icon color |
| `--corner-large` | FAB resting border-radius (16dp) |
| `--corner-extra-large` | FAB open border-radius (28dp) |
| `--corner-full` | Item icon pill (circle) |
| `--elevation-3` | FAB resting shadow |
| `--elevation-4` | FAB hover shadow |
| `--easing-emphasized-decelerate` | Open morph and item entrance |
| `--easing-emphasized-accelerate` | Close morph and item exit |
| `--duration-medium4` | 400ms — FAB morph |
| `--label-large-*` | Item label typography |
| `--state-hover` | 8% |
| `--state-focus` | 10% |
| `--state-pressed` | 10% |

---

## Interaction & Accessibility

### Keyboard

| Key | Behavior |
|---|---|
| `Tab` | Focus reaches FAB from document tab order |
| `Space` / `Enter` | Closed FAB: opens menu; focus moves to first item. Open FAB: closes menu; focus returns to FAB. |
| `Escape` | Closes menu; focus returns to FAB |
| `ArrowUp` | Open menu: move to previous item (wraps to last) |
| `ArrowDown` | Open menu: move to next item (wraps to first) |
| `Home` | Focus first item |
| `End` | Focus last item |
| `Space` / `Enter` | Focused item: fires action; closes menu; focus returns to FAB |
| `Tab` | Open menu: closes menu; moves focus out of component |

### ARIA

- FAB trigger: `role="button"`, `aria-haspopup="menu"`, `aria-expanded` (true/false),
  `aria-controls={menuId}`, `aria-label` **required** (e.g. `"Create actions"`).
- Menu panel: `role="menu"`, `id={menuId}`, `aria-label` = FAB `aria-label`.
- Items: `role="menuitem"`, accessible name from visible `__item-label` text (no additional
  `aria-label` needed when label is always visible).
- Disabled items: `aria-disabled="true"`.

> **Items always show their label** — unlike MUI SpeedDial's tooltip-on-hover approach. This means
> items are always accessible on touch devices (no hover step required) and no additional
> `aria-label` is needed per item.

### Focus Management

On open: after animation starts, move focus to the first non-disabled `.fab-menu__item`.
On close (Escape / item activation): return focus to `.fab-menu__fab`.

Use RAC `FocusScope` wrapping the menu panel for focus trapping while open (non-modal: Tab still
exits and closes the menu).

### Outside-Click Dismiss

Use `useInteractOutside` (from `@react-aria/interactions`) on the root element. This is already
a dependency via RAC; no additional package needed.

### Scroll Dismiss

`useEffect` adds a `scroll` event listener on `window` when open (if `closeOnScroll={true}`).
Remove on cleanup. Default: `true`.

### Touch / Hit Area

- Medium FAB (56dp): exceeds 48dp minimum.
- Small FAB (40dp): `min-block-size: 48px` + `padding-block` extends touch target.
- Item pills: 56dp diameter — exceeds 48dp minimum.

### Screen Reader Announcements

On open: focus moving to first item causes SR to read item label + "menu" context via
`role="menu"` / `role="menuitem"`.
On close via item: item label announced at press; focus returns to FAB.

### Reduced Motion

`@media (prefers-reduced-motion: reduce)`: `animation: none; transition: none` on FAB, both
icons, and all items. Menu appears/disappears via `visibility` toggle only (set via
`pointer-events: none` → `auto` + `visibility: hidden` → `visible` driven by `[data-open]`).

### RTL

`labelSide="end"` in RTL: `end` resolves to `inline-end` = left in RTL via CSS logical property
`align-items: flex-end` within `[dir="rtl"]` (or directly use `align-items: flex-end` on the
logical flex axis). Prefer `align-items: flex-end` with `align-items: flex-start` for
`--label-start` rather than physical `left`/`right`.

---

## Proposed kafUI React API

```tsx
import { FabMenu } from "@kafui/react";

<FabMenu
  fabVariant="primary"
  fabSize="medium"
  icon={<Icon name="add" />}
  openIcon={<Icon name="close" />}  // optional; defaults to <Icon name="close" />
  aria-label="Create actions"
  labelSide="end"
>
  <FabMenu.Item id="photo"   icon={<Icon name="photo_camera" />} label="Take photo"    onPress={handlePhoto} />
  <FabMenu.Item id="upload"  icon={<Icon name="upload" />}       label="Upload file"   onPress={handleUpload} />
  <FabMenu.Item id="compose" icon={<Icon name="edit" />}         label="Compose"       onPress={handleCompose} isDisabled />
</FabMenu>
```

### API Rationale

- **`FabMenu.Item` compound sub-component** — keeps imports minimal; collocates item type with
  parent.
- **`label` as a required string prop on `FabMenu.Item`** (not `children`) — items always need a
  visible text label (M3 spec requirement; also the accessible name source). `children` would be
  ambiguous. Using a named prop makes the requirement explicit and TypeScript-enforced.
- **`icon` as ReactNode on `FabMenu.Item`** — component-agnostic; works with any icon system.
- **`openIcon` optional** — defaults to `<Icon name="close" />` internally. Consumer can override
  (e.g. for a custom "X" or animated icon).
- **`extended` + `fabLabel`** — when `extended={true}`, `fabLabel` is required (TypeScript
  discriminated union). The pair is semantically linked.
- **No `direction` prop** — vertical-upward is the only M3-spec direction. A `direction` prop
  would invite spec deviation. Document this clearly.
- **`closeOnScroll` prop** — default `true` (M3 best practice: floating UI should not persist
  over scrolled content).

### Type Signatures

```ts
type FabMenuVariant = "primary" | "secondary" | "tertiary" | "surface";
type FabMenuSize    = "small" | "medium" | "large";
type LabelSide      = "start" | "end";

// Discriminated union for extended vs non-extended
type FabMenuProps =
  | {
      extended:    true;
      fabLabel:    string;      // required when extended
      fabVariant?: FabMenuVariant;
      fabSize?:    FabMenuSize;
      icon:        ReactNode;
      openIcon?:   ReactNode;
      labelSide?:  LabelSide;
      closeOnScroll?: boolean;
      isOpen?:     boolean;
      defaultOpen?: boolean;
      onOpenChange?: (open: boolean) => void;
      isDisabled?: boolean;
      "aria-label": string;
      className?:  string;
      children:    ReactNode;
    }
  | {
      extended?:   false;
      fabLabel?:   never;
      fabVariant?: FabMenuVariant;
      fabSize?:    FabMenuSize;
      icon:        ReactNode;
      openIcon?:   ReactNode;
      labelSide?:  LabelSide;
      closeOnScroll?: boolean;
      isOpen?:     boolean;
      defaultOpen?: boolean;
      onOpenChange?: (open: boolean) => void;
      isDisabled?: boolean;
      "aria-label": string;
      className?:  string;
      children:    ReactNode;
    };

interface FabMenuItemProps {
  id:          Key;
  icon:        ReactNode;
  label:       string;          // required; visible text; accessible name
  onPress?:    (e: PressEvent) => void;
  isDisabled?: boolean;
  className?:  string;
}
```

### React Aria Primitive

```
FabMenu:
  Root <div [data-open] [data-closing]>
    Menu panel <div role="menu" [aria-label]>
      FocusScope (traps focus while open; non-modal)
        FabMenu.Item × n → RAC Button role="menuitem"
    FAB <Button aria-haspopup="menu" aria-expanded aria-controls>
```

**No `MenuTrigger`** — `FabMenu` is not a positional popover; items are absolutely positioned
within the component's own stacking context (or `position: fixed` by consumer). Using RAC
`MenuTrigger` + `Popover` would add unnecessary indirection and fight the stacking-context model.
Instead: `useState`/`useControlledState` for open state; `FocusScope` for focus trapping;
`useInteractOutside` for outside-click dismiss.

### BEM Classes Emitted

```
.fab-menu
.fab-menu--size-{small|medium|large}
.fab-menu--variant-{primary|secondary|tertiary|surface}
.fab-menu--extended
.fab-menu--label-start    (default is --label-end, no class needed)

.fab-menu__menu           (data-open / data-closing on root drives CSS)
.fab-menu__item
.fab-menu__item-label
.fab-menu__item-pill
.fab-menu__item-icon
.fab-menu__fab
.fab-menu__fab-open-icon
.fab-menu__fab-close-icon
.fab-menu__fab-label
.state-layer              (shared utility; on FAB and each item pill)
```

Inline styles emitted per item: `style="--_i: {n}; --_i-reverse: {m}"` for stagger. This is
the only acceptable use of inline styles — CSS `animation-delay` cannot be authored statically.

All block/modifier classes inside `@layer kafui { … }`. No `kafui-` prefix.

---

## Open Questions

1. **`MenuTrigger` vs custom open state**: The current design uses custom open state + `FocusScope`
   rather than RAC `MenuTrigger` + `Popover`. Rationale: items are stacked in-flow (not positioned
   in a portal), and the morph/stagger animation model requires the items to live in the same DOM
   subtree as the FAB. Confirm this is the right call before implementation. If items need to
   escape the stacking context (e.g., when FAB is inside `overflow: hidden`), a portal approach
   with RAC `Popover` would be required.
2. **`data-closing` timing**: The close animation total duration = `(n_items × 40ms) + 200ms`. The
   `data-closing` attribute must be removed exactly at this duration to avoid a flash of invisible
   items. Recommend `useEffect` with `setTimeout` keyed on item count. Alternatively, listen to
   `animationend` on the first item (lowest `animation-delay`). Decide and document.
3. **Extended FAB morph**: When `extended={true}`, the resting shape is a wide pill. On open, it
   must transition to a compact square/circular shape hosting only the close icon. This requires a
   `width` or `inline-size` transition in addition to `border-radius`. Confirm the exact M3
   Expressive spec dimensions for this morph and whether `aspect-ratio: 1` or fixed width is used
   for the open state.
4. **Item pill size**: The spec says 56dp. This means the touch target already exceeds 48dp.
   Confirm whether M3 specifies 56dp for all FAB sizes or scales with `fabSize`.
5. **`aria-label` on items**: Labels are always visible text, so no additional `aria-label` is
   needed per item (the label text serves as the accessible name). However, if a consumer were to
   set a very long `label`, consider a `aria-label` override prop. Leave for v2.
