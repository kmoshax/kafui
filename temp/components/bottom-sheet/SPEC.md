# Bottom Sheet

A surface that slides up from the bottom edge of the screen, used to present supplementary content or actions. M3 category: **Containment → Bottom Sheets**. Two subtypes: **standard** (persistent, non-modal) and **modal** (blocks background interaction via scrim).

---

## Anatomy / Parts → BEM Elements

| BEM element | Description |
|---|---|
| `.bottom-sheet` | Root container; the sheet surface itself |
| `.bottom-sheet__drag-handle` | Optional 32×4 dp pill at top-center; signals draggability |
| `.bottom-sheet__header` | Optional sticky area for title / close action at top of content |
| `.bottom-sheet__content` | Scrollable content region; grows to fill available detent height |
| `.scrim` | Shared semi-transparent overlay behind modal variant only; sibling in DOM, not child |

The scrim uses the shared `.scrim` class (same as dialog and side sheet) and is rendered as a separate portal node so it does not clip or inherit the sheet's `border-radius`.

---

## Variants

| Variant | `variant` prop | Description |
|---|---|---|
| Standard | `"standard"` | Persistent; rendered inline in layout flow or absolutely at page bottom; does not block background; no scrim; can be hidden/partially shown at any detent |
| Modal | `"modal"` (default) | Overlays content via scrim; background inert; must be dismissed explicitly (swipe-down, button, or scrim-tap); built on React Aria `Modal` + `Dialog` |

---

## Detents (sheet heights)

Detents define snap points as a fraction of the viewport height or a fixed offset.

| Detent name | Typical height | Description |
|---|---|---|
| `peek` | 120–160 dp fixed | Drag handle and minimal content visible; standard variant resting state |
| `half` | 50 vh | Partial expansion |
| `expanded` | `calc(100dvh - env(safe-area-inset-top))` | Fully expanded; top corners animate to 0 at true full-screen |
| `hidden` | 0 / off-screen | Dismissed state (standard variant only; modal is unmounted) |

The active detent is tracked via the `detent` prop. On screens ≥ 600 dp wide the modal sheet can optionally promote to a side sheet (`promoteTo="side-sheet"` prop — cross-component, opt-in).

> **Corner radius at `expanded`:** When the sheet reaches the top of the viewport (`expanded` detent touching the status bar area), the top corner-radius animates from `--corner-extra-large` to `0`. This corner morph uses `--easing-standard` + `--duration-short4` and is driven by a CSS transition on `border-start-start-radius` / `border-start-end-radius`, toggled by the `.bottom-sheet--detent-expanded` modifier.

---

## States

| State | Description |
|---|---|
| Hidden | Sheet is off-screen or `display: none` (standard); modal is unmounted |
| Peek | Resting at `peek` detent |
| Partial | Resting at any non-full detent |
| Expanded | At full detent; top corner radius animates to 0 if touching viewport top |
| Dragging | Pointer/touch active on drag handle or sheet surface; `.bottom-sheet--dragging` disables CSS transition |
| Dismissed (modal) | Scrim fades out; sheet translates off-screen; `aria-modal` removed on unmount |

State layers on the drag handle: hover 8%, focus 10%, pressed 10%, dragged 16%.

---

## Design Tokens

> All custom properties inside `@layer kafui { … }`. Component-internal vars declared at the top of `.bottom-sheet { }`.

### Color

| Role | CSS custom property | Usage |
|------|---------------------|-------|
| `surface-container-low` | `--surface-container-low` | Sheet surface background |
| `on-surface-variant` | `--on-surface-variant` | Drag handle color (at 40% opacity) |
| `scrim` | `--scrim` | Backdrop at 32% (via shared `.scrim` class) |
| `on-surface` | `--on-surface` | Content text |
| `on-surface-variant` | `--on-surface-variant` | Subheading text |

### Shape

| Token | CSS custom property | Usage |
|-------|---------------------|-------|
| `corner-extra-large` (28 dp) | `--corner-extra-large` | Top corners at rest |
| `corner-none` | `--corner-none` | Top corners at `expanded` detent (animated) |

Only top corners are rounded; bottom corners are always 0 (flush with screen edge):

```css
.bottom-sheet {
  border-start-start-radius: var(--corner-extra-large);
  border-start-end-radius:   var(--corner-extra-large);
  border-end-start-radius:   0;
  border-end-end-radius:     0;
}
.bottom-sheet--detent-expanded {
  border-start-start-radius: var(--corner-none);
  border-start-end-radius:   var(--corner-none);
  transition: border-start-start-radius var(--duration-short4) var(--easing-standard),
              border-start-end-radius   var(--duration-short4) var(--easing-standard);
}
```

### Typography

| Scale | CSS custom properties | Usage |
|-------|-----------------------|-------|
| `title-medium` | `--title-medium-size`, etc. | Header title |

