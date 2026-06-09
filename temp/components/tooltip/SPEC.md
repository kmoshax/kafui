# Tooltip

**Purpose:** Provides short, contextual text labels or richer descriptions for UI elements on hover/focus. Plain tooltips label icon buttons; rich tooltips surface more detail — title, body, and optional action — for complex controls.
**M3 category:** Communication → Tooltip.

---

## Anatomy / Parts → BEM Elements

### Plain Tooltip

```
.tooltip                            tooltip surface (positioned overlay)
.tooltip__label                     single-line text label
```

### Rich Tooltip

```
.tooltip--rich                      modifier on .tooltip for rich variant
.tooltip__title                     optional heading line
.tooltip__body                      multi-line description text
.tooltip__actions                   optional footer row of action buttons
.tooltip__action                    individual text/tonal button in actions row
```

Trigger wrapper is not styled as part of the tooltip BEM — it is the consumer's host element (e.g., `.icon-button`).

---

## Variants

| Variant | `variant` prop | Description |
|---------|---------------|-------------|
| Plain | `"plain"` (default) | Single label, max 1 line, shown on hover/focus. Not interactive. |
| Rich | `"rich"` | Multi-section: title + body + optional actions. Interactive (can receive focus). |

---

## States

| State | Notes |
|-------|-------|
| Hidden | Default; `visibility: hidden` + `pointer-events: none` (NOT `display: none` — keeps `aria-describedby` resolvable) |
| Visible | After hover delay (300 ms default plain / 500 ms rich) or immediate on keyboard focus |
| Focused (rich) | Rich tooltip can receive keyboard focus when it contains action buttons |
| Exiting | Short fade-out animation |

**Open delay:** Plain tooltip — 300 ms hover delay, 0 ms on focus. Rich tooltip — 500 ms hover delay; instant on focus.
**Close delay:** 150 ms after pointer leaves both trigger and tooltip (to allow pointer to move into rich tooltip).
**No state layers** on the tooltip surface itself. Action buttons inside rich tooltips carry standard text-button state layers.

---

## Design Tokens

> All CSS custom properties must be declared inside `@layer kafui { … }`.
> Component-internal vars are declared at the top of the component's rule block (short, unprefixed).

### Plain Tooltip

| Role | CSS custom property | Usage |
|------|---------------------|-------|
| `inverse-surface` | `--inverse-surface` | Surface background |
| `inverse-on-surface` | `--inverse-on-surface` | Label text |
| `corner-extra-small` | `--corner-extra-small` | Border-radius (4 dp) |
| `body-small` typescale | `--body-small-size`, `--body-small-line-height`, `--body-small-weight` | Label typography |
| elevation level 2 | `--elevation-2` | Box shadow |
| `duration-short2` | `--duration-short2` | Enter/exit transition |

### Rich Tooltip

| Role | CSS custom property | Usage |
|------|---------------------|-------|
| `surface` | `--surface` | Surface background |
| `on-surface-variant` | `--on-surface-variant` | Body text |
| `on-surface` | `--on-surface` | Title text |
| `primary` | `--primary` | Action button label |
| `corner-medium` | `--corner-medium` | Border-radius (12 dp) |
| `title-small` typescale | `--title-small-size`, etc. | Title typography |
| `body-medium` typescale | `--body-medium-size`, etc. | Body typography |
| `label-large` typescale | `--label-large-size`, etc. | Action button typography |
| elevation level 2 | `--elevation-2` | Box shadow |
| `outline-variant` | `--outline-variant` | Border (rich tooltip has a 1 px border in M3) |

Max width: Plain **200 dp**, Rich **320 dp**.

### Example CSS (illustrative, not exhaustive)

```css
@layer kafui {
  .tooltip {
    /* component-internal vars */
    --max-w: 200px;
    --pad-block: 4px;
    --pad-inline: 8px;

    max-width: var(--max-w);
    padding: var(--pad-block) var(--pad-inline);
    background: var(--inverse-surface);
    color: var(--inverse-on-surface);
    border-radius: var(--corner-extra-small);
    box-shadow: var(--elevation-2);
    font-size: var(--body-small-size);
    line-height: var(--body-small-line-height);
    visibility: hidden;
    opacity: 0;
    transition: opacity var(--duration-short2) var(--easing-standard),
                visibility 0s var(--duration-short2);
  }

  .tooltip[data-entering],
  .tooltip[data-open] {
    visibility: visible;
    opacity: 1;
    transition: opacity var(--duration-short2) var(--easing-standard);
  }

  .tooltip--rich {
    --max-w: 320px;
    --pad-block: 12px;
    --pad-inline: 16px;

    background: var(--surface);
    color: var(--on-surface-variant);
    border-radius: var(--corner-medium);
    border: 1px solid var(--outline-variant);
  }

  .tooltip--rich .tooltip__title {
    color: var(--on-surface);
    font-size: var(--title-small-size);
    margin-block-end: 4px;
  }

  .tooltip--rich .tooltip__actions {
    display: flex;
    gap: 8px;
    justify-content: flex-end;
    margin-block-start: 8px;
  }

  @media (prefers-reduced-motion: reduce) {
    .tooltip { transition: none; }
  }
}
```

---

## Interaction & Accessibility

### Plain Tooltip ARIA

```html
<!-- Trigger -->
<button aria-describedby="tt-1">
  <svg aria-hidden="true">…</svg>
</button>

<!-- Tooltip (in portal, always in DOM) -->
<div id="tt-1" role="tooltip">
  Edit
</div>
```

