# Dialog

**Purpose:** Surfaces critical information or requests a decision that requires user input before proceeding. Blocks interaction with underlying content until dismissed. M3 category: **Containment → Dialogs**.

---

## Anatomy / Parts → BEM Elements

```
.scrim                              full-viewport backdrop (shared pattern; outside dialog DOM)
.dialog                             root container; elevation 3; M3 shape extra-large
.dialog__icon                       optional hero icon above headline (basic only)
.dialog__headline                   required — title text; headline-small typescale
.dialog__supporting-text            scrollable body copy region; body-medium typescale
.dialog__divider                    optional separator between supporting-text and actions (shown when scrolled)
.dialog__actions                    button row; filled + text, or two text buttons
```

Full-screen variant replaces the floating container with a full-viewport sheet:

```
.dialog--full-screen                modifier on .dialog
.dialog__header                     top app-bar-style header: close/back icon + headline + confirm CTA
.dialog__content                    scrollable body (replaces supporting-text + actions in full-screen)
```

> The scrim element uses the shared `.scrim` class (not `.dialog__scrim`) because it is a sibling in the portal, not a child of `.dialog`. All modal overlay components (dialog, modal bottom sheet, modal side sheet) share this class and its token-driven opacity.

---

## Variants

| Variant | `variant` prop | Description |
|---|---|---|
| Basic | `"basic"` (default) | Floating card; up to two text or filled+text buttons in actions row; optional hero icon; max-width 560 dp |
| Full-screen | `"full-screen"` | Fills viewport (mobile) or large pane (desktop); header with close and confirm; used for complex forms or long content |

---

## States

| State | Behavior |
|---|---|
| Closed | Not rendered (RAC unmounts by default); no focus trap |
| Opening | Enter transition: scale 0.8→1 + fade-in; `--easing-emphasized-decelerate`; `--duration-medium2` |
| Open | Focus trapped inside; scrim at 32% `--scrim` token; scroll lock on `<body>` |
| Closing | Exit transition: scale 1→0.8 + fade-out; `--easing-emphasized-accelerate`; `--duration-short4` |
| Scrim pressed | Closes modal dialog; fires `onOpenChange(false)` |
| Disabled action | Follows button disabled rules; dialog remains open |

---

## Design Tokens

> All custom properties declared inside `@layer kafui { … }`.
> Component-internal vars declared at the top of the `.dialog { }` block.

### Color

| Role | CSS custom property | Usage |
|------|---------------------|-------|
| `surface-container-high` | `--surface-container-high` | Container background |
| `on-surface` | `--on-surface` | Headline text |
| `on-surface-variant` | `--on-surface-variant` | Supporting text |
| `secondary` | `--secondary` | Hero icon color (basic) |
| `scrim` | `--scrim` | Backdrop color at 32% opacity |
| `outline-variant` | `--outline-variant` | Divider color |

### Shape

| Token | CSS custom property | Usage |
|-------|---------------------|-------|
| `corner-extra-large` (28 dp) | `--corner-extra-large` | Basic dialog border-radius |
| `corner-none` (0) | `--corner-none` | Full-screen dialog border-radius |

### Typography

| Scale | CSS custom properties | Usage |
|-------|-----------------------|-------|
| `headline-small` | `--headline-small-size`, `--headline-small-line-height`, `--headline-small-weight` | Headline (basic) |
| `title-large` | `--title-large-size`, etc. | Headline (full-screen header) |
| `body-medium` | `--body-medium-size`, etc. | Supporting text |

### Elevation

| Level | CSS custom property | Usage |
|-------|---------------------|-------|
| Level 3 | `--elevation-3` | Basic dialog box-shadow |
| Level 0 | `--elevation-0` | Full-screen (flush, no shadow) |

### Motion

| Token | CSS custom property | Usage |
|-------|---------------------|-------|
| Emphasized decelerate | `--easing-emphasized-decelerate` | Enter easing |
| Emphasized accelerate | `--easing-emphasized-accelerate` | Exit easing |
| Medium2 (~300 ms) | `--duration-medium2` | Enter duration |
| Short4 (~200 ms) | `--duration-short4` | Exit duration |

### Example CSS (illustrative)

