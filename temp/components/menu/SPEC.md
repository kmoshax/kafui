# Menu

M3 category: **Containment / Menus**. A menu displays a list of choices on a temporary surface, anchored to a trigger element. Items can carry a leading icon, a label, trailing shortcut text, or trailing subtext. Dividers group related items. Items can be selectable (radio/checkbox semantics). Submenus cascade from a parent item.

---

## Anatomy / Parts

```
[menu-trigger]              — the anchor element (Button, IconButton, etc.)
[menu-positioner]           — RAC Popover, manages placement & flip
  └─ [menu-container]       — the surface
       ├─ [menu-list]        — scrollable item list
       │    ├─ [menu-item]*
       │    │    ├─ [item-leading-icon]?
       │    │    ├─ [item-label]
       │    │    ├─ [item-supporting-text]?   — secondary line
       │    │    └─ [item-trailing-text]?     — keyboard shortcut or badge
       │    ├─ [menu-divider]*
       │    └─ [menu-submenu-item]*           — item with "›" indicator
       │         └─ [submenu]                 — nested menu-container
       └─ [menu-header]?    — non-interactive section label
```

| Part | BEM element |
|---|---|
| Root | `.menu` |
| Container surface | `.menu__container` |
| Scrollable list | `.menu__list` |
| Menu item | `.menu__item` |
| Leading icon | `.menu__item-leading-icon` |
| Label | `.menu__item-label` |
| Supporting text | `.menu__item-supporting-text` |
| Trailing text/shortcut | `.menu__item-trailing-text` |
| Divider | `.menu__divider` |
| Section header | `.menu__section-header` |
| Submenu indicator icon | `.menu__item-submenu-icon` |
| State layer | `.menu__item-state-layer` |

BEM modifiers on `.menu__item`:
- `--focused`, `--hovered`, `--pressed` (driven by RAC `data-*` attributes; prefer `data-*` selectors over manual modifier classes)
- `--selected` (for selectable items — checkmark or radio dot)
- `--disabled`
- `--has-leading-icon` (layout shift prevention when mixing icon/no-icon items)
- `--has-submenu`

> Prefer `[data-hovered]`, `[data-focused]`, `[data-pressed]`, `[data-selected]`, `[data-disabled]` selectors over modifier classes where RAC emits data attributes — class modifiers are only needed for non-RAC contexts.

---

## Variants

### Standard Menu
- Non-selectable items; each triggers an action on press.
- Items: leading icon (optional), label, trailing text (optional).
- Dividers separate groups.
- Most common variant.

### Menu with Submenu
- A menu item has a right-pointing chevron (`›`, flips in RTL to `‹`).
- Hovering or pressing the item opens a nested menu anchored to the item's trailing edge.
- Submenus can be nested (though M3 recommends max 1 level).
- Keyboard: `→` opens submenu, `←` closes and returns focus to parent item.

### Selectable Menu
- Items behave as radio buttons or checkboxes within a section.
- Selected items show a leading checkmark (single-select) or checkbox indicator.
- `selectionMode="single"` (radio) or `"multiple"` (checkbox).
- State persists across opens/closes.

---

## States

| State | Token applied |
|---|---|
| Default | No state layer |
| Hover | State-layer 8% `--on-surface` |
| Focus | State-layer 10% `--on-surface`; outline focus ring |
| Pressed | State-layer 10% `--on-surface` + ripple |
| Disabled | Label/icon `--on-surface` @ 38% opacity; no interaction |
| Selected | Leading checkmark/dot; label `--on-surface` |

Items do not have an "active/current" highlight beyond focus; hover and focus are visually distinct.

---

## Design Tokens

### Color
| Role | Token |
|---|---|
| Container surface | `--surface-container` |
| Container overlay (elevation tint) | `--surface-tint` |
| Item label | `--on-surface` |
| Item supporting text | `--on-surface-variant` |
| Item trailing text | `--on-surface-variant` |
| Leading/trailing icons | `--on-surface-variant` |
| State-layer color | `--on-surface` |
| Divider | `--outline-variant` |
| Section header | `--on-surface-variant` |
| Disabled | `--on-surface` @ 38% |

### Shape
| Part | Token |
|---|---|
| Container | `--corner-extra-small` (4 dp) |

