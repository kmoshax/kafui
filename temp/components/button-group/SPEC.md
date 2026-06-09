# Button Group ✦ M3 Expressive

## Purpose & M3 Category

Button Group is an **M3 Expressive (2025) Actions** component. It arranges two or more related
buttons in a connected or loosely-spaced horizontal row. On press, individual buttons respond with
**shape morphing** — the pressed button's inner corners contract and adjacent siblings' touching
corners soften, communicating physical connection through motion. It operates in two modes:

- **Action mode** (`selectionMode="none"`): each button fires an independent action; no persistent
  selection state.
- **Toggle mode** (`selectionMode="single"|"multiple"`): one-of-N or multi-select persistent
  selection, with radio or checkbox semantics.

No direct MUI equivalent. MUI `ButtonGroup` is a bordered row with static appearance and no
selection model.

---

## Anatomy / Parts

```
[ButtonGroup]
  └── [ButtonGroupItem] × n
        ├── __leading-icon    (optional; 18dp)
        ├── __label           (required if no icon; or alongside icon)
        ├── __trailing-icon   (optional; 18dp)
        └── __state-layer     (::before pseudo)
```

| Part | BEM class | Notes |
|---|---|---|
| Root container | `.button-group` | `role="group"` or `role="radiogroup"` |
| Individual item | `.button-group__item` | RAC `Button` or `ToggleButton` |
| Leading icon | `.button-group__item__leading-icon` | 18dp sprite |
| Label | `.button-group__item__label` | `label-large` typescale |
| Trailing icon | `.button-group__item__trailing-icon` | 18dp sprite |
| State layer | `.state-layer` | Shared `::before` pseudo-element |

Modifiers on `.button-group`:
- `--connected` — items share group border; pill-end caps only at group edges; 1dp dividers inside
- `--standard` — items have individual full-pill shapes with a small gap (default)
- `--{variant}` — `filled | tonal | outlined | elevated | text` — group-level variant default
- `--size-{xs|sm|md|lg|xl}` — height and padding scale

Modifiers on `.button-group__item`:
- `[data-selected]` (RAC automatic) — item selected in toggle mode
- `[data-disabled]` (RAC automatic)
- `[data-pressed]` (RAC automatic) — drives shape-morph CSS
- `--{variant}` — per-item variant override
- `--first` / `--last` — explicit data attributes for connected corner targeting (supplement
  `:first-child`/`:last-child` in case of wrapper elements)

> **Prefer `data-*` over BEM modifier classes.** React Aria emits `data-pressed`, `data-selected`,
> `data-hovered`, `data-disabled`, `data-focused` automatically. Target these in CSS. Only add
> explicit BEM modifiers where no RAC equivalent exists.

---

## Variants

### Connection style

| Variant | Prop | Description |
|---|---|---|
| Standard | `connected={false}` (default) | Independent full-pill shapes with 4dp gap |
| Connected | `connected` | Shared outline; inner borders are 1dp dividers; pill ends only at group edges |

### Fill variant (on group or per-item)

| Value | Container | Content |
|---|---|---|
| `"filled"` | `--primary` | `--on-primary` |
| `"tonal"` | `--secondary-container` | `--on-secondary-container` |
| `"outlined"` | transparent + `--outline` border | `--primary` |
| `"elevated"` | `--surface-container-low` + shadow | `--primary` |
| `"text"` | transparent | `--primary` |

Default: `"outlined"`.

### Sizes (M3 Expressive 5-level scale)

| Size | `size` | Height | Inline padding |
|---|---|---|---|
| Extra-small | `"xs"` | 32dp | 12dp |
| Small | `"sm"` | 40dp | 16dp |
| Medium | `"md"` | 48dp | 24dp (default) |
| Large | `"lg"` | 56dp | 32dp |
| Extra-large | `"xl"` | 96dp | 48dp |

`xs` and `sm` use `min-block-size: 48px` with `padding-block` to meet the 48-dp touch target.

### Selection mode (toggle variant)