```css
@layer kafui {
  .scrim {
    position: fixed;
    inset: 0;
    background: color-mix(in srgb, var(--scrim) 32%, transparent);
    z-index: var(--z-modal-scrim, 1200);
  }

  .dialog {
    /* component-internal vars */
    --max-w: 560px;
    --min-w: 280px;
    --pad: 24px;
    --radius: var(--corner-extra-large);

    position: relative;
    background: var(--surface-container-high);
    border-radius: var(--radius);
    box-shadow: var(--elevation-3);
    min-width: var(--min-w);
    max-width: var(--max-w);
    width: calc(100% - 48px);
    padding: var(--pad);
    color: var(--on-surface);
    outline: none;
  }

  .dialog--full-screen {
    --radius: var(--corner-none);
    max-width: 100dvw;
    width: 100dvw;
    height: 100dvh;
    border-radius: var(--radius);
    box-shadow: var(--elevation-0);
    padding: 0;
  }

  /* RAC data attributes drive enter/exit */
  .dialog[data-entering] {
    animation: dialog-enter var(--duration-medium2) var(--easing-emphasized-decelerate) both;
  }
  .dialog[data-exiting] {
    animation: dialog-exit var(--duration-short4) var(--easing-emphasized-accelerate) both;
  }

  @keyframes dialog-enter {
    from { opacity: 0; transform: scale(0.8); }
    to   { opacity: 1; transform: scale(1); }
  }
  @keyframes dialog-exit {
    from { opacity: 1; transform: scale(1); }
    to   { opacity: 0; transform: scale(0.8); }
  }

  .dialog--scrolled .dialog__divider {
    visibility: visible;
  }
  .dialog__divider {
    visibility: hidden;
    border-block-end: 1px solid var(--outline-variant);
  }

  .dialog__supporting-text {
    overflow-y: auto;
    overscroll-behavior: contain;
    color: var(--on-surface-variant);
    font-size: var(--body-medium-size);
  }

  .dialog__actions {
    display: flex;
    flex-wrap: wrap;
    justify-content: flex-end;
    gap: 8px;
    padding-block-start: 24px;
  }

  .dialog__header {
    display: flex;
    align-items: center;
    gap: 4px;
    padding: 4px 4px 0;
    flex-shrink: 0;
  }

  @media (prefers-reduced-motion: reduce) {
    .dialog[data-entering],
    .dialog[data-exiting] {
      animation-duration: 1ms;
      /* transform removed; opacity only */
    }
    @keyframes dialog-enter { from { opacity: 0; } to { opacity: 1; } }
    @keyframes dialog-exit  { from { opacity: 1; } to { opacity: 0; } }
  }
}
```

---

## Interaction & Accessibility

**Focus trap:** On open, focus moves to the first focusable element inside `.dialog` (or the headline if nothing else is focusable). React Aria's `Modal` + `Dialog` provides this automatically. On close, focus returns to the trigger element (`DialogTrigger` preserves the reference).

**Escape:** Pressing `Esc` calls `onOpenChange(false)` and closes the dialog. RAC `ModalOverlay` handles this natively.

**Scroll lock:** `<body>` overflow is locked while any modal dialog is open. RAC `ModalOverlay` via its built-in `ScrollLock`.

**ARIA:**

- Root receives `role="dialog"` and `aria-modal="true"` (from RAC `Dialog`).
- `aria-labelledby` points to `.dialog__headline`'s id (auto-generated via `useId`).
- `aria-describedby` points to `.dialog__supporting-text`'s id when present; **omitted** when no supporting text.
- Scrim/overlay: `aria-hidden="true"` (non-interactive backdrop; RAC `ModalOverlay` sets this).
- Full-screen close button: `aria-label="Close"` (or i18n equivalent via `closeLabel` prop).

**Keyboard navigation:**

- `Tab`/`Shift+Tab` cycles within the dialog; RAC focus trap ensures no escape.
- Action buttons are standard `<button>` elements — `Space`/`Enter` activate.
- Full-screen confirm CTA in the header is a button reachable by `Tab`.

**RTL:** Logical properties throughout (`padding-inline`, `inset-inline`). Action buttons in full-screen header use `margin-inline-start: auto` to push to inline-end.

