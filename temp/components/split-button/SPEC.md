# Split Button ✦ M3 Expressive

## Purpose & M3 Category

Split Button is an **M3 Expressive (2025) Actions** component. It is a two-segment control:

- **Leading action segment**: executes the primary action immediately on press.
- **Trailing trigger segment**: opens a dropdown menu of secondary or variant actions via a chevron
  button.

When the menu opens, the trailing segment **morphs its outer (end) corners from pill to
`extra-small`** (≈4dp), shifting the group's silhouette to visually signal the open/anchored state.
The chevron rotates 180° simultaneously.

Available in **Elevated**, **Filled**, **Tonal**, and **Outlined** styles and in **5 sizes**.

---

## Anatomy / Parts

```
[SplitButton]   role="group"
  ├── .split-button__action      (RAC Button — primary action)
  │     ├── .split-button__action-icon      (optional leading icon; 18–24dp by size)
  │     ├── .split-button__action-label     (required label text)
  │     └── .state-layer                   (::before pseudo)
  ├── .split-button__divider     (1dp vertical line; decorative)
  ├── .split-button__trigger     (RAC Button — opens menu)
  │     ├── .split-button__trigger-icon    (<Icon name="arrow_drop_down" />; rotates on open)
  │     └── .state-layer
  └── [Menu]  (RAC Menu positioned below/above trigger)
        └── [SplitButton.Item / MenuItem] × n
```

| Part | BEM class | Notes |
|---|---|---|
| Root | `.split-button` | `<div role="group">` — NOT a button |
| Action segment | `.split-button__action` | RAC `Button` |
| Action icon | `.split-button__action-icon` | Optional; scaled by size modifier |
| Action label | `.split-button__action-label` | `label-large` typescale |
| Divider | `.split-button__divider` | decorative `aria-hidden` |
| Trigger segment | `.split-button__trigger` | RAC `Button`; opens `Menu` |
| Trigger icon (chevron) | `.split-button__trigger-icon` | Rotates 180° when `[data-open]` |
| State layer | `.state-layer` | Shared `::before` pseudo on each segment |

Modifiers on `.split-button`:
- `--filled` / `--tonal` / `--outlined` / `--elevated` — visual variant
- `[data-open]` — applied by React when `MenuTrigger` is open; drives morph + chevron rotation
- `[data-disabled]`
- `--size-{xs|sm|md|lg|xl}` — five sizes

> **`[data-open]` as a data attribute, not a BEM modifier** — the open state is toggled via
> `onOpenChange` from RAC `MenuTrigger`. Using a `data-open` attribute on the root is
> idiomatic RAC style and avoids extra JS class-string manipulation.

---

## Variants

### Fill variant (`variant` prop)

| Value | Container | Content | Border |
|---|---|---|---|
| `"filled"` (default) | `--primary` | `--on-primary` | none |
| `"tonal"` | `--secondary-container` | `--on-secondary-container` | none |
| `"outlined"` | transparent | `--primary` | `1px solid --outline` |
| `"elevated"` | `--surface-container-low` + shadow | `--primary` | none |

`"text"` is intentionally omitted — the divider and connected silhouette require a visible
container; a borderless text split-button is incoherent in M3 spec.

### Sizes

| Size | `size` | Total height | Action `padding-inline` | Trigger width | Min touch |
|---|---|---|---|---|---|
| `"xs"` | 32dp | 12dp | 32dp | 48dp (padded) |
| `"sm"` | 40dp | 16dp | 36dp | 48dp (padded) |
| `"md"` | 48dp (default) | 24dp | 40dp | 48dp |
| `"lg"` | 56dp | 32dp | 44dp | 56dp |
| `"xl"` | 96dp | 48dp | 64dp | 96dp |

For `xs` and `sm`, the minimum 48dp touch target is achieved via `min-block-size: 48px` +
transparent `padding-block` on each segment.

---

## States

### Action segment

| State | Treatment |
|---|---|
| Enabled | Per-variant fill + content colors |
| Hover | State layer at `--state-hover` (8%) |
| Focus | State layer at `--state-focus` (10%); `outline` using `--secondary` |
| Pressed | State layer at `--state-pressed` (10%) |
| Disabled | 38% opacity; `pointer-events: none` |