| `selectionMode` | Behavior | RAC primitive |
|---|---|---|
| `"none"` (default) | Independent action buttons | `Button` per item; plain `<div role="group">` |
| `"single"` | Radio-like | `ToggleButtonGroup` + `ToggleButton`; `role="radiogroup"` |
| `"multiple"` | Checkbox-like | `ToggleButtonGroup` + `ToggleButton`; `role="group"` |

---

## States

| State | Visual treatment |
|---|---|
| Enabled | Per-variant colors; full pill border-radius |
| Hover | State layer at `--state-hover` (8%) |
| Focus | State layer at `--state-focus` (10%); 3dp `outline` using `--secondary` |
| Pressed | State layer at `--state-pressed` (10%) + **shape morph** |
| Selected (toggle) | Same as variant's filled/tonal pressed-selected appearance |
| Disabled | 38% opacity; `pointer-events: none` |

### Shape Morph Details

M3 Expressive specifies that pressing a connected button contracts adjacent corners to communicate
physicality:

- **Pressed item** (`[data-pressed]`):
  - Outer corners: remain at `--corner-full` (pill).
  - Inner corners (touching neighbours): contract to `--corner-extra-small` (4dp).
- **Item immediately after pressed item** (`:has([data-pressed]) + .button-group__item`):
  - Leading (start) corners: soften to `--corner-small` (8dp).
  - Trailing corners: unchanged.
- **Item immediately before pressed item** (`:has(+ .button-group__item[data-pressed])`):
  - Trailing (end) corners: soften to `--corner-small` (8dp).

All four per-item corners are controlled via CSS custom properties set on the item:

```css
@layer kafui {
  .button-group__item {
    --_rs-ss: var(--corner-full);  /* border-start-start-radius */
    --_rs-se: var(--corner-full);  /* border-start-end-radius */
    --_rs-es: var(--corner-full);  /* border-end-start-radius */
    --_rs-ee: var(--corner-full);  /* border-end-end-radius */

    border-start-start-radius: var(--_rs-ss);
    border-start-end-radius:   var(--_rs-se);
    border-end-start-radius:   var(--_rs-es);
    border-end-end-radius:     var(--_rs-ee);

    transition: border-radius var(--duration-short4, 200ms) var(--easing-standard);
  }

  /* Pressed item: inner corners contract */
  .button-group--connected .button-group__item[data-pressed] {
    --_rs-se: var(--corner-extra-small);
    --_rs-ee: var(--corner-extra-small);
  }

  /* Item after pressed: start corners soften */
  .button-group--connected:has(.button-group__item[data-pressed])
    + .button-group__item {
    --_rs-ss: var(--corner-small);
    --_rs-es: var(--corner-small);
  }

  /* Item before pressed: end corners soften */
  .button-group--connected .button-group__item:has(+ .button-group__item[data-pressed]) {
    --_rs-se: var(--corner-small);
    --_rs-ee: var(--corner-small);
  }

  @media (prefers-reduced-motion: reduce) {
    .button-group__item { transition: none; }
  }
}
```

> **`:has()` availability**: Safari ≥15.4, Chrome ≥105, Firefox ≥121. For environments that
> cannot guarantee this, the React component applies `data-adjacent-pressed` attributes to
> siblings on `pointerdown`/`pointerup` as a progressive-enhancement fallback (see TODO).

> **Standard (non-connected) mode**: no shape morph — items are independent; morph only applies
> under `.button-group--connected`.

---

## Design Tokens

All token names are unprefixed M3 role names. Component-local variables are scoped inside the
block selector.

```css
@layer kafui {
  .button-group {
    /* component-local */
    --_gap: 4px;    /* standard mode gap; 0 in connected mode */
    --_h: 48px;     /* height; overridden by size modifier */
    --_pad-inline: 24px;

    display: inline-flex;
    gap: var(--_gap);
  }

  .button-group--connected { --_gap: 0; }

  /* Size scale */
  .button-group--size-xs { --_h: 32px; --_pad-inline: 12px; }
  .button-group--size-sm { --_h: 40px; --_pad-inline: 16px; }
  .button-group--size-md { --_h: 48px; --_pad-inline: 24px; }
  .button-group--size-lg { --_h: 56px; --_pad-inline: 32px; }
  .button-group--size-xl { --_h: 96px; --_pad-inline: 48px; }
}
```