- `role="tooltip"` on the tooltip element.
- `aria-describedby` links trigger to tooltip — supplements the accessible name; does NOT replace `aria-label` on a bare icon button.
- Tooltip **must remain in the DOM** at all times (even when hidden) so `aria-describedby` resolves for screen readers. Use `visibility: hidden` / `opacity: 0`, never `display: none` while associated.
- React Aria's `TooltipTrigger` manages this automatically via the `Tooltip` component's portal.

### Rich Tooltip ARIA

Rich tooltips containing interactive elements are closer to a non-modal popover than a tooltip:

```html
<!-- Non-interactive rich (no actions): role="tooltip" is acceptable -->
<div id="rt-1" role="tooltip" aria-label="Performance score">
  <p class="tooltip__title">Performance score</p>
  <p class="tooltip__body">…</p>
</div>

<!-- Interactive rich (with actions): role="dialog" required -->
<div id="rt-1" role="dialog" aria-label="Performance score" aria-modal="false">
  <p class="tooltip__title">…</p>
  <p class="tooltip__body">…</p>
  <div class="tooltip__actions">
    <button>Learn more</button>
  </div>
</div>
```

- `role="tooltip"` is non-interactive per ARIA spec. As soon as action buttons appear, switch to `role="dialog"` with `aria-modal="false"` (it is NOT a full modal; background remains interactive).
- Trigger uses `aria-describedby` (non-interactive variant) or `aria-controls` + `aria-expanded` (interactive/dialog variant).
- `aria-modal="false"` is explicit and important — avoids AT hiding background content.

### Keyboard — Plain Tooltip

- Tooltip shows on `focus`; hides on `blur`.
- No interactive elements — focus never moves into the tooltip.

### Keyboard — Rich Tooltip

- `Tab` on trigger (interactive variant): show tooltip + move focus to first focusable element inside.
- `Escape`: close + return focus to trigger.
- `Tab` past last action in tooltip: close + move focus to next tabbable element in document (NOT a full focus trap — this is a non-modal overlay).

### Screen reader

- Plain tooltip is announced as part of the trigger's accessible description via `aria-describedby`.
- Rich tooltip (dialog): AT reads the full dialog content when focus enters.

### Reduced motion

```css
@media (prefers-reduced-motion: reduce) {
  .tooltip { transition: none; }
}
```

Instant show/hide.

### RTL

Tooltip overlay placement uses logical positioning. React Aria's built-in `Popover` placement engine respects `dir` automatically. No internal directionality issues with text — inherits document direction.

---

## Proposed kafUI React API

```tsx
// Plain tooltip — wraps RAC TooltipTrigger + Tooltip
// Rich non-interactive — same, with extra content props
// Rich interactive (actions present) — wraps RAC DialogTrigger + Popover

interface TooltipProps {
  variant?: "plain" | "rich";         // default: "plain"
  /** Label text (plain) or title (rich) */
  label: string;
  /** Rich tooltip body text */
  body?: string;
  /** Rich tooltip action buttons; presence switches backing primitive to DialogTrigger + Popover */
  actions?: Array<{ label: string; onPress: () => void }>;
  /** Hover open delay in ms. Defaults: 300 (plain) / 500 (rich). */
  delay?: number;
  /** Placement relative to trigger. Default: "top". Logical values preferred. */
  placement?: "top" | "bottom" | "start" | "end" | "top-start" | "top-end" | "bottom-start" | "bottom-end";
  children: React.ReactElement;       // the trigger element
}

// Plain usage
<Tooltip label="Edit">
  <IconButton name="edit" aria-label="Edit" />
</Tooltip>

// Rich non-interactive
<Tooltip variant="rich" label="Performance score"
  body="Measures how fast this page loads on a standard connection."
>
  <button>Score: 92</button>
</Tooltip>

// Rich interactive (backed by DialogTrigger + Popover; transparent to consumer)
<Tooltip
  variant="rich"
  label="Performance score"
  body="Measures how fast this page loads on a standard connection."
  actions={[{ label: "Learn more", onPress: openDocs }]}
>
  <button>Score: 92</button>
</Tooltip>
```

**React Aria primitives:**

| Case | RAC primitive |
|------|--------------|
| Plain tooltip | `TooltipTrigger` + `Tooltip` |
| Rich, no actions | `TooltipTrigger` + `Tooltip` (extra content rendered inside) |
| Rich, with actions | `DialogTrigger` + `Popover` + `Dialog` — `aria-modal="false"` |

React Aria's `Tooltip` intentionally does not support interactive content. The `DialogTrigger + Popover` path is RAC's own recommended approach and is **transparent to the consumer** via the unified `<Tooltip>` API.

**BEM classes emitted:**

```
.tooltip                    always
.tooltip--plain             variant="plain" (default)
.tooltip--rich              variant="rich"
.tooltip__label             plain label
.tooltip__title             rich title
.tooltip__body              rich body
.tooltip__actions           rich actions row
.tooltip__action            individual action button
```

**Deviations from M3 reference:**

- Interactive rich tooltip backed by `DialogTrigger + Popover` with `aria-modal="false"` — correct per ARIA; M3 visual spec does not distinguish the backing primitive.
- `placement` uses logical values (`start`/`end`) instead of `left`/`right`; React Aria's popover engine maps these to physical directions based on `dir`.
- No `role="tooltip"` on interactive rich variants — correct per ARIA spec (M3 spec predates ARIA's clarification on interactivity).

**Open decisions:**

- [ ] Does `variant="rich"` without `actions` use `TooltipTrigger` (simpler path) or `DialogTrigger` (consistent code path for all rich variants)? Recommendation: split on `actions` presence only; no-actions rich stays on `TooltipTrigger`.
- [ ] Positioning: React Aria's built-in Popover overlay vs `@floating-ui/react`. Prefer RAC built-in to keep zero extra dependencies; fall back to `@floating-ui` only if collision/flip behavior is insufficient.