### Trigger segment

| State | Treatment |
|---|---|
| Closed (rest) | Shares container fill; chevron at `rotate(0deg)` |
| Open | `[data-open]` on root: trigger **end** corners morph to `--corner-extra-small` (≈4dp); chevron `rotate(180deg)` |
| Hover | State layer 8% |
| Focus | State layer 10%; focus ring |
| Pressed | State layer 10% |
| Disabled | 38% opacity |

### Shape Morph on Open

The trailing corners of the trigger segment contract when the menu opens, and the pill silhouette
of the trigger "opens up" toward the menu panel:

```css
@layer kafui {
  .split-button__trigger {
    /* component-local corner vars */
    --_rs-ss: var(--corner-full);
    --_rs-se: var(--corner-full);
    --_rs-es: var(--corner-full);
    --_rs-ee: var(--corner-full);

    border-start-start-radius: var(--_rs-ss);
    border-start-end-radius:   var(--_rs-se);
    border-end-start-radius:   var(--_rs-es);
    border-end-end-radius:     var(--_rs-ee);

    transition:
      border-radius var(--duration-medium1, 250ms) var(--easing-emphasized-decelerate),
      border-radius var(--duration-medium1, 250ms) var(--easing-emphasized-accelerate);
  }

  /* When menu is open: trailing corners contract */
  .split-button[data-open] .split-button__trigger {
    --_rs-se: var(--corner-extra-small);
    --_rs-ee: var(--corner-extra-small);
    /* open easing overrides */
    transition-timing-function: var(--easing-emphasized-decelerate);
  }

  /* Separate close easing requires JS class toggle or CSS @starting-style */
  /* Recommended: use JS to toggle data-closing for the accelerate easing */

  .split-button__trigger-icon {
    transition: transform var(--duration-medium1, 250ms) var(--easing-emphasized);
    transform: rotate(0deg);
  }
  .split-button[data-open] .split-button__trigger-icon {
    transform: rotate(180deg);
  }

  @media (prefers-reduced-motion: reduce) {
    .split-button__trigger,
    .split-button__trigger-icon { transition: none; }
  }
}
```

> **Open vs close easing**: M3 spec uses `emphasized-decelerate` on open and
> `emphasized-accelerate` on close. Pure CSS cannot switch easing direction on removal of a class.
> The recommended approach is for React to add `data-open` on open and `data-closing`
> (briefly, for the close animation duration) on close, so both easings are CSS-targeted.
> Alternatively, use the CSS `@starting-style` rule if the target baseline supports it (Chrome
> 117+, not yet Safari/Firefox as of 2026-06 — document the fallback).

---

## Design Tokens

```css
@layer kafui {
  .split-button {
    /* component-local */
    --_h: 48px;
    --_action-pad: 24px;
    --_trigger-w: 40px;
    --_container: var(--primary);
    --_content:   var(--on-primary);

    display: inline-flex;
    align-items: stretch;
    block-size: var(--_h);
    min-block-size: 48px;
    border-radius: var(--corner-full);
    overflow: hidden;
  }

  /* Variant token maps */
  .split-button--filled   { --_container: var(--primary); --_content: var(--on-primary); }
  .split-button--tonal    { --_container: var(--secondary-container); --_content: var(--on-secondary-container); }
  .split-button--outlined { --_container: transparent; --_content: var(--primary); }
  .split-button--elevated { --_container: var(--surface-container-low); --_content: var(--primary); }

  /* Size scale */
  .split-button--size-xs { --_h: 32px; --_action-pad: 12px; --_trigger-w: 32px; }
  .split-button--size-sm { --_h: 40px; --_action-pad: 16px; --_trigger-w: 36px; }
  .split-button--size-md { --_h: 48px; --_action-pad: 24px; --_trigger-w: 40px; }
  .split-button--size-lg { --_h: 56px; --_action-pad: 32px; --_trigger-w: 44px; }
  .split-button--size-xl { --_h: 96px; --_action-pad: 48px; --_trigger-w: 64px; }
}
```

