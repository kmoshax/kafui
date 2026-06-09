# Side Sheet

**Purpose:** A supplementary panel that slides in from the trailing (inline-end) edge, surfacing detail or contextual content without fully blocking the main view. M3 category: **Containment → Side Sheets**. **Distinct from Navigation Drawer**, which slides from the leading edge and contains navigation links.

---

## Anatomy / Parts → BEM Elements

```
.scrim                              shared full-viewport backdrop (modal variant only; portal sibling, NOT child)
.side-sheet                         root panel; slides from inline-end edge; elevation 1 (standard) or 2 (modal)
.side-sheet__header                 top region; contains close icon + title + optional header actions
.side-sheet__close-btn              leading icon button in header; aria-label="Close" (or closeLabel prop)
.side-sheet__title                  header title text; title-large typescale
.side-sheet__header-actions         trailing icons or buttons in header (optional)
.side-sheet__divider                separator below header; visible when header is present
.side-sheet__content                scrollable body region; flex column; overflow-y: auto
.side-sheet__actions                optional bottom action bar (standard variant; 1–2 buttons)
```

The scrim uses the shared `.scrim` class (same as dialog and bottom sheet). It is rendered as a separate portal node so it does not inherit or clip the sheet's border-radius.

---

## Variants

| Variant | `variant` prop | Description |
|---|---|---|
| Standard (persistent) | `"standard"` (default) | Inline; shares layout with main content (CSS grid/flex); no scrim; always visible when open; elevation 1; width 360 dp typical |
| Modal | `"modal"` | Overlays main content; scrim behind; dismissible by scrim tap, close button, or Esc; focus-trapped; elevation 2 |

**Standard side sheet:** The parent layout must allocate space (CSS grid or flex column). The sheet has `position: relative` / `position: sticky` in the layout flow. No focus trap. Close button in header dismisses.

**Modal side sheet:** `position: fixed; inset-block: 0; inset-inline-end: 0`. Scrim covers the rest of the viewport.

---

## States

| State | Behavior |
|---|---|
| Closed | Translated off-screen or `display: none`; no focus trap |
| Opening | Slide in from inline-end + optional fade; `--easing-emphasized-decelerate`; `--duration-medium2` |
| Open | Modal: focus trapped; scrim visible; scroll lock on `<body>`. Standard: no trap; layout reflows |
| Closing | Slide out to inline-end + optional fade; `--easing-emphasized-accelerate`; `--duration-short4` |
| Scrim pressed | Modal only: fires `onOpenChange(false)` |

---

## Design Tokens

> All custom properties inside `@layer kafui { … }`. Component-internal vars declared at the top of `.side-sheet { }`.

### Color

| Role | CSS custom property | Usage |
|------|---------------------|-------|
| `surface-container-low` | `--surface-container-low` | Sheet surface background |
| `on-surface` | `--on-surface` | Header title text |
| `on-surface-variant` | `--on-surface-variant` | Header icon color |
| `scrim` | `--scrim` | Backdrop (32%, via shared `.scrim` class) |
| `outline-variant` | `--outline-variant` | Divider below header |

### Shape

| Token | CSS custom property | Usage |
|-------|---------------------|-------|
| `corner-extra-large` (28 dp) | `--corner-extra-large` | Inline-start corners only |
| `corner-none` (0) | implied `0` | Inline-end corners always 0 — sheet is flush with viewport edge |

Only the two inline-start corners are rounded; inline-end corners are always 0:

```css
.side-sheet {
  border-start-start-radius: var(--corner-extra-large);
  border-end-start-radius:   var(--corner-extra-large);
  border-start-end-radius:   0;
  border-end-end-radius:     0;
}
```

In RTL (`dir="rtl"`), `inset-inline-end: 0` places the sheet at the left edge, and `border-start-start-radius` / `border-end-start-radius` become the right-side corners — logical properties handle this automatically.

### Typography

| Scale | CSS custom properties | Usage |
|-------|-----------------------|-------|
| `title-large` | `--title-large-size`, `--title-large-line-height`, `--title-large-weight` | Header title |

Body content typography is consumer-defined; the sheet provides layout container only.

### Elevation

| Level | CSS custom property | Usage |
|-------|---------------------|-------|
| Level 1 | `--elevation-1` | Standard — subtle shadow on inline-start edge |
| Level 2 | `--elevation-2` | Modal |

### Motion

| Token | CSS custom property | Usage |
|-------|---------------------|-------|
| Emphasized decelerate | `--easing-emphasized-decelerate` | Enter easing |
| Emphasized accelerate | `--easing-emphasized-accelerate` | Exit easing |
| Medium2 (~300 ms) | `--duration-medium2` | Enter duration |
| Short4 (~200 ms) | `--duration-short4` | Exit duration |

### Width