| Role | Usage |
|---|---|
| `--primary` | Filled container |
| `--on-primary` | Filled label/icon |
| `--secondary-container` | Tonal container |
| `--on-secondary-container` | Tonal content |
| `--surface-container-low` | Elevated container |
| `--outline` | Outlined border; connected dividers |
| `--outline-variant` | Dividers within connected group (lighter than group outline) |
| `--secondary` | Focus ring |
| `--corner-full` | Default outer corners |
| `--corner-small` | Adjacent softened corners (8dp) |
| `--corner-extra-small` | Pressed inner corners (4dp) |
| `--elevation-1` | Elevated variant shadow |
| `--elevation-2` | Elevated hover shadow |
| `--easing-standard` | Shape morph transition |
| `--duration-short4` | 200ms — shape morph duration |
| `--label-large-*` | Typography |
| `--state-hover` | 8% state layer opacity |
| `--state-focus` | 10% state layer opacity |
| `--state-pressed` | 10% state layer opacity |

---

## Interaction & Accessibility

### Keyboard

| Key | Action mode | Toggle mode |
|---|---|---|
| `Tab` | Move focus item to item; exit on last | Same |
| `Space` / `Enter` | Fire `onPress` | Toggle selection |
| `Arrow Left / Right` | No cycle (Tab is used) | Roving tabindex cycle (RAC) |
| `Home` / `End` | — | Jump to first/last item (RAC toggle) |

In action mode (each button is independent), arrow keys do NOT cycle focus — Tab moves between
them. In toggle mode, RAC `ToggleButtonGroup` provides roving tabindex and arrow navigation.

### ARIA

- Root: `role="group"` (action + multi-toggle) or `role="radiogroup"` (single-toggle), set by RAC.
  `aria-label` or `aria-labelledby` **required**.
- Items: `role="button"` (action); `role="radio"` (single-toggle); managed by RAC `ToggleButton`.
- `aria-checked` / `aria-pressed` managed by RAC.
- `aria-disabled="true"` on disabled items.

### Touch

Minimum 48dp touch target. `xs` and `sm` use `min-block-size: 48px` + transparent `padding-block`.

### RTL

CSS logical properties (`border-start-start-radius`, `padding-inline`, `border-inline-end`)
guarantee correct rendering under `dir="rtl"`. Shape-morph inner/outer corners use logical
`start/end` references so morph mirrors in RTL without JS.

### Reduced Motion

`@media (prefers-reduced-motion: reduce)`: `transition: none` on all `border-radius` and
state-layer `opacity` transitions.

---

## Proposed kafUI React API

```tsx
import { ButtonGroup, ButtonGroupItem } from "@kafui/react";

// Action mode — connected, outlined (toolbar use-case)
<ButtonGroup
  aria-label="Text formatting"
  variant="outlined"
  connected
  size="md"
>
  <ButtonGroupItem onPress={bold} icon={<Icon name="format_bold" />}>Bold</ButtonGroupItem>
  <ButtonGroupItem onPress={italic} icon={<Icon name="format_italic" />}>Italic</ButtonGroupItem>
  <ButtonGroupItem onPress={underline} isDisabled icon={<Icon name="format_underlined" />}>
    Underline
  </ButtonGroupItem>
</ButtonGroup>

// Toggle mode — single-select, tonal connected (alignment switcher)
<ButtonGroup
  aria-label="Text alignment"
  variant="tonal"
  connected
  selectionMode="single"
  defaultSelectedKeys={["left"]}
  onSelectionChange={(keys) => setAlignment([...keys][0])}
>
  <ButtonGroupItem id="left"   icon={<Icon name="format_align_left" />}   aria-label="Left" />
  <ButtonGroupItem id="center" icon={<Icon name="format_align_center" />} aria-label="Center" />
  <ButtonGroupItem id="right"  icon={<Icon name="format_align_right" />}  aria-label="Right" />
</ButtonGroup>
```

