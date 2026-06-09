# Segmented Button

## Purpose & M3 Category

Segmented Button is an **M3 Actions** component used to offer a set of 2–5 compact choices in a
horizontal row. It supports two selection modes: **single-select** (radio-like, exactly one item
always selected) and **multi-select** (checkbox-like, zero-or-more). Every segment may carry a
label, an icon, or both; a checkmark icon animates in and replaces or joins the leading icon when a
segment is selected.

Segmented buttons replace toggle-button groups, chip groups, and compact tab bars for mutually-
exclusive or multi-select filtering/view-switching use cases.

---

## Anatomy / Parts

```
[SegmentedButtonGroup]
  └── [SegmentedButton] × n  (2–5)
        ├── __check-icon       (always in DOM; CSS controls visibility/width)
        ├── __leading-icon     (optional; shifts or hides when check appears)
        ├── __label            (optional text)
        └── __state-layer      (shared ::before pseudo-element on segment)
```

| Part | BEM class | Notes |
|---|---|---|
| Root group | `.segmented-button-group` | RAC `ToggleButtonGroup` root |
| Individual segment | `.segmented-button` | RAC `ToggleButton` |
| Check icon | `.segmented-button__check-icon` | `aria-hidden`; animates in/out via width+opacity |
| Leading icon | `.segmented-button__leading-icon` | Hides on select if icon-only segment |
| Label | `.segmented-button__label` | `label-large` typescale |
| State layer | `.state-layer` (shared utility) | `::before` pseudo; `background: currentColor` |

Modifiers on `.segmented-button`:
- `--selected` — segment is active (RAC sets `data-selected` automatically; CSS targets that)
- `--disabled` — segment is disabled
- `--icon-only` — no label; icon-only layout (narrower padding)
- `--label-only` — no leading icon slot

Modifier on `.segmented-button-group`:
- `--density-0` through `--density--3` — maps to heights 40 / 36 / 32 / 28 dp

> **Note on modifiers vs data-attributes:** React Aria emits `data-selected`, `data-disabled`,
> `data-hovered`, `data-focused`, `data-pressed` automatically. CSS in `@layer kafui` should
> target `[data-selected]`, `[data-disabled]` etc. **directly** rather than adding BEM modifier
> classes in JS — this keeps the React component lean. BEM modifiers (`--icon-only`,
> `--density-*`) are only added where RAC does not emit an equivalent data-attribute.

---

## Variants

Segmented Button has a **single visual style** (outlined group, pill ends, flat internal dividers).
The variant is the *selection mode*:

| Variant | `selectionMode` | Behavior |
|---|---|---|
| Single-select | `"single"` (default) | Exactly one segment selected at all times (radio semantics) |
| Multi-select | `"multiple"` | Zero or more segments selected (checkbox semantics) |

---

## States

| State | Visual treatment |
|---|---|
| Enabled unselected | Border `--outline`; label/icon `--on-surface` |
| Enabled selected | Fill `--secondary-container`; label/icon `--on-secondary-container`; check icon visible |
| Hover | State layer at `--state-hover` (8%) over content color |
| Focus | State layer at `--state-focus` (10%); `outline` using `--secondary` |
| Pressed | State layer at `--state-pressed` (10%) |
| Disabled | 38% opacity on content; 12% on container; `pointer-events: none`; `aria-disabled` |

State layer is `::before` pseudo-element inside `.segmented-button`, absolutely positioned,
`background: currentColor`, `opacity` driven by `[data-hovered]`/`[data-pressed]`/
`[data-focus-visible]` attributes from RAC.

---

## Design Tokens

All token references use unprefixed role names (resolved by `@kafui/styles` via the OKLCH pipeline
into real CSS custom properties). Component-local variables are declared inside the block selector.