```css
.side-sheet {
  --w: 360px;    /* component-internal var; override at any scope */
  width: min(var(--w), 100vw);
}
```

Consumers override with `.side-sheet { --w: 400px; }` at any scope — no `--kafui-*` prefix needed (the var lives inside the layer).

### Example CSS (illustrative)

```css
@layer kafui {
  .side-sheet {
    /* component-internal vars */
    --w: 360px;

    position: fixed;
    inset-block: 0;
    inset-inline-end: 0;
    width: min(var(--w), 100vw);
    background: var(--surface-container-low);
    border-start-start-radius: var(--corner-extra-large);
    border-end-start-radius:   var(--corner-extra-large);
    border-start-end-radius:   0;
    border-end-end-radius:     0;
    box-shadow: var(--elevation-2);
    display: flex;
    flex-direction: column;
    z-index: var(--z-modal, 1300);
    outline: none;
  }

  .side-sheet--standard {
    position: relative;  /* or sticky — layout-dependent; consumer wraps in grid */
    box-shadow: var(--elevation-1);
    z-index: auto;
  }

  /* Enter/exit for modal variant */
  .side-sheet[data-entering] {
    animation: side-sheet-enter var(--duration-medium2) var(--easing-emphasized-decelerate) both;
  }
  .side-sheet[data-exiting] {
    animation: side-sheet-exit var(--duration-short4) var(--easing-emphasized-accelerate) both;
  }

  @keyframes side-sheet-enter {
    from { translate: 100% 0; }
    to   { translate: 0 0; }
  }
  @keyframes side-sheet-exit {
    from { translate: 0 0; }
    to   { translate: 100% 0; }
  }

  /* RTL: logical translate — CSS translate (individual property) respects writing mode.
     `translate: 100% 0` always moves in the inline-end direction, so it works for
     both LTR (right) and RTL (left) without a [dir="rtl"] selector. */

  .side-sheet__header {
    display: flex;
    align-items: center;
    gap: 4px;
    padding: 16px;
    flex-shrink: 0;
  }

  .side-sheet__title {
    flex: 1;
    color: var(--on-surface);
    font-size: var(--title-large-size);
    line-height: var(--title-large-line-height);
  }

  .side-sheet__close-btn {
    color: var(--on-surface-variant);
    /* 48px minimum touch target via padding */
  }

  .side-sheet__header-actions {
    display: flex;
    gap: 4px;
    margin-inline-start: auto;
  }

  .side-sheet__divider {
    border-block-end: 1px solid var(--outline-variant);
    flex-shrink: 0;
  }

  .side-sheet__content {
    flex: 1;
    overflow-y: auto;
    overscroll-behavior: contain;
    padding: 16px;
  }

  .side-sheet__actions {
    flex-shrink: 0;
    padding: 16px;
    display: flex;
    gap: 8px;
    justify-content: flex-end;
  }

  @media (prefers-reduced-motion: reduce) {
    .side-sheet[data-entering],
    .side-sheet[data-exiting] {
      animation-duration: 1ms;
    }
    @keyframes side-sheet-enter { from { opacity: 0; } to { opacity: 1; } }
    @keyframes side-sheet-exit  { from { opacity: 1; } to { opacity: 0; } }
  }
}
```

> **RTL note:** The `translate` CSS property (individual transform, not shorthand) is direction-aware when the value is a percentage: `translate: 100% 0` always moves the element in the inline-end direction. In LTR that is right; in RTL that is left. No `[dir="rtl"]` overrides needed.

---

## Interaction & Accessibility

**Focus trap (modal only):** On open, focus moves to the first focusable element inside the sheet (typically the close button). RAC `Modal` + `Dialog` provides this. On close, focus returns to the element that triggered opening.

**Escape (modal only):** `Esc` fires `onOpenChange(false)`. RAC `ModalOverlay` handles this.

**Scroll lock (modal only):** `<body>` overflow locked while modal sheet is open. RAC `ModalOverlay` via built-in `ScrollLock`.

**Standard variant — no focus trap:** Tab moves freely through the document including the sheet. The close button in the header is the primary dismiss mechanism. No scroll lock.

**ARIA (modal):**

- Sheet container: `role="dialog"` + `aria-modal="true"` (from RAC `Dialog`)
- `aria-labelledby` → `.side-sheet__title` id (auto-generated via `useId`)
- Scrim: `aria-hidden="true"` (set by RAC `ModalOverlay`)
- Close button: `aria-label` = `closeLabel` prop (default `"Close"`)

**ARIA (standard):**

- Sheet container: `role="complementary"` + `aria-labelledby` pointing to title id
- NOT modal — no `aria-modal`, no focus trap
- `role="complementary"` exposes the sheet as a landmark; AT users can navigate to it via landmark shortcuts

**Keyboard (both variants):**

