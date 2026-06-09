# Snackbar

**Purpose:** Brief, bottom-anchored messages that inform users of non-critical background processes (file saved, message sent, network error). Ephemeral — appear temporarily and may include a single action or dismiss button.
**M3 category:** Communication → Snackbar.

---

## Anatomy / Parts → BEM Elements

```
.snackbar-region                    fixed viewport anchor (houses the live region + queue)
.snackbar                           single snackbar surface (one visible at a time per M3)
.snackbar__text                     supporting text (the message)
.snackbar__actions                  optional action + close row
.snackbar__action                   text/label button for primary action
.snackbar__close                    icon button — dismiss (×)
```

> **Naming note:** The region (fixed viewport position) and the individual message surface (`.snackbar`) are separate DOM concerns. `.snackbar-region` is the `role="status"` live-region wrapper; `.snackbar` is the visual chip/pill that appears inside it.

---

## Variants / Configurations

M3 defines **two layout variants** and **three action configurations** — they combine, not multiply:

### Layout variants (`variant` on `SnackbarOptions`)

| Variant | `variant` value | Description |
|---------|----------------|-------------|
| Single-line | `"single-line"` | Message + optional action on one row |
| Two-line | `"two-line"` | Message wraps; action may stack below text |

### Action configurations (presence flags on `SnackbarOptions`)

| Config | Props | Notes |
|--------|-------|-------|
| Message only | neither `actionLabel` nor `showClose` | pure informational |
| With action | `actionLabel` + `onAction` | single text action (Undo, Retry, etc.) |
| With close | `showClose: true` | icon ×; required when `duration=0` (persistent) |
| Action + close | both above | two-line layout recommended |

---

## States

| State | Behavior |
|-------|----------|
| Enter | Slide up (`translateY(100%)→0`) + `opacity 0→1` from bottom edge |
| Visible | Static; auto-dismiss after `duration` ms (default 4000 ms) |
| Hover | Pause auto-dismiss timer (mouse users) |
| Focus-within | Pause auto-dismiss timer (keyboard users trapped in action/close) |
| Exiting | Slide down (`translateY(0)→100%`) + `opacity 1→0` |
| Dismissed | Removed from DOM; next queued item surfaces |

State layers on `.snackbar__action` button: use `--inverse-primary` tint at hover 8%, focus 10%, pressed 10% via `.state-layer` pseudo-element.

---

## Design Tokens

| Token | CSS custom property (unprefixed) | Usage |
|-------|----------------------------------|-------|
| `md.sys.color.inverse-surface` | `--inverse-surface` | snackbar surface background |
| `md.sys.color.inverse-on-surface` | `--inverse-on-surface` | supporting text color |
| `md.sys.color.inverse-primary` | `--inverse-primary` | action button label color |
| `md.sys.shape.corner.extra-small` | `--corner-extra-small` | surface border-radius (4 dp) |
| `md.sys.typescale.body-medium` | `--body-medium-font` etc. | supporting text typography |
| `md.sys.typescale.label-large` | `--label-large-font` etc. | action button label typography |
| `md.sys.elevation.level3` | `--elevation-3` | box-shadow |
| `md.sys.motion.duration.short4` | `--duration-short4` | enter/exit animation |
| `md.sys.motion.easing.emphasized-decelerate` | `--easing-emphasized-decelerate` | enter easing |
| `md.sys.motion.easing.emphasized-accelerate` | `--easing-emphasized-accelerate` | exit easing |

**Min width:** 288 dp. **Max width:** 568 dp (full-width `100%` on screens < 600 dp).

**Component-internal variables** (scoped inside `.snackbar { … }`):
```css
@layer kafui.components {
.snackbar {
  --_bg:        var(--inverse-surface);
  --_fg:        var(--inverse-on-surface);
  --_action-fg: var(--inverse-primary);
  --_r:         var(--corner-extra-small);
  --_elev:      var(--elevation-3);
  --_min-w:     288px;
  --_max-w:     568px;
}
}
```

---

## Interaction & Accessibility

### Keyboard
- Snackbar does NOT receive automatic focus on appear (non-blocking UX).
- Action and close buttons are reachable via `Tab` from current focus position.
- `Escape` dismisses the focused snackbar (handled by React Aria `Toast`).
- Focus returns to the previously focused element after dismissal (React Aria handles this).

### ARIA / Live Regions

```html
<!-- Region: polite for informational; one instance per app at root -->
<div
  class="snackbar-region"
  role="status"
  aria-live="polite"
  aria-relevant="additions"
  aria-atomic="true"
  aria-label="Notifications"
>
  <!-- .snackbar injected / removed here -->
</div>
```

- `role="status"` + `aria-live="polite"` for informational toasts.
- `role="alert"` + `aria-live="assertive"` for `priority="assertive"` messages — React Aria `Toast` supports this via `alertDialog` role swapping; kafUI wires `priority` to this.
- `aria-atomic="true"` on the region ensures the full message string is read, not just the diff.
- Close button: `aria-label="Dismiss"` (i18n key `snackbar.dismiss`).
- Action button: label from `actionLabel` prop — must be self-explanatory without surrounding context ("Undo", not "OK").