| Role | Usage |
|---|---|
| `--primary` | Filled container |
| `--on-primary` | Filled content |
| `--secondary-container` | Tonal container |
| `--on-secondary-container` | Tonal content |
| `--surface-container-low` | Elevated container |
| `--outline` | Outlined border; divider |
| `--outline-variant` | Divider in filled/tonal variants |
| `--secondary` | Focus ring |
| `--corner-full` | Default pill corners |
| `--corner-extra-small` | Morphed trigger trailing corners (≈4dp) |
| `--elevation-1` | Elevated shadow at rest |
| `--elevation-2` | Elevated hover shadow |
| `--easing-emphasized-decelerate` | Menu open morph |
| `--easing-emphasized-accelerate` | Menu close morph |
| `--easing-emphasized` | Chevron rotation |
| `--duration-medium1` | 250ms — morph and chevron |
| `--label-large-*` | Typography |
| `--state-hover` | 8% |
| `--state-focus` | 10% |
| `--state-pressed` | 10% |

---

## Interaction & Accessibility

### Keyboard

| Key | Target | Behavior |
|---|---|---|
| `Tab` | Group | Focus enters action segment, then trigger |
| `Space` / `Enter` | Action segment | Fires `onPress`; menu stays closed |
| `Space` / `Enter` / `ArrowDown` | Trigger segment | Opens menu; focus moves to first menu item |
| `Escape` | Open menu | Closes; focus returns to trigger |
| `ArrowUp` / `ArrowDown` | Open menu | Navigate items |
| `Home` / `End` | Open menu | First / last item |
| `Tab` | Open menu | Closes menu; moves focus out |

### ARIA

- Root `<div>`: `role="group"`, `aria-label` **required** (describes whole grouped action,
  e.g. `"Create with options"`).
- Action segment: `role="button"`, `aria-label` matching its visible label.
- Trigger segment: `role="button"`, `aria-haspopup="menu"`, `aria-expanded` (managed by RAC
  `MenuTrigger`), `aria-controls={menuId}`, `aria-label` (e.g. `"More actions"`).
- Menu: `role="menu"`, `aria-labelledby` pointing to trigger.
- Menu items: `role="menuitem"`.
- `aria-disabled` on disabled items.

> **`aria-label` on the trigger segment** must be explicit because the segment contains only a
> chevron icon. Default: `"More {label} actions"` where `label` is the action segment label.
> The component generates this automatically if `triggerLabel` is not provided.

### Focus Management

RAC `MenuTrigger` handles:
- On open: focus to first non-disabled menu item.
- On Escape / item activation: focus returns to trigger segment.

### Touch / Hit Area

All segments: `min-block-size: 48px`. At `xs`/`sm`, transparent `padding-block` provides the
invisible extension.

### RTL

Under `dir="rtl"` the action segment is visually on the right and the trigger on the left. Use
`flex-direction: row-reverse` on the root under `:dir(rtl)`, **or** rely entirely on CSS logical
properties for corners and padding so no directional selector is needed:

- `border-start-start-radius` / `border-start-end-radius` / `border-end-start-radius` /
  `border-end-end-radius` on the trigger correctly reference the trailing (end) corners in both
  LTR and RTL.
- Preferred approach: pure logical properties, no `row-reverse` hack.

### Reduced Motion

`@media (prefers-reduced-motion: reduce)`: all `transition: none` on trigger corners and chevron.
Menu appears/disappears instantly (`transition: none` on any menu entrance animation).

---

## Proposed kafUI React API

```tsx
import { SplitButton } from "@kafui/react";

<SplitButton
  variant="filled"
  size="md"
  label="Create"
  icon={<Icon name="add" />}
  onPress={() => handleCreate()}
  aria-label="Create with options"
  onAction={(key) => handleMenuAction(key)}
>
  <SplitButton.Item id="template">From template</SplitButton.Item>
  <SplitButton.Item id="import">Import…</SplitButton.Item>
  <SplitButton.Item id="duplicate" isDisabled>Duplicate</SplitButton.Item>
</SplitButton>
```

### API Rationale