```css
@layer kafui {
  .segmented-button-group {
    /* component-local */
    --h: 40px;          /* height at density-0; overridden per density modifier */
    --icon-size: 18px;

    border: 1px solid var(--outline);
    border-radius: var(--corner-full);
    overflow: hidden;
  }

  .segmented-button {
    /* component-local */
    --_pad-inline: 12px;

    block-size: var(--h);
    min-block-size: 48px;       /* touch target */
    padding-inline: var(--_pad-inline);
    color: var(--on-surface);
    font: var(--label-large-font);
    font-size: var(--label-large-size);
    font-weight: var(--label-large-weight);
    line-height: var(--label-large-line-height);
  }

  /* density scale */
  .segmented-button-group.--density--1 { --h: 36px; }
  .segmented-button-group.--density--2 { --h: 32px; }
  .segmented-button-group.--density--3 { --h: 28px; }

  /* selected segment */
  .segmented-button[data-selected] {
    background: var(--secondary-container);
    color: var(--on-secondary-container);
  }

  /* disabled */
  .segmented-button[data-disabled] {
    opacity: 0.38;
    pointer-events: none;
  }

  /* check icon */
  .segmented-button__check-icon {
    inline-size: 0;
    opacity: 0;
    overflow: hidden;
    transition:
      inline-size 150ms var(--easing-standard),
      opacity 100ms var(--easing-standard);
  }
  .segmented-button[data-selected] .segmented-button__check-icon {
    inline-size: var(--icon-size);   /* 18px */
    opacity: 1;
  }

  /* reduced motion */
  @media (prefers-reduced-motion: reduce) {
    .segmented-button__check-icon { transition: none; }
    .state-layer { transition: none; }
  }
}
```

| Role | Usage |
|---|---|
| `--outline` | Group border and internal dividers |
| `--secondary-container` | Selected segment fill |
| `--on-secondary-container` | Label + icon when selected |
| `--on-surface` | Label + icon when unselected |
| `--secondary` | Focus ring color |
| `--corner-full` | Pill ends |
| `--label-large-*` | `font-size`, `weight`, `line-height` |
| `--easing-standard` | Check-icon entrance transition |
| `--state-hover` | 8% state layer opacity |
| `--state-focus` | 10% state layer opacity |
| `--state-pressed` | 10% state layer opacity |

---

## Interaction & Accessibility

### Keyboard

| Key | Behavior (single) | Behavior (multi) |
|---|---|---|
| `Tab` | Focus enters group (roving tabindex); exits on next Tab | Same |
| `Arrow Left / Right` | Move focus between segments; wraps | Move focus only (no auto-select) |
| `Space` / `Enter` | Select focused segment; deselects others | Toggle focused segment |
| `Home` / `End` | Jump to first / last segment | Same |

RAC `ToggleButtonGroup` implements roving tabindex automatically — only one segment holds
`tabindex="0"` at a time.

### ARIA

- Group: `role="radiogroup"` (single-select) or `role="group"` (multi-select), automatically
  set by RAC based on `selectionMode`. `aria-label` or `aria-labelledby` **required**.
- Each segment: RAC `ToggleButton` emits `role="radio"` (single) or standard `role="button"` with
  `aria-pressed` (multi). M3 spec calls for `role="checkbox"` in multi — verify RAC v1 behavior
  and add an `aria-role` override if needed.
- `aria-checked` / `aria-pressed` managed by RAC via `data-selected`.
- `aria-disabled="true"` on disabled segments.

### Touch / Hit Area

Minimum 48×48 dp hit area per M3. Visual height may be 40 dp; `min-block-size: 48px` with
`padding-block` provides the invisible extension without affecting layout.

### RTL

CSS logical properties (`padding-inline-start/end`, `border-start-start-radius`,
`margin-inline-end`) ensure correct rendering under `dir="rtl"`. The leading icon and check icon
reverse order naturally via the logical-property flex model.

### Reduced Motion

Check-icon entrance and state-layer transitions are wrapped in
`@media (prefers-reduced-motion: reduce)` with `transition: none`.

---

## Proposed kafUI React API