**Reduced motion:** `@media (prefers-reduced-motion: reduce)` → skip scale transform; opacity-only fade at half duration.

---

## Proposed kafUI React API

```tsx
// React Aria primitives: DialogTrigger, Modal, ModalOverlay, Dialog (react-aria-components)

type DialogVariant = 'basic' | 'full-screen';

interface DialogProps {
  variant?: DialogVariant;            // default: 'basic'
  headline: string;                   // required; maps to aria-labelledby target
  icon?: string;                      // hero icon name (basic only); rendered as <Icon name="..." />
  children?: React.ReactNode;         // supporting text and/or form content
  actions?: React.ReactNode;          // basic: 1-2 <Button> elements
  isDismissable?: boolean;            // scrim click closes; default true (basic), false (full-screen)
  onOpenChange?: (open: boolean) => void;
  className?: string;
}

interface DialogTriggerProps {
  children: [React.ReactElement, React.ReactElement]; // [trigger, <Dialog>]
  isOpen?: boolean;                   // controlled
  defaultOpen?: boolean;              // uncontrolled
  onOpenChange?: (open: boolean) => void;
}

// Compound convenience alias (co-located on the Dialog export)
Dialog.Trigger = DialogTrigger;

// Usage — uncontrolled
<Dialog.Trigger>
  <Button variant="text">Open</Button>
  <Dialog
    headline="Confirm deletion"
    actions={
      <>
        <Button variant="text">Cancel</Button>
        <Button variant="text">Delete</Button>
      </>
    }
  >
    Are you sure you want to delete this item? This cannot be undone.
  </Dialog>
</Dialog.Trigger>

// Usage — controlled
const [open, setOpen] = useState(false);
<>
  <Button onPress={() => setOpen(true)}>Open</Button>
  <Dialog.Trigger isOpen={open} onOpenChange={setOpen}>
    {/* DialogTrigger with controlled isOpen still needs a trigger placeholder */}
    <span />
    <Dialog headline="Settings">…</Dialog>
  </Dialog.Trigger>
</>
```

> **Design decision — controlled pattern:** When `isOpen` is passed to `Dialog.Trigger`, a placeholder trigger element is still required by RAC's `DialogTrigger`. For the fully-controlled case (no trigger UI), consider a `DialogPortal` escape hatch that renders `ModalOverlay` directly without `DialogTrigger`. This is an open question — see TODO.

**BEM classes emitted:**

```
.dialog                             always
.dialog--basic                      variant="basic" (default)
.dialog--full-screen                variant="full-screen"
.dialog--has-icon                   when icon prop is provided
.dialog--scrolled                   added dynamically when __supporting-text is scrolled
```

RAC data attributes on the `<Modal>` / `<Dialog>` root: `data-entering`, `data-exiting` — drive keyframe animations.

**React Aria base:** `DialogTrigger` → `ModalOverlay` (scrim, `aria-hidden`, scroll lock) + `Modal` (positioning, portal) + `Dialog` (`role`, `aria-modal`, focus trap, Esc). The kafUI wrapper injects BEM classNames and M3 structure.

**Justifications vs MUI:**

- No `maxWidth` / `fullWidth` boolean props — M3 specifies a fixed max-width (560 dp) and a distinct `full-screen` variant; sizing is token-driven.
- No `TransitionComponent` prop — motion is CSS-only via RAC `data-entering`/`data-exiting`.
- `isDismissable` replaces MUI's `disableBackdropClick` (inverted, positive semantics).
- Compound `Dialog.Trigger` + `Dialog` instead of MUI's `open`/`onClose` on Dialog root — matches RAC idiom, avoids prop-drilling, keeps trigger co-located with dialog.
- `actions` as `ReactNode` slot now; `Dialog.Actions` compound child is a valid future upgrade (see TODO).

**Open decisions:**

- [ ] `actions` as a prop (current) vs `<Dialog.Actions>` as a required compound child — compound is more DX-forward; prop is simpler for the basic 2-button case.
- [ ] Full-screen breakpoint: auto-switch from basic → full-screen below `--breakpoint-compact`? Opt-in `responsive` prop recommended rather than automatic, to preserve predictability.
- [ ] Fully-controlled pattern without a trigger element: add `<DialogPortal>` / use `ModalOverlay` directly.
