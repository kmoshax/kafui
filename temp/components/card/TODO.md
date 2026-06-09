# Card — TODO

## MUI equivalent

`@mui/material/Card` + `CardMedia` / `CardHeader` / `CardContent` / `CardActions` / `CardActionArea`.

MUI Card is a thin wrapper over `Paper`; variants are loosely mapped (`variant="outlined"` exists, elevated is the default, no `filled`). Shape defaults to 4 dp (not M3's 12 dp). Interactions rely on a JS-driven ripple. `CardHeader` renders `<span>` for the title, not a semantic heading. `CardActionArea` is a separate wrapping element that consumers must manually nest.

**kafUI beats MUI here on every axis.** The items below are framed as concrete winning opportunities.

---

## Beat-MUI opportunities

### 1. Ship all three M3 variants with exact surface tokens — MUI only does two
MUI `Card` has `variant="outlined"` and the default (elevated), but no `filled`. The elevated variant uses `elevation` (an integer) instead of M3's named levels and picks the wrong surface color — MUI uses `background.paper`, not `surface-container-low`. kafUI maps each variant exactly: `--surface-container-low` / `--surface-container-highest` / `--surface` with `--outline-variant` border.

**Tasks:**
- [ ] Create `card.css` inside `@layer kafui { … }`
- [ ] `.card--elevated`: `background: var(--surface-container-low); box-shadow: var(--elevation-1)`
- [ ] `.card--filled`: `background: var(--surface-container-highest); box-shadow: var(--elevation-0)`
- [ ] `.card--outlined`: `background: var(--surface); border: 1px solid var(--outline-variant); box-shadow: var(--elevation-0)`
- [ ] Shape: `border-*-radius: var(--corner-medium)` via logical properties (all four corners)
- [ ] Verify colors match M3 spec in both light and dark (automatic via `light-dark()`)

### 2. Zero-JS state layer — MUI uses a runtime ripple component
MUI injects a JS `Ripple` component for every interactive surface. kafUI delivers the same visual feedback with a pure CSS `.state-layer` pseudo-element driven by RAC `data-*` attributes — zero extra JS, zero extra DOM renders.

**Tasks:**
- [ ] `.state-layer` utility: `position: absolute; inset: 0; border-radius: inherit; pointer-events: none; background: currentColor; opacity: 0; transition: opacity var(--duration-short2)`
- [ ] `.card .state-layer { color: var(--on-surface) }`
- [ ] `.card[data-hovered] .state-layer { opacity: var(--state-hover) }`
- [ ] `.card[data-focus-visible] .state-layer { opacity: var(--state-focus) }`
- [ ] `.card[data-pressed] .state-layer { opacity: var(--state-pressed) }`
- [ ] `.card--dragging .state-layer { opacity: var(--state-dragged) }`
- [ ] Elevation step on hover: `.card[data-hovered].card--elevated { box-shadow: var(--elevation-2) }`
- [ ] Dragged elevation: `.card--dragging { box-shadow: var(--elevation-4) }`
- [ ] Elevation transition: `transition: box-shadow var(--duration-short4) var(--easing-standard)`
- [ ] `@media (prefers-reduced-motion: reduce)`: remove all transitions

### 3. Compound anatomy with semantic heading — MUI's `CardHeader` renders `<span>` for the title
MUI's `CardHeader` emits a `<span>` for the title regardless of context, breaking document outline and screen reader heading navigation. kafUI's `Card.Header` renders a real heading element at the configurable level.

**Tasks:**
- [ ] `Card.Header`: renders headline as `<h{headingLevel}>` (default `h3`); `headingLevel` prop `2 | 3 | 4 | 5 | 6`
- [ ] `Card.Header` grid layout: `grid-template-columns: auto 1fr auto`; logical gap and padding
- [ ] `Card.Media`: requires `alt` prop at TypeScript level (`alt: string` not optional)
- [ ] `Card.Content`: simple wrapper; `padding-inline: var(--pad-inline); padding-block: var(--pad-block)`
- [ ] `Card.Actions`: `display: flex; flex-wrap: wrap; gap: 8px; padding-inline/block`
- [ ] Export compound: `Card.Media`, `Card.Header`, `Card.Content`, `Card.Actions`
- [ ] `__media` img/video: `width: 100%; aspect-ratio: 16/9; object-fit: cover; display: block`
- [ ] `__media` clips to card's top corners via `border-start-*-radius: var(--radius)`

### 4. Correct overlay pattern for nested interactives — MUI's `CardActionArea` creates invalid nesting
MUI wraps `CardActionArea` around clickable content, which means `<button>` inside `<button>` is possible. kafUI uses a full-surface overlay `<Button>`/`<Link>` behind the content so action buttons remain independently focusable with no invalid HTML nesting.

**Tasks:**
- [ ] Clickable card: root `<div>` + full-surface overlay RAC `Button`/`Link` at `position: absolute; inset: 0; z-index: 0`
- [ ] Card content at `position: relative; z-index: 1` so it's above the overlay
- [ ] `Card.Actions` children automatically at `z-index: 1` — no `stopPropagation` needed
- [ ] Document the pattern in JSDoc so consumers don't accidentally nest interactives
- [ ] Selectable card: root renders as RAC `Checkbox`/`ListBoxItem`; `aria-checked` handled by RAC
- [ ] Draggable card: `isDragging` prop applies `.card--dragging`; integrate RAC `useDraggableItem`

### 5. Dev-time accessibility warning — neither MUI nor shadcn do this
Emit a dev-mode `console.warn` when a card is interactive and has neither a visible heading nor `aria-label`, since screen readers would announce a meaningless button/link.

**Tasks:**
- [ ] `aria-label` warning: if `onPress`/`href` present, no `Card.Header` with headline, and no `aria-label` prop — warn in dev
- [ ] TypeScript: make `alt` on `Card.Media` required (no `?`)
- [ ] TypeScript: make `headline` on `Card.Header` required

### 6. Disabled state done right — MUI disables only `CardActionArea`, not the full surface
MUI has no first-class disabled surface; consumers must manually style text. kafUI applies the full M3 disabled treatment via CSS alone.

**Tasks:**
- [ ] `.card[data-disabled]` or `.card--disabled`: surface overlay `on-surface` at 12%; text `on-surface` at 38%
- [ ] `isDisabled` prop passed through to RAC overlay; RAC sets `data-disabled`
- [ ] Confirm disabled card suppresses all state-layer hover/focus/pressed effects

### 7. Dark mode zero-config — MUI requires a separate dark theme object
Because all tokens use `light-dark()`, dark mode is free. Validate it; document it; use it as a selling point.

**Tasks:**
- [ ] Write a dark-mode visual test: all three variants in dark scheme
- [ ] Confirm `--surface-container-low/highest/surface` + `--outline-variant` all flip correctly under `color-scheme: dark`

---

## Styles (`@kafui/styles`) summary checklist
- [ ] `card.css` — all rules wrapped in `@layer kafui`
- [ ] Logical properties throughout (`padding-inline`, `padding-block`, `margin-inline-start`, `border-start-start-radius`, …)
- [ ] All token names unprefixed: `--surface-container-low`, `--corner-medium`, `--elevation-1`, `--state-hover`, `--easing-standard`, `--duration-short4`, etc.
- [ ] Component-internal vars declared at `.card {}` block: `--radius`, `--pad-block`, `--pad-inline`
- [ ] No `kafui-` prefix on any BEM class
- [ ] `@layer kafui` provides collision safety instead of class prefix

## React (`@kafui/react`) summary checklist
- [ ] Infer interactivity from presence of `onPress`/`href`/`isSelected`/`isDraggable` — no redundant `isInteractive` flag
- [ ] Overlay pattern for clickable cards (no nested `<button>` inside `<button>`)
- [ ] Compound: `Card.Media`, `Card.Header`, `Card.Content`, `Card.Actions` all exported
- [ ] `headingLevel` on `Card.Header` with default `3`
- [ ] Dev warning for inaccessible interactive cards
- [ ] Full TypeScript strict types for all sub-component props

## Testing
- [ ] Unit: all three variant class names applied correctly
- [ ] Unit: clickable card fires `onPress` on click and Enter/Space
- [ ] Unit: action buttons inside clickable card do NOT trigger card press (overlay pattern)
- [ ] Unit: inferred interactivity — no `isInteractive` prop needed
- [ ] A11y: heading level is correct in document outline
- [ ] A11y: `aria-checked` updates on selectable card
- [ ] A11y: dev warning fires when interactive card is missing `aria-label` and heading
- [ ] Visual: hover elevation step (box-shadow change) — all three variants
- [ ] Visual: disabled state — surface at 12% + text at 38%
- [ ] Visual: dark mode — all three variants