- **`SplitButton.Item` compound sub-component** (not a separate `SplitButtonItem` export) — keeps
  the import surface minimal and collocates the menu item type with its parent.
- **`label` as a string prop** (not `children`) — the action label is structurally required and
  semantically separate from the menu items (which are `children`). `children` is reserved for
  `SplitButton.Item` elements only, keeping the API unambiguous.
- **`onAction` not `onMenuAction`** — mirrors RAC `Menu`'s own `onAction` prop name; consistent
  with the kafUI `Menu` component.
- **`triggerLabel` optional** — defaults to `"More ${label} actions"` for a11y; overridable for
  i18n.
- **No separate `isMenuOpen` + toggle toggle** — `isOpen` / `defaultOpen` / `onOpenChange` for
  controlled/uncontrolled open state. The `[data-open]` attribute on the root is the CSS hook;
  no separate class management needed.
- **`aria-label` on root is required** — TypeScript non-optional.

### Type Signatures

```ts
type SplitButtonVariant = "filled" | "tonal" | "outlined" | "elevated";
type SplitButtonSize    = "xs" | "sm" | "md" | "lg" | "xl";

interface SplitButtonProps {
  variant?:       SplitButtonVariant;       // default "filled"
  size?:          SplitButtonSize;          // default "md"
  label:          string;                   // action segment label (required)
  icon?:          ReactNode;                // leading icon on action segment
  onPress?:       (e: PressEvent) => void;  // primary action handler
  onAction?:      (key: Key) => void;       // menu item selected handler
  triggerLabel?:  string;                   // accessible name for chevron trigger
  isOpen?:        boolean;                  // controlled open state
  defaultOpen?:   boolean;                  // uncontrolled; default false
  onOpenChange?:  (open: boolean) => void;
  isDisabled?:    boolean;
  "aria-label":   string;                   // required — describes the group
  className?:     string;
  children:       ReactNode;                // SplitButton.Item elements
}

interface SplitButtonItemProps {
  id:          Key;
  isDisabled?: boolean;
  className?:  string;
  children:    ReactNode;                   // menu item label
}
```

### React Aria Primitive

```
SplitButton root:    <div role="group" aria-label={...} data-open={isOpen}>
  action segment:    RAC <Button onPress={onPress}> (no MenuTrigger involvement)
  trigger + menu:    RAC <MenuTrigger isOpen onOpenChange>
                       <Button __trigger aria-haspopup="menu" aria-expanded aria-controls>
                       <Popover>
                         <Menu onAction>{children}</Menu>
                       </Popover>
```

`MenuTrigger.onOpenChange` drives the `data-open` attribute on the root div, which CSS uses for
morph and chevron rotation. No separate state needed.

`SplitButton.Item` renders RAC `MenuItem`.

### BEM Classes Emitted

```
.split-button
.split-button--{variant}
.split-button--size-{xs|sm|md|lg|xl}

.split-button__action
.split-button__action-icon
.split-button__action-label
.split-button__divider
.split-button__trigger
.split-button__trigger-icon
.state-layer                   (on each segment)
```

Data attributes on root: `data-open`, `data-disabled`.
All inside `@layer kafui { … }`. No `kafui-` prefix; no inline styles.

---

## Open Questions

1. **Open/close easing split**: CSS cannot natively play different easing functions on the same
   property when a class is added vs removed. Three options: (a) `data-closing` attribute set
   briefly on close; (b) CSS `@starting-style` (Chrome 117+ only); (c) JS `style` assignment for
   close transition. Decide now before implementation.
2. **Divider color per variant**: In `filled` and `tonal`, the divider should use
   `--outline-variant` (lighter) so it does not visually compete with the container. In
   `outlined`, the divider should match `--outline`. Confirm per-variant divider color in M3 spec
   or use a single `--outline-variant` for all.
3. **Menu width**: Should the menu panel be at least as wide as the full split-button, or only as
   wide as the trigger segment? M3 spec examples show menu matching the full button width. Confirm.
4. **`text` variant exclusion**: Confirmed intentional (see Variants). Document clearly so
   consumers do not file a "missing variant" bug.