- Close button in header is a standard `<button>`; `Space`/`Enter` call `onOpenChange(false)`.
- Modal: `Tab`/`Shift+Tab` cycles within sheet; `Esc` closes.

**RTL:** `inset-inline-end: 0` places sheet at the correct edge in both LTR and RTL. `translate: 100% 0` on the `translate` property handles animation direction. Logical border-radius applies to the correct corners automatically.

**Reduced motion:** `@media (prefers-reduced-motion: reduce)` → opacity-only fade; `animation-duration: 1ms` on `[data-entering]`/`[data-exiting]`.

---

## Proposed kafUI React API

```tsx
// Modal: DialogTrigger + ModalOverlay + Modal + Dialog (react-aria-components)
// Standard: plain controlled; <section role="complementary">

type SideSheetVariant = 'standard' | 'modal';

interface SideSheetProps {
  variant?: SideSheetVariant;            // default: 'standard'
  title: string;                         // required; header title + aria-labelledby source
  isOpen?: boolean;                      // controlled
  defaultOpen?: boolean;                 // uncontrolled
  onOpenChange?: (open: boolean) => void;
  closeLabel?: string;                   // aria-label for close button; default 'Close'
  closeIcon?: 'close' | 'back';          // icon to show in close button; default 'close'
  headerActions?: React.ReactNode;       // trailing header slot (icon buttons)
  actions?: React.ReactNode;             // bottom action bar (standard variant)
  isDismissable?: boolean;               // modal: scrim click closes; default true
  children: React.ReactNode;             // body content
  className?: string;
}

interface SideSheetTriggerProps {        // modal variant
  children: [React.ReactElement, React.ReactElement]; // [trigger, <SideSheet>]
  isOpen?: boolean;
  defaultOpen?: boolean;
  onOpenChange?: (open: boolean) => void;
}

// Compound alias
SideSheet.Trigger = SideSheetTrigger;

// Modal usage
<SideSheet.Trigger>
  <Button variant="filled">Open details</Button>
  <SideSheet variant="modal" title="Item details">
    <p>Detail content here</p>
  </SideSheet>
</SideSheet.Trigger>

// Standard usage (caller controls isOpen)
<SideSheet
  variant="standard"
  title="Filters"
  isOpen={filtersOpen}
  onOpenChange={setFiltersOpen}
  actions={<Button variant="filled" onPress={applyFilters}>Apply</Button>}
>
  <FilterForm />
</SideSheet>
```

**BEM classes emitted:**

```
.side-sheet                         always
.side-sheet--standard               variant="standard"
.side-sheet--modal                  variant="modal"
.side-sheet--open                   CSS-targetable open state (for standard variant transitions)
```

RAC `data-entering` / `data-exiting` on the modal root for keyframe animation coordination.

**React Aria base:**

- Modal: `DialogTrigger` → `ModalOverlay` (scrim, scroll lock) → `Modal` (positioning, portal) → `Dialog` (role, focus trap, Esc). Sheet panel is the `Dialog` child.
- Standard: plain React controlled component; `<section role="complementary">` with `aria-labelledby`; no RAC wrapper.

**Justifications vs MUI `Drawer`:**

- `anchor` prop removed — M3 side sheet is always trailing-edge. Navigation Drawer is always leading-edge. They are separate components with separate semantics. Allowing `anchor="left"` on a side sheet would be incorrect M3 usage.
- No `PaperProps` — shape and surface token are not overridable per-instance per M3; CSS var override at any scope is the escape hatch.
- `standard` vs `modal` is a clear semantic `variant` prop; replaces MUI's `temporary`/`permanent`/`persistent` which conflate layout position with modal behavior.
- `closeIcon` prop accepts `'close' | 'back'` — addresses the M3 multi-panel detail view pattern where a back arrow is appropriate.
- `SideSheet.Trigger` compound for modal follows RAC idiom; avoids `open`/`onClose` prop-drilling.

**Open decisions:**

- [ ] Standard variant layout: provide `<SideSheetLayout>` CSS grid helper (main + sheet columns) or leave layout to consumer? A helper would reduce boilerplate significantly.
- [ ] Width breakpoint: M3 recommends side sheet only on medium+ viewports. Should the modal variant auto-convert to a bottom sheet on compact viewports? Recommend opt-in `promoteTo="bottom-sheet"` (mirrors bottom sheet's `promoteTo="side-sheet"`).
- [ ] Standard animation: width transition (avoids reflow but requires absolute positioning) vs `translate` (correct M3 motion but causes layout shift). M3 spec shows translate; recommend translate with `position: absolute` in a `SideSheetLayout` wrapper.
- [ ] `closeIcon` default: M3 shows `×` (close) for modal sheets and `←` (back) for standard detail panels. Consider auto-defaulting based on `variant`: modal → `'close'`, standard → `'back'`.
