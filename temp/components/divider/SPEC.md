# Divider

**Purpose:** A thin line that groups content into sections or separates distinct list items, creating visual hierarchy. M3 category: **Containment → Divider**.

---

## Anatomy / Parts → BEM Elements

```
.divider      root element; renders as <hr> (horizontal) or <div role="separator" aria-orientation="vertical">
```

The divider has no child elements — it is a single self-contained line. Inset and middle-inset variants are controlled via CSS margin/padding on the root, not additional DOM nodes.

---

## Variants

| Variant | `variant` prop | Inset (inline-start) | Inset (inline-end) | Use |
|---|---|---|---|---|
| Full-width | `"full-width"` (default) | 0 | 0 | Between full-width list items or page sections |
| Inset | `"inset"` | 72 dp (aligns with list text column after avatar/icon + padding) | 0 | Below list items with leading icons/avatars |
| Middle inset | `"middle-inset"` | 16 dp | 16 dp | Floating between cards or sections with symmetric margins |

### Orientation
Controlled by `orientation` prop (`"horizontal"` default / `"vertical"`). Vertical dividers appear inside flex/grid rows (e.g. between toolbar actions or adjacent columns).

---

## States

Dividers are **not interactive** and carry no state layers. They are purely presentational. No hover, focus, pressed, or disabled states.

---

## Design Tokens

| Token | CSS var | Usage |
|---|---|---|
| Line color | `--outline-variant` | Adapts automatically via `light-dark()` — no dark-mode override needed |
| Line thickness | `1px` (hardcoded per M3 spec) | Stroke width |

No shape, typography, or elevation tokens. No state tokens.

**Inset distances** are not M3 tokens — they derive from the list leading-element column width (icon: 24 dp + 16 dp padding = 40 dp; avatar: 40 dp + 16 dp = 56 dp; image: 56 dp + 16 dp = 72 dp). kafUI exposes component-scoped CSS custom properties `--inset-start` and `--inset-end` so consumers can override without creating a new variant:

```css
.divider {
  --inset-start: 72px;  /* default inset; consumer overrides per context */
  --inset-end: 0px;
}
```

---

## Interaction & Accessibility

**ARIA:**
- Horizontal `<hr>` carries implicit `role="separator"` and `aria-orientation="horizontal"` — no explicit attributes needed.
- Vertical divider renders as `<div role="separator" aria-orientation="vertical">` since `<hr>` is a block-level void element.
- Purely decorative dividers that add no structural grouping for AT should set `aria-hidden="true"`. The `decorative` prop controls this. Default is `false` (structural) — be explicit.

**Keyboard:** Non-interactive; not focusable; not in tab order.

**RTL:** `margin-inline-start` and `margin-inline-end` ensure insets flip automatically under `dir="rtl"`. No JS needed.

**Reduced motion:** No animations — not applicable.

---

## CSS Architecture

```css
@layer kafui {
  .divider {
    /* Component-scoped vars — consumer overrides these, not token vars */
    --inset-start: 0px;
    --inset-end: 0px;

    /* Reset <hr> browser defaults */
    border: none;
    margin: 0;
    padding: 0;
    background: var(--outline-variant);
  }

  /* Horizontal (default) */
  .divider--horizontal {
    display: block;
    block-size: 1px;
    inline-size: auto;
    margin-inline-start: var(--inset-start);
    margin-inline-end: var(--inset-end);
  }

  /* Vertical */
  .divider--vertical {
    display: inline-block;
    inline-size: 1px;
    block-size: auto;
    align-self: stretch;
    margin-block-start: var(--inset-start);
    margin-block-end: var(--inset-end);
  }

  /* Variant: inset */
  .divider--inset {
    --inset-start: 72px;
  }

  /* Variant: middle-inset */
  .divider--middle-inset {
    --inset-start: 16px;
    --inset-end: 16px;
  }
}
```

> The `--inset-start` / `--inset-end` vars are component-scoped (defined inside `.divider`), not global tokens. This is correct per naming conventions: global tokens use bare role names (`--outline-variant`); component-local tuning vars use short unprefixed names inside the component block.

---

## Proposed kafUI React API

```tsx
// React Aria primitive: Separator (from react-aria-components)
// RAC Separator renders <hr> for horizontal, <div role="separator"> for vertical,
// and handles aria-orientation automatically.

type DividerVariant = 'full-width' | 'inset' | 'middle-inset';
type DividerOrientation = 'horizontal' | 'vertical';

interface DividerProps {
  variant?: DividerVariant;           // default: 'full-width'
  orientation?: DividerOrientation;   // default: 'horizontal'
  /**
   * When true, adds aria-hidden="true" — tells assistive technology to skip.
   * Use for purely visual spacing dividers that carry no structural meaning.
   * Default: false (structural separator).
   */
  decorative?: boolean;
  className?: string;
}

// Usage:
<Divider />
<Divider variant="inset" />
<Divider variant="middle-inset" />
<Divider orientation="vertical" />
<Divider decorative />
```

**BEM classes emitted:**
- `.divider` (always)
- `.divider--full-width` / `.divider--inset` / `.divider--middle-inset`
- `.divider--horizontal` / `.divider--vertical`

**React Aria base:** `Separator` from `react-aria-components`. It selects `<hr>` vs `<div role="separator">` and sets `aria-orientation` correctly. kafUI wraps it, merging BEM classNames and passing `aria-hidden` when `decorative`.

**Justifications vs MUI:**
- No `light` prop (MUI's lighter color variant) — M3 uses a single `--outline-variant` token; "lighter" is done by adjusting the token at scope, not a boolean prop.
- No `flexItem` prop — vertical dividers in flex containers need `align-self: stretch`; this is a one-line consumer CSS concern and keeping it out of the API keeps `Divider` at 3 props total.
- No `textAlign` / `children` support (MUI allows text-labeled dividers) — M3 spec has no text-in-divider pattern; this would be a separate "section label" component if ever needed.
- `decorative` is explicit with a clear doc comment — intent is unambiguous vs. MUI's undocumented `aria-hidden` behavior.
- `--inset-start` / `--inset-end` are consumer-overridable via CSS, removing the need for numeric `insetStart`/`insetEnd` props on the component.