### Typography
| Role | Token |
|---|---|
| Item label | `--label-large-size` / `--label-large-weight` |
| Item supporting text | `--label-medium-size` / `--label-medium-weight` |
| Item trailing text (shortcut) | `--label-large-size` / `--label-large-weight` |
| Section header | `--label-small-size` / `--label-small-weight` |

### Elevation
| State | Token |
|---|---|
| Container at rest | `--elevation-2` |

### Spacing
| Spec | Value |
|---|---|
| Minimum width | 112 dp |
| Maximum width | 280 dp |
| Item height (single-line) | 48 dp |
| Item height (two-line, with supporting text) | 64 dp |
| Horizontal padding | 12 dp (leading icon) / 16 dp (label start if no icon) |
| Leading icon size | 24 dp |
| Trailing text | inline-end aligned in item row |

### Motion
| Transition | Token |
|---|---|
| Menu open (scale + fade) | `--duration-short4` + `--easing-emphasized-decelerate` |
| Menu close (fade) | `--duration-short2` + `--easing-emphasized-accelerate` |
| Submenu open | Same as menu open |

---

## Interaction & Accessibility

### React Aria Primitives
- `MenuTrigger` — manages open/close, anchors `Menu` to trigger.
- `Menu` — `role="menu"`, keyboard navigation, type-ahead, selection.
- `MenuItem` — `role="menuitem"` (or `"menuitemradio"` / `"menuitemcheckbox"` for selectable).
- `MenuSection` — groups items; renders `role="group"` with `aria-label`.
- `Separator` — `role="separator"` (divider).
- `Popover` from RAC — handles placement, flip, dismiss on outside click / Escape.
- `SubmenuTrigger` — RAC primitive (available in RAC 1.3+) for submenu cascade.

### ARIA Roles
- Menu container: `role="menu"` with `aria-label` or `aria-labelledby` (trigger label).
- Standard item: `role="menuitem"`.
- Selectable single: `role="menuitemradio"` + `aria-checked`.
- Selectable multiple: `role="menuitemcheckbox"` + `aria-checked`.
- Section: `role="group"` + `aria-label`.
- Divider: `role="separator"`.
- Submenu item: `role="menuitem"` with `aria-haspopup="menu"` + `aria-expanded`.
- Icons: `aria-hidden="true"`; shortcut text `aria-keyshortcuts`.

### Keyboard Navigation
| Key | Action |
|---|---|
| `↑/↓` | Move focus between items (wraps) |
| `Home/End` | Focus first/last item |
| `Enter`/`Space` | Activate item (trigger action or toggle selection) |
| `Escape` | Close menu; return focus to trigger |
| `→` | Open submenu (if item has submenu); in RTL: close submenu |
| `←` | Close submenu; return focus to parent menu item; in RTL: open submenu |
| `Tab` | Close menu; move focus to next element in document |
| Type-ahead | Typing letters jumps focus to matching item label |

### RTL
- Menu opens from opposite side (RAC `Popover` placement flips automatically).
- Submenu indicator icon: `›` (chevron-right) flips to `‹` (chevron-left) in RTL via `:dir(rtl) .menu__item-submenu-icon { transform: scaleX(-1) }`.
- Arrow key direction for submenu (`→`/`←`) swaps in RTL — RAC `SubmenuTrigger` handles this.
- All padding logical (`padding-inline-start/end`).

### Reduced Motion
```css
@layer kafui {
  @media (prefers-reduced-motion: reduce) {
    .menu__container { transition: none; animation: none; }
  }
}
```

### Focus Management
- On open: focus moves to first non-disabled menu item.
- On close: focus returns to trigger.
- RAC `MenuTrigger` handles both automatically.

---

## CSS Architecture