Body content typography is consumer-defined.

### Elevation

| Level | CSS custom property | Usage |
|-------|---------------------|-------|
| Level 1 | `--elevation-1` | Both variants (scrim provides depth separation for modal) |

### Motion

| Token | CSS custom property | Usage |
|-------|---------------------|-------|
| Emphasized | `--easing-emphasized` | Enter / expand |
| Emphasized accelerate | `--easing-emphasized-accelerate` | Exit / collapse |
| Long2 (~500 ms) | `--duration-long2` | Enter duration |
| Medium4 (~300 ms) | `--duration-medium4` | Exit duration |
| Standard | `--easing-standard` | Corner morph |
| Short4 (~200 ms) | `--duration-short4` | Corner morph duration |

### State layer opacities

```css
--state-hover:    0.08;
--state-focus:    0.10;
--state-pressed:  0.10;
--state-dragged:  0.16;
```

### Example CSS (illustrative)

```css
@layer kafui {
  .bottom-sheet {
    /* component-internal vars */
    --translate-y: 0px;   /* updated by drag hook via inline CSS var */

    position: fixed;
    inset-inline: 0;
    inset-block-end: 0;
    background: var(--surface-container-low);
    border-start-start-radius: var(--corner-extra-large);
    border-start-end-radius: var(--corner-extra-large);
    border-end-start-radius: 0;
    border-end-end-radius: 0;
    box-shadow: var(--elevation-1);
    transform: translateY(var(--translate-y));
    transition: transform var(--duration-long2) var(--easing-emphasized),
                border-start-start-radius var(--duration-short4) var(--easing-standard),
                border-start-end-radius   var(--duration-short4) var(--easing-standard);
    will-change: transform;
  }

  /* During drag: JS controls position; disable CSS transition */
  .bottom-sheet--dragging {
    transition: none;
  }

  .bottom-sheet--detent-expanded {
    border-start-start-radius: 0;
    border-start-end-radius: 0;
  }

  /* Enter/exit for modal variant via RAC data attributes */
  .bottom-sheet[data-entering] {
    animation: sheet-enter var(--duration-long2) var(--easing-emphasized) both;
  }
  .bottom-sheet[data-exiting] {
    animation: sheet-exit var(--duration-medium4) var(--easing-emphasized-accelerate) both;
  }

  @keyframes sheet-enter {
    from { transform: translateY(100%); }
    to   { transform: translateY(0); }
  }
  @keyframes sheet-exit {
    from { transform: translateY(0); }
    to   { transform: translateY(100%); }
  }

  .bottom-sheet__drag-handle {
    width: 32px;
    height: 4px;
    border-radius: var(--corner-full);
    background: var(--on-surface-variant);
    opacity: 0.4;
    margin-inline: auto;
    margin-block: 22px 0;
    cursor: grab;
  }

  .bottom-sheet__content {
    overflow-y: auto;
    overscroll-behavior: contain;
    padding-block-end: env(safe-area-inset-bottom);
  }

  @media (prefers-reduced-motion: reduce) {
    .bottom-sheet,
    .bottom-sheet[data-entering],
    .bottom-sheet[data-exiting] {
      transition: none;
      animation: none;
    }
  }
}
```

---

## Interaction & Accessibility

### Drag / swipe

- Pointer-down on drag handle (or optionally sheet body — `@media (pointer: coarse)`) initiates drag.
- `pointermove` updates `--translate-y` CSS custom property via `element.style.setProperty`; zero layout reflow.
- On pointer-up: compute velocity from position delta / time; snap to nearest detent (fast flick → next detent in flick direction; slow release → geometrically closest detent).
- `@media (pointer: fine)` (mouse): restrict drag to drag handle only.
- `@media (pointer: coarse)` (touch): allow full-surface drag.

### Keyboard

- Modal: `Tab` cycles focus within sheet (RAC `Dialog` focus scope); `Escape` dismisses (fires `onOpenChange(false)`).
- Standard: no forced focus trap; `Escape` collapses to `peek` or `hidden`.
- Drag handle (when `role="slider"`): `ArrowUp` → next expanded detent; `ArrowDown` → next collapsed detent; `Home` → `expanded`; `End` → `hidden` / `peek`.

### Focus management

- On open (modal): focus moves to the first focusable element inside `__content` (or the drag handle if nothing else is focusable).
- On close: focus returns to the element that triggered the sheet (RAC `useOverlayTrigger` / `DialogTrigger` handles this).

### ARIA

