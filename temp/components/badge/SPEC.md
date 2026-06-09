# Badge

**Purpose:** A small status descriptor anchored to an icon, avatar, or navigation item. Communicates count, presence, or alert state without requiring user interaction.
**M3 category:** Communication → Badge.

---

## Anatomy / Parts → BEM Elements

```
.badge                      root element (the badge pill/dot)
.badge__label               numeric/text label (large badge only; aria-hidden)
.badge__anchor              wrapping positioner — position:relative inline-flex
.badge__anchor > [slot]     the badged child (icon, avatar, nav item)
```

The badge is absolutely positioned inside `.badge__anchor` which is `position: relative; display: inline-flex`.

> **Anatomy note:** M3 defines the badge as sitting at the *top-end* corner of its anchor (block-start / inline-end). The anchor wrapper must not clip overflow — it emits `overflow: visible` so the dot that protrudes beyond the icon edge is never cut off. Avoid wrapping in a parent with `overflow: hidden`.

---

## Variants

| Variant | `variant` prop  | M3 size spec | Description |
|---------|----------------|--------------|-------------|
| Small   | `"small"`      | 6 × 6 dp dot | No label; conveys presence or unread state; no minimum width |
| Large   | `"large"`      | min-width 16 dp, height 16 dp, h-padding 4 dp | Displays numeric count 1–999+; pill shape |

`variant` auto-resolves: if `count` is supplied, default is `"large"`; otherwise `"small"`. Explicitly setting `variant="small"` with a `count` suppresses the count visually (dot only) — this is intentional for "has activity" vs "exact count" situations.

---

## States

| State   | Behavior |
|---------|----------|
| Default | Full opacity, no state layer (badge is not interactive) |
| Hidden  | `visible={false}` → scale-out exit animation (`transform: scale(0)`), `aria-hidden="true"` on badge element |

Badges carry **no hover / pressed / focus state**. State layers belong to the host element (icon button, nav item). The badge only ever participates in enter/exit transitions.

---

## Design Tokens

| Token | CSS custom property (unprefixed) | Usage |
|-------|----------------------------------|-------|
| `md.sys.color.error` | `--error` | badge background |
| `md.sys.color.on-error` | `--on-error` | label text color |
| `md.sys.shape.corner.full` | `--corner-full` | border-radius (fully rounded pill/dot) |
| `md.sys.typescale.label-small` | `--label-small-font` / `--label-small-size` / `--label-small-weight` / `--label-small-line-height` | label typography (large badge) |
| `md.sys.motion.duration.short2` | `--duration-short2` | enter/exit transition duration |
| `md.sys.motion.easing.emphasized-decelerate` | `--easing-emphasized-decelerate` | enter easing |
| `md.sys.motion.easing.emphasized-accelerate` | `--easing-emphasized-accelerate` | exit easing |

**Component-internal variables** (scoped inside `.badge { … }`):
```css
.badge {
  --_bg:        var(--error);
  --_fg:        var(--on-error);
  --_r:         var(--corner-full);
  --_h:         16px;          /* large height */
  --_dot-size:  6px;           /* small diameter */
  --_px:        4px;           /* large horizontal padding */
}
```

Consumers override color by redeclaring `--error` / `--on-error` on an ancestor scope, or use the internal `--_bg` / `--_fg` overrides directly on `.badge` for one-off recoloring.

> **M3 color note:** M3 spec uses *only* the error role for badges. No "primary" or "secondary" badge color is defined. Arbitrary color overrides are achievable via CSS var scope redeclaration — no `color` prop needed.

---

## Interaction & Accessibility

### Keyboard
Non-interactive. Tab order targets the host element, not the badge.

### Focus
No focus ring on the badge itself. The containing icon button / nav item receives focus normally.

### ARIA
- **Small badge (dot):** `aria-label` on the *anchor wrapper* describes state, e.g. `aria-label="Notifications — new activity"`. The `.badge` element is `aria-hidden="true"`.
- **Large badge:** anchor `aria-label` MUST incorporate the count, e.g. `aria-label="Notifications, 12 unread"`. The `.badge__label` is `aria-hidden="true"` to prevent double-read.
- When `count > max`, render `{max}+` visually; `aria-label` uses the exact count if known (`"120 unread"`), otherwise `"{max} or more unread"`.
- **Dot with count (suppressed display):** `aria-label` still references the count for AT even though the dot hides it.

### Live-region strategy
The badge itself emits **no `aria-live` region**. Dynamic count updates (incoming messages) are the responsibility of the host component. If the host cannot manage it, consumers wrap the anchor in an `aria-live="polite"` region and update `aria-label` when count changes. Do not place `aria-live` on `.badge__anchor` by default — it will fire on every re-render, not just meaningful count changes.

### Reduced motion
```css
@media (prefers-reduced-motion: reduce) {
  .badge { transition: none; transform: none !important; }
}
```
`visible={false}` → instant `display: none` or `opacity: 0`. No scale animation.

### RTL
Badge sits at `inset-inline-end: 0; inset-block-start: 0; transform: translate(50%, -50%)`. Using CSS logical insets means the badge automatically moves to the correct corner in RTL layouts (top-left → top-right swap is automatic). No `dir`-specific overrides needed.

---

## Proposed kafUI React API

```tsx
interface BadgeProps {
  /**
   * "small" = dot, "large" = numbered pill.
   * Default: auto-detected — "large" if count is set, "small" otherwise.
   */
  variant?: "small" | "large";
  /** Numeric count for the large badge. Omit → small dot. */
  count?: number;
  /** Count ceiling; displays "{max}+" when count > max. Default: 999 */
  max?: number;
  /**
   * When false, plays exit animation then sets aria-hidden.
   * Default: true
   */
  visible?: boolean;
  /**
   * Accessible label for the anchor wrapper.
   * Required when the host element has no other label that includes badge state.
   * kafUI will warn in dev if this is absent and the child has no aria-label.
   */
  "aria-label"?: string;
  /** The icon / avatar / nav-item to anchor the badge over. */
  children: React.ReactNode;
  className?: string;
}

// Numbered badge
<Badge count={12} aria-label="Notifications, 12 unread">
  <Icon name="notifications" />
</Badge>

// Overflow ceiling
<Badge count={1204} max={999} aria-label="Notifications, 999 or more unread">
  <Icon name="notifications" />
</Badge>

// Dot (new activity, no count)
<Badge aria-label="Mail — new messages">
  <Icon name="mail" />
</Badge>

// Controlled hide/show
<Badge count={count} visible={count > 0} aria-label={`Cart, ${count} items`}>
  <Icon name="shopping_cart" />
</Badge>
```

**React Aria primitive:** None — Badge is a pure presentational overlay. The anchor emits a plain `<span>` wrapping the children. If the *child itself* is not yet focusable (e.g. a raw `<img>` or `<svg>`), consumers must add their own focusable wrapper; kafUI does not silently inject a `<button>`.

**BEM classes emitted:**
```
.badge__anchor
.badge  .badge--small | .badge--large
.badge__label        (large only, aria-hidden)
.badge--hidden       (exit animation pending)
```

**Layer:**
```css
@layer kafui { /* all badge styles */ }
```

**Why this beats MUI `<Badge>`:**
- No `color`, `overlap`, `anchorOrigin` props. M3 has one position rule; color override is a CSS var swap — zero API surface.
- `visible` (positive semantics) vs MUI `invisible` (negative) — eliminates the `!invisible` double-negative.
- Dev-mode `aria-label` warning surfaces a11y gaps that MUI silently ignores.
- Zero emotion / sx overhead — pure BEM + CSS custom props.