```css
@layer kafui {
  .menu {
    /* Component-scoped vars */
    --radius: var(--corner-extra-small);
    --min-w: 112px;
    --max-w: 280px;
    --item-h: 48px;
    --item-h-2line: 64px;
    --icon-size: 24px;
    --state-color: var(--on-surface);
  }

  .menu__container {
    border-radius: var(--radius);
    background: var(--surface-container);
    box-shadow: var(--elevation-2);
    min-width: var(--min-w);
    max-width: var(--max-w);
    overflow: hidden;
  }

  /* Elevation tint overlay */
  .menu__container::before {
    content: "";
    position: absolute;
    inset: 0;
    border-radius: inherit;
    background: var(--surface-tint);
    opacity: var(--elevation-tint-2); /* elevation-level2 tint opacity */
    pointer-events: none;
  }

  .menu__list {
    padding-block: 8px;
    overflow-y: auto;
    max-height: 50dvh;
  }

  .menu__item {
    position: relative;
    display: flex;
    align-items: center;
    gap: 12px;
    min-height: var(--item-h);
    padding-inline: 12px;
    font-size: var(--label-large-size);
    font-weight: var(--label-large-weight);
    color: var(--on-surface);
    cursor: pointer;
    overflow: hidden;
  }

  /* State-layer */
  .menu__item::before {
    content: "";
    position: absolute;
    inset: 0;
    background: var(--state-color);
    opacity: 0;
    pointer-events: none;
    transition: opacity var(--duration-short3) var(--easing-standard);
  }
  .menu__item[data-hovered]::before  { opacity: var(--state-hover); }
  .menu__item[data-focused]::before  { opacity: var(--state-focus); }
  .menu__item[data-pressed]::before  { opacity: var(--state-pressed); }

  .menu__item--has-supporting-text {
    min-height: var(--item-h-2line);
    align-items: flex-start;
    padding-block: 8px;
  }

  .menu__item[data-disabled] {
    opacity: 0.38;
    pointer-events: none;
  }

  /* Trailing text pushed to inline-end */
  .menu__item-trailing-text {
    margin-inline-start: auto;
    color: var(--on-surface-variant);
    font-size: var(--label-large-size);
  }

  /* Submenu chevron flips in RTL */
  .menu__item-submenu-icon {
    margin-inline-start: auto;
  }
  :dir(rtl) .menu__item-submenu-icon { transform: scaleX(-1); }

  .menu__divider {
    block-size: 1px;
    background: var(--outline-variant);
    margin-block: 8px;
    margin-inline: 0;
    border: none;
  }

  .menu__section-header {
    font-size: var(--label-small-size);
    font-weight: var(--label-small-weight);
    color: var(--on-surface-variant);
    padding-inline: 16px;
    padding-block: 8px;
    pointer-events: none;
  }

  /* Open animation — transform-origin at logical block-start inline-start */
  @keyframes menu-open {
    from { opacity: 0; transform: scale(0.9); }
    to   { opacity: 1; transform: scale(1); }
  }
  @keyframes menu-close {
    from { opacity: 1; }
    to   { opacity: 0; }
  }
  .menu__container[data-entering] {
    animation: menu-open var(--duration-short4) var(--easing-emphasized-decelerate);
    transform-origin: top left; /* overridden for RTL in :dir block */
  }
  :dir(rtl) .menu__container[data-entering] {
    transform-origin: top right;
  }
  .menu__container[data-exiting] {
    animation: menu-close var(--duration-short2) var(--easing-emphasized-accelerate);
  }

  @media (prefers-reduced-motion: reduce) {
    .menu__container { animation: none; transition: none; }
    .menu__item::before { transition: none; }
  }
}
```

---

## Proposed kafUI React API

