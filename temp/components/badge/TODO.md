# Badge — TODO

## MUI Equivalent
`@mui/material/Badge` — direct functional equivalent. MUI has `color`, `overlap`, `anchorOrigin`, `badgeContent`, `invisible`, `showZero`, `variant` ("dot" | "standard"). kafUI beats it on: zero JS theming overhead, M3-faithful error-only color, cleaner API surface, positive `visible` semantics, dev-mode a11y warning.

---

## Beat-MUI Opportunities

| # | kafUI wins by… | vs MUI |
|---|----------------|--------|
| 1 | **Positive `visible` boolean** removes double-negative mental model | MUI `invisible={true}` to hide, `invisible={false}` to show — backwards |
| 2 | **CSS var color override** — one `--error` redeclaration, zero runtime cost | MUI `color` prop generates emotion class per value, bloats stylesheet |
| 3 | **Dev-mode `aria-label` warning** — surfaces missing a11y at build time | MUI emits no warning; silent a11y hole |
| 4 | **Logical CSS inset** — RTL just works, zero `anchorOrigin` prop needed | MUI `anchorOrigin={{ vertical, horizontal }}` prop; RTL requires manual flip |
| 5 | **Auto variant detection** from `count` presence — unambiguous defaults | MUI `variant="dot"` vs `variant="standard"` — extra prop, same info as `badgeContent` |

---

## Actionable TODO Checklist

### Core implementation
- [ ] `packages/react/src/badge/Badge.tsx` — anchor wrapper + badge element
- [ ] `packages/styles/src/badge/badge.css` inside `@layer kafui { … }`
- [ ] Auto-detect `variant`: `count === undefined` → `"small"`; `count !== undefined` → `"large"` (overridable)
- [ ] `max` overflow: render `{max}+` visually; supply exact count (or `"{max} or more"`) to `aria-label` composition helper
- [ ] `visible={false}` → add `.badge--hidden` → CSS plays exit scale transition → `transitionend` sets `aria-hidden="true"` on `.badge`; `visible={true}` removes `.badge--hidden` and `aria-hidden`

### Dev-mode a11y guard
- [ ] In development, warn to console if `aria-label` is absent AND the immediate child has neither `aria-label` nor `aria-labelledby` — badge count is then unannounced to AT

### Styles — inside `@layer kafui`
```css
@layer kafui {
  .badge__anchor {
    position: relative;
    display: inline-flex;
    overflow: visible; /* never clip the protruding dot */
  }

  .badge {
    --_bg:       var(--error);
    --_fg:       var(--on-error);
    --_r:        var(--corner-full);
    --_h:        16px;
    --_dot-size: 6px;
    --_px:       4px;

    position: absolute;
    inset-inline-end: 0;
    inset-block-start: 0;
    transform: translate(50%, -50%);
    background: var(--_bg);
    color: var(--_fg);
    border-radius: var(--_r);
    display: flex;
    align-items: center;
    justify-content: center;
    font: var(--label-small-font);
    pointer-events: none;
    transition:
      transform var(--duration-short2) var(--easing-emphasized-decelerate),
      opacity   var(--duration-short2) var(--easing-emphasized-decelerate);
  }

  .badge--small {
    width: var(--_dot-size);
    height: var(--_dot-size);
    padding: 0;
  }

  .badge--large {
    height: var(--_h);
    min-width: var(--_h);
    padding-inline: var(--_px);
  }

  .badge--hidden {
    transform: translate(50%, -50%) scale(0);
    opacity: 0;
    transition-timing-function: var(--easing-emphasized-accelerate);
  }

  @media (prefers-reduced-motion: reduce) {
    .badge { transition: none; }
    .badge--hidden { transform: translate(50%, -50%) scale(1); opacity: 0; }
  }
}
```
- [ ] Implement styles above; verify `overflow: visible` on anchor is not overridden by any reset
- [ ] `aria-hidden` on `.badge__label` (large only) to avoid double SR read

### Accessibility
- [ ] Anchor wrapper `aria-label` receives count-aware string from `aria-label` prop (consumer-supplied; no magic construction in default)
- [ ] Provide a helper `badgeAriaLabel(count, max, label)` utility that returns correct string — export it so consumers can use in their own `aria-label` prop
- [ ] `aria-hidden="true"` on `.badge` (the visual element); AT reads only the anchor label
- [ ] Guard: `visible={false}` must not leave `aria-label` with stale count after hide

### Tokens (map to unprefixed CSS custom props)
- [ ] `--error` → `light-dark(var(--ref-error-40), var(--ref-error-80))` (from `_TOKENS.md` pattern)
- [ ] `--on-error` → `light-dark(#fff, var(--ref-error-20))`
- [ ] `--corner-full` → `9999px`
- [ ] `--label-small-font` — typography shorthand
- [ ] `--duration-short2`, `--easing-emphasized-decelerate`, `--easing-emphasized-accelerate`

### Tests
- [ ] Unit: `count=1000`, `max=999` → renders "999+" visually
- [ ] Unit: `visible={false}` → `.badge--hidden` applied; `aria-hidden="true"` set after transition
- [ ] Unit: `count` undefined → variant resolves to `"small"`, no `.badge__label` rendered
- [ ] A11y: anchor `aria-label` present; `.badge__label` is `aria-hidden`
- [ ] Dev-mode: missing `aria-label` triggers console.warn in test

### Storybook
- [ ] Small dot (default, no count)
- [ ] Large: count 1, 12, 99, 1000 (overflow ceiling)
- [ ] `visible` toggle (show → hide → show animation)
- [ ] Inside `<IconButton>` (nav item context)
- [ ] RTL layout (dir="rtl" wrapper — confirm top-end corner)
- [ ] Dark mode