### Priority / politeness

`priority="assertive"` changes the individual toast to `role="alertdialog"` (or React Aria's assertive channel) and shifts focus to the snackbar. Use only for critical, blocking errors. Most snackbars should be `"polite"`.

> **React Aria `ToastQueue` priority:** RAC's `ToastQueue` accepts a `priority` number, not a string. kafUI maps: `"polite"` → `0`, `"assertive"` → `1` internally. The toast item then checks priority to pick its ARIA role.

### Queue management (M3 spec)
- Only **one snackbar visible at a time**. Subsequent `add()` calls enqueue; next item shows only after the current exit animation completes.
- Queue is owned by the `ToastQueue` instance at `SnackbarRegion` level, not per-item.
- `duration=0` → persistent; requires `showClose=true` (kafUI enforces in dev).

### Screen reader
AT reads the message when injected into the live region. Keep messages ≤ 60 characters and ≤ 2 lines for AT-friendly announcements.

### Reduced motion
```css
@media (prefers-reduced-motion: reduce) {
  .snackbar { animation: none; transition: none; }
  .snackbar-region .snackbar { opacity: 1; transform: none; }
}
```
Snackbar appears and disappears instantly; timer behavior unchanged.

### RTL
`.snackbar-region` anchors with `inset-inline: 0` (centered, full width on mobile). Actions use `flex-direction: row`; logical flex order handles RTL naturally.

---

## Queue API — Imperative Surface (the beat-MUI core)

The queue API is the central DX bet. It must be ergonomic, composable, and type-safe.

### Queue instantiation
```ts
// packages/react/src/snackbar/snackbarQueue.ts
import { ToastQueue } from "@react-stately/toast";
import type { SnackbarOptions } from "./types";

// A singleton queue — one per app, shared via context.
// Consumers who need multiple regions (e.g. app + dialog overlay) can create additional instances.
export const snackbarQueue = new ToastQueue<SnackbarOptions>({
  maxVisibleToasts: 1,
});
```

### Hook
```ts
// useSnackbar — wraps queue.add() with M3-idiomatic defaults
export function useSnackbar() {
  return {
    add(options: SnackbarOptions): string {         // returns toast key for imperative close
      return snackbarQueue.add(options, {
        timeout: options.duration ?? 4000,
        priority: options.priority === "assertive" ? 1 : 0,
      });
    },
    close(key: string): void {
      snackbarQueue.close(key);
    },
    closeAll(): void {
      snackbarQueue.closeAll?.();
    },
  };
}
```

### `SnackbarOptions` type
```ts
export interface SnackbarOptions {
  /** The message text. Required. */
  message: string;
  /** Layout variant. Default: auto-detected as "two-line" if message > 40 chars, else "single-line" */
  variant?: "single-line" | "two-line";
  /** Text action label (e.g. "Undo"). Only one action supported per M3 spec. */
  actionLabel?: string;
  /** Callback when action button is pressed. Dismisses snackbar automatically. */
  onAction?: () => void;
  /** Show × dismiss button. Automatically true when duration=0. Default: false */
  showClose?: boolean;
  /** Auto-dismiss delay ms. 0 = persistent (requires showClose). Default: 4000 */
  duration?: number;
  /** ARIA politeness. Default: "polite". "assertive" shifts focus + uses role="alert". */
  priority?: "polite" | "assertive";
}
```

### Region component
```tsx
// Placed ONCE at app root (typically in RootLayout or App.tsx)
<SnackbarRegion queue={snackbarQueue} aria-label="App notifications" />
```

- `queue` prop is explicit — no magic global singleton injection. This makes SSR-safe and testable.
- `aria-label` defaults to `"Notifications"` but should be app-specific.

### Provider pattern (optional convenience)
```tsx
// Optional: <SnackbarProvider> wraps SnackbarRegion + provides queue via context
// so useSnackbar() works without prop-drilling the queue instance.
<SnackbarProvider>
  <App />
</SnackbarProvider>

// Anywhere in tree:
const { add } = useSnackbar();
add({ message: "File saved", actionLabel: "Undo", onAction: handleUndo });
```

**Deviations from MUI `<Snackbar>` / `notistack`:**
- No `severity` prop — M3 snackbars are monochromatic (inverse-surface). Error priority is an ARIA concern (`priority="assertive"`), not a color concern.
- No `anchorOrigin` — M3 specifies bottom-center only; RTL is handled by logical CSS.
- No `open` boolean per instance — queue API eliminates manual state management.
- `add()` returns a key for targeted `close(key)` — better than notistack's `closeSnackbar` with implicit ID.
- Auto-detect `"two-line"` variant from message length — zero config for common case.