```tsx
import {
  MenuTrigger,
  Menu,
  MenuItem,
  MenuSection,
  SubmenuTrigger,
  Separator,
  Popover,
  Header,
} from "react-aria-components";

// ── Menu Item data shape ──────────────────────────────────────
interface MenuItemConfig {
  id: string;
  label: string;
  leadingIcon?: string;        // sprite icon name
  trailingText?: string;       // shortcut e.g. "⌘C"
  supportingText?: string;     // second line
  isDisabled?: boolean;
  onPress?: () => void;
  /** Nested submenu items (one level max recommended) */
  children?: MenuItemConfig[];
}

// ── Compound API ──────────────────────────────────────────────
// Menu         — root compound component (wraps MenuTrigger + Popover + RAC Menu)
// Menu.Item    — individual item (wraps RAC MenuItem)
// Menu.Section — group with label (wraps RAC MenuSection + Header)
// Menu.Divider — separator (wraps RAC Separator)
// Menu.SubmenuItem — item that opens a nested Menu

// ── Top-level convenience component ──────────────────────────
interface MenuProps {
  /** The trigger element */
  trigger: React.ReactElement;
  /** Where the menu opens relative to trigger */
  placement?: "bottom" | "bottom-start" | "bottom-end" | "top" | "top-start" | "top-end";
  /** "single" = menuitemradio; "multiple" = menuitemcheckbox; undefined = menuitem */
  selectionMode?: "single" | "multiple";
  selectedKeys?: Set<string>;
  defaultSelectedKeys?: Set<string>;
  onSelectionChange?: (keys: Set<string>) => void;
  onAction?: (key: string) => void;
  isDisabled?: boolean;
  "aria-label"?: string;
  children: React.ReactNode; // Menu.Item, Menu.Section, Menu.Divider
}

namespace Menu {
  interface ItemProps {
    id: string;
    leadingIcon?: string;
    trailingText?: string;
    supportingText?: string;
    isDisabled?: boolean;
    onPress?: () => void;
    children: React.ReactNode; // label
  }

  interface SectionProps {
    "aria-label": string;
    children: React.ReactNode; // Menu.Item elements
  }

  interface SubmenuItemProps {
    id: string;
    leadingIcon?: string;
    isDisabled?: boolean;
    children: React.ReactNode; // label
    submenu: React.ReactElement; // <Menu ...> without trigger (trigger=null)
  }
  // Internally wraps RAC SubmenuTrigger + nested Menu
}

// ── Usage Examples ────────────────────────────────────────────
// Standard:
<Menu
  trigger={<Button aria-label="More options"><Icon name="more_vert" /></Button>}
  onAction={(key) => handleAction(key)}
  aria-label="Actions"
>
  <Menu.Item id="edit" leadingIcon="edit">Edit</Menu.Item>
  <Menu.Item id="copy" leadingIcon="content_copy" trailingText="⌘C">Copy</Menu.Item>
  <Menu.Divider />
  <Menu.Item id="delete" leadingIcon="delete">Delete</Menu.Item>
</Menu>

// Selectable (single):
<Menu
  trigger={<Button>Sort by</Button>}
  selectionMode="single"
  selectedKeys={sortKeys}
  onSelectionChange={setSortKeys}
>
  <Menu.Item id="name">Name</Menu.Item>
  <Menu.Item id="date">Date modified</Menu.Item>
  <Menu.Item id="size">Size</Menu.Item>
</Menu>

// With submenu (RAC SubmenuTrigger, requires RAC ≥ 1.3):
<Menu trigger={<Button>Share</Button>} onAction={handleAction}>
  <Menu.SubmenuItem
    id="export"
    leadingIcon="ios_share"
    submenu={
      <Menu onAction={handleExport}>
        <Menu.Item id="pdf">PDF</Menu.Item>
        <Menu.Item id="png">PNG</Menu.Item>
      </Menu>
    }
  >
    Export
  </Menu.SubmenuItem>
  <Menu.Item id="copy-link" leadingIcon="link">Copy link</Menu.Item>
</Menu>
```

**BEM classes emitted:**
- `.menu` — RAC `Menu` element
- `.menu__container` — inner container (surface + shadow)
- `.menu__list` — scrollable wrapper
- `.menu__item` — RAC `MenuItem`
- `.menu__item--has-leading-icon` — aligns labels when mixing icon/no-icon items
- `.menu__item--has-submenu`
- `.menu__item--has-supporting-text`
- `.menu__item-leading-icon`
- `.menu__item-label`
- `.menu__item-supporting-text`
- `.menu__item-trailing-text`
- `.menu__item-submenu-icon`
- `.menu__divider` — RAC `Separator`
- `.menu__section-header` — RAC `Header`
- `.menu__section` — RAC `MenuSection`

**Deviations / justifications:**
- Compound namespace (`Menu.Item`, `Menu.Section`, `Menu.Divider`, `Menu.SubmenuItem`) keeps discoverability co-located and avoids name collisions with RAC primitives re-exported from `@kafui/react`.
- `trigger` as a prop (not children) makes the compound API unambiguous — no need to wrap in `<Menu.Trigger>` slot; internally wired to RAC `MenuTrigger`.
- Both `onAction` (key-based routing) and item-level `onPress` are supported. `onAction` is the preferred pattern for data-driven menus; item `onPress` covers imperative/async cases.
- `selectionMode` presence gates the ARIA role: absent → `menuitem`; `"single"` → `menuitemradio`; `"multiple"` → `menuitemcheckbox`. RAC handles this natively.
- Submenu uses RAC `SubmenuTrigger` (RAC ≥ 1.3); `trigger={null}` on a nested `<Menu>` signals it is a submenu panel and `Menu.SubmenuItem` wires `SubmenuTrigger` internally.
- State-layer is a `::before` pseudo-element (not a DOM child) — same pattern as `chip` for consistency across all kafUI interactive surfaces.