```tsx
import { SegmentedButtonGroup, SegmentedButton } from "@kafui/react";

// Single-select (default)
<SegmentedButtonGroup
  selectionMode="single"
  defaultSelectedKeys={["day"]}
  aria-label="View"
  density={0}               // 0 | -1 | -2 | -3; default 0
  onSelectionChange={(keys) => setView([...keys][0])}
>
  <SegmentedButton id="day" icon={<Icon name="calendar_today" />}>Day</SegmentedButton>
  <SegmentedButton id="week">Week</SegmentedButton>
  <SegmentedButton id="month" isDisabled>Month</SegmentedButton>
</SegmentedButtonGroup>

// Multi-select — icon-only segments
<SegmentedButtonGroup selectionMode="multiple" aria-label="Active filters">
  <SegmentedButton id="wifi" icon={<Icon name="wifi" />} aria-label="Wi-Fi" />
  <SegmentedButton id="bluetooth" icon={<Icon name="bluetooth" />} aria-label="Bluetooth" />
</SegmentedButtonGroup>
```

### API Rationale

- **`selectionMode`** on the group, not a boolean `multiple` — mirrors RAC's own prop and makes
  the tri-state (none / single / multiple, if ever needed) extensible.
- **`density`** not `size` — matches M3 density scale semantics; size implies different tap
  target, density is pure visual compactness. Range `0 → -3` mirrors M3 spec exactly.
- **No `value` / `onChange` string array** — uses RAC `Set<Key>` throughout for consistency with
  every other kafUI selection component (Tabs, ChipGroup, etc.).
- **`icon` as ReactNode on `SegmentedButton`** — not a string name — keeps the API component-
  agnostic (works with any icon system, not just the sprite-backed `<Icon>`).
- **`aria-label` on icon-only segments** — TypeScript requires `aria-label` when `children` is
  absent (discriminated union).

### Type Signatures

```ts
interface SegmentedButtonGroupProps {
  selectionMode?: "single" | "multiple";      // default "single"
  selectedKeys?: Iterable<Key>;               // controlled
  defaultSelectedKeys?: Iterable<Key>;        // uncontrolled
  onSelectionChange?: (keys: Set<Key>) => void;
  density?: 0 | -1 | -2 | -3;               // default 0
  isDisabled?: boolean;
  className?: string;
  "aria-label"?: string;
  "aria-labelledby"?: string;
  children: ReactNode;
}

// Icon-only: aria-label required; label optional when icon provided
type SegmentedButtonProps =
  | {
      id: Key;
      icon: ReactNode;
      "aria-label": string;   // required when no children
      children?: never;
      isDisabled?: boolean;
      className?: string;
    }
  | {
      id: Key;
      icon?: ReactNode;
      children: ReactNode;    // label text
      "aria-label"?: string;
      isDisabled?: boolean;
      className?: string;
    };
```

### React Aria Primitive

`SegmentedButtonGroup` → RAC `ToggleButtonGroup` with `selectionMode`, `selectedKeys`,
`defaultSelectedKeys`, `onSelectionChange`, `isDisabled` forwarded directly.

`SegmentedButton` → RAC `ToggleButton` with `id`, `isDisabled`. The check icon
(`<span class="segmented-button__check-icon" aria-hidden>`) is always rendered in the DOM;
CSS drives its visual presence via `[data-selected]` — zero JS toggling, zero layout shift.

### BEM Classes Emitted

```
.segmented-button-group
.segmented-button-group--density-{0|-1|-2|-3}

.segmented-button                    (RAC data-* attributes drive states)
.segmented-button--icon-only         (set in JS when no children text)
.segmented-button__check-icon
.segmented-button__leading-icon
.segmented-button__label
.state-layer                         (shared utility class on ::before)
```

All inside `@layer kafui { … }`. No `kafui-` prefix on classes; no inline styles; no `style`
attribute in JSX.

---

## Open Questions

1. **RAC multi-select ARIA**: RAC `ToggleButtonGroup` with `selectionMode="multiple"` currently
   emits `role="group"` + `aria-pressed` per button (not `role="checkbox"`). Verify whether M3 a11y
   guidelines require `role="checkbox"` and if a RAC `slot` override is appropriate.
2. **Density class syntax**: The modifier `--density--1` has a double-dash which looks odd. Consider
   `density-neg1` or `density-minus-1`. Resolve consistently with other density-bearing components
   (e.g. List). Flag for lead to align across library.
3. **`selectionMode="none"`**: M3 spec does not define a non-selecting segmented button; do not add
   `"none"` to avoid confusion with `ButtonGroup`. If an action-row is needed, use `ButtonGroup`.