### API Rationale

- **Single `<ButtonGroup>` component, not two** — `selectionMode` on the group switches the
  underlying RAC primitive (action: `Button`; toggle: `ToggleButtonGroup`). The consumer faces
  one API; the implementation bifurcates internally.
- **`connected` boolean prop** (not `variant="connected"`) — connection is a layout/shape concern,
  not a visual fill variant. Orthogonal to `variant`.
- **`variant` on the group with per-item override** — variant cascades via React Context to items;
  items that differ from the group simply pass their own `variant` prop. No context overengineering.
- **`id` required only in toggle mode** — TypeScript discriminated union enforces this.
- **No `value` string prop** — `Set<Key>` everywhere, consistent with the rest of kafUI.

### Type Signatures

```ts
type ButtonGroupVariant = "filled" | "tonal" | "outlined" | "elevated" | "text";
type ButtonGroupSize = "xs" | "sm" | "md" | "lg" | "xl";

interface ButtonGroupProps {
  variant?: ButtonGroupVariant;             // default "outlined"
  size?: ButtonGroupSize;                   // default "md"
  connected?: boolean;                      // default false
  selectionMode?: "none" | "single" | "multiple"; // default "none"
  selectedKeys?: Iterable<Key>;
  defaultSelectedKeys?: Iterable<Key>;
  onSelectionChange?: (keys: Set<Key>) => void;
  isDisabled?: boolean;
  "aria-label"?: string;
  "aria-labelledby"?: string;
  className?: string;
  children: ReactNode;
}

interface ButtonGroupItemProps {
  id?: Key;                   // required when group selectionMode !== "none"
  icon?: ReactNode;
  trailingIcon?: ReactNode;
  variant?: ButtonGroupVariant;
  isDisabled?: boolean;
  onPress?: (e: PressEvent) => void;
  "aria-label"?: string;      // required when icon-only
  className?: string;
  children?: ReactNode;       // label text
}
```

### React Aria Primitive

- `selectionMode="none"`: `ButtonGroup` renders `<div role="group" aria-label={...}>`;
  each `ButtonGroupItem` wraps RAC `Button`.
- `selectionMode="single"|"multiple"`: `ButtonGroup` renders RAC `ToggleButtonGroup` with
  `selectionMode`, `selectedKeys`, `defaultSelectedKeys`, `onSelectionChange`, `isDisabled`;
  each `ButtonGroupItem` wraps RAC `ToggleButton`.

Context provides `{variant, size, connected}` to items so they inherit group defaults.

### BEM Classes Emitted

```
.button-group
.button-group--connected
.button-group--{variant}         (e.g. --tonal)
.button-group--size-{xs|sm|md|lg|xl}

.button-group__item               (data-* attributes from RAC drive states)
.button-group__item--{variant}    (per-item override only)
.button-group__item__leading-icon
.button-group__item__label
.button-group__item__trailing-icon
.state-layer                      (shared utility)
```

All inside `@layer kafui { … }`. No `kafui-` prefix; no inline styles.

---

## Open Questions

1. **`:has()` fallback threshold**: The shape-morph sibling selectors require `:has()`. Firefox
   support arrived in 121 (Dec 2023). Define the minimum browser baseline for kafUI now and decide
   whether the JS `data-adjacent-pressed` fallback is worth the complexity or if a graceful
   degradation (morph simply absent in older browsers) is sufficient.
2. **Action mode `aria-label` requirement**: When `selectionMode="none"` the group's `role="group"`
   still requires an accessible name per ARIA spec. Enforce as required or warn?
3. **Per-item `id` enforcement in toggle mode**: TypeScript can enforce this with an overloaded
   type but it requires that the component know at compile time which mode is active. A runtime
   `invariant()` is simpler — decide which is preferred.
4. **`text` variant in connected mode**: M3 Expressive spec may not include a connected text
   variant — the divider lines look odd on borderless buttons. Flag this for visual review.