- **Modal variant:** root is `role="dialog"` + `aria-modal="true"` + `aria-labelledby` pointing to `__header` title id.
- **Standard variant:** `role="region"` + `aria-label` or `aria-labelledby` (accessible landmark, not modal).
- **Drag handle:** `role="slider"` (preferred over `role="button"` for detent state exposure); `aria-label="Sheet size"` (or i18n equivalent via `dragHandleLabel` prop); `aria-valuenow` = current detent index; `aria-valuemin` = 0; `aria-valuemax` = detents.length - 1; `aria-valuetext` = detent name (e.g. "expanded").
- **Scrim:** `aria-hidden="true"`; RAC `ModalOverlay` sets `aria-hidden` on the application root automatically.

> **`role="slider"` on drag handle vs `role="button"`:** `role="slider"` better communicates the graduated detent state to AT (aria-valuenow/valuetext announces current position). `role="button"` only communicates "activate." Use `role="slider"` consistently across bottom-sheet and any future resizable panel.

### Reduced motion

```css
@media (prefers-reduced-motion: reduce) {
  .bottom-sheet { transition: none; animation: none; }
}
```

Sheet appears/disappears instantly; drag gesture remains fully functional.

### RTL

Drag handle is centered (unaffected by direction). Sheet origin is always bottom edge; no inline-direction dependency. `inset-inline: 0` covers both LTR and RTL.

---

## Proposed kafUI React API

```tsx
// Modal: ModalOverlay + Modal + Dialog (react-aria-components)
// Standard: custom controlled component; no RAC modal

type DetentKey = 'hidden' | 'peek' | 'half' | 'expanded' | (string & {});

interface BottomSheetProps {
  variant?: 'standard' | 'modal';           // default: 'modal'
  isOpen?: boolean;                          // controlled open state
  defaultOpen?: boolean;                     // uncontrolled
  onOpenChange?: (isOpen: boolean) => void;  // RAC convention
  detent?: DetentKey;                        // controlled detent
  defaultDetent?: DetentKey;                 // default: 'peek'
  onDetentChange?: (detent: DetentKey) => void;
  detents?: DetentKey[];                     // available snap points; default: ['hidden','peek','expanded']
  showDragHandle?: boolean;                  // default: true
  dragHandleLabel?: string;                  // aria-label for drag handle slider; default: 'Sheet size'
  isDismissable?: boolean;                   // scrim tap / swipe-down dismisses; default: true (modal)
  promoteTo?: 'side-sheet';                  // render as SideSheet above 600 dp breakpoint (opt-in)
  'aria-label'?: string;
  'aria-labelledby'?: string;
  children: React.ReactNode;
  className?: string;
}

// Compound sub-components
<BottomSheet variant="modal" isOpen={open} onOpenChange={setOpen} defaultDetent="peek">
  <BottomSheet.Header>Share</BottomSheet.Header>
  <BottomSheet.Content>…</BottomSheet.Content>
</BottomSheet>

// With explicit trigger (modal)
<BottomSheet.Trigger>
  <Button variant="filled">Open</Button>
  <BottomSheet variant="modal">
    <BottomSheet.Header>Options</BottomSheet.Header>
    <BottomSheet.Content>…</BottomSheet.Content>
  </BottomSheet>
</BottomSheet.Trigger>
```

**BEM classes emitted:**

```
.bottom-sheet                       root
.bottom-sheet--modal                variant="modal"
.bottom-sheet--standard             variant="standard"
.bottom-sheet--detent-{key}         e.g. .bottom-sheet--detent-expanded
.bottom-sheet--dragging             during active drag gesture
.bottom-sheet__drag-handle          handle pill
.bottom-sheet__header               optional header region
.bottom-sheet__content              scrollable body
```

Shared `.scrim` class on the portal sibling (modal only). RAC `data-entering` / `data-exiting` on the modal root for animation coordination.

**React Aria base:**

- Modal variant: `ModalOverlay` + `Modal` + `Dialog` from `react-aria-components`; `useOverlayTrigger` for trigger wiring.
- Standard variant: custom hook `useBottomSheet` managing detent state + pointer events; no RAC Modal (non-inert by design).

**Deviations from generic drawer patterns:**

- Detents instead of a single `open` boolean — faithful to M3; no MUI equivalent.
- Standard variant uses no RAC Modal (M3 specifies it is non-modal and non-blocking).
- Drag velocity computed via `pointermove` delta/time — no third-party gesture library.
- CSS custom property `--translate-y` on the element drives drag position without layout reflow.
- `promoteTo="side-sheet"` is opt-in; never automatic (predictability over magic).

**Open decisions:**

- [ ] `useBottomSheet` hook: expose publicly as `@kafui/react/use-bottom-sheet` for headless consumers building custom sheet UIs.
- [ ] Drag on `__content` (scroll vs drag conflict): use `touchAction: 'none'` on drag handle; allow native scroll in `__content`; only intercept drag when pointer is on handle (or when `@media (pointer: coarse)` and content is scrolled to top).
- [ ] `promoteTo="side-sheet"`: should this use a `ResizeObserver` on the viewport (not `window.innerWidth`) to react to container queries?
