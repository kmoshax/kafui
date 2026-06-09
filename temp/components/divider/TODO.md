# Divider — TODO

## MUI Equivalent

`Divider` — supports `orientation`, `variant` (fullWidth/inset/middle), `flexItem`, `light`, and optional text children via `textAlign`.

---

## kafUI beats MUI — opportunities

| Dimension | kafUI wins | MUI pain |
|---|---|---|
| **Minimal API — 3 props** | `variant`, `orientation`, `decorative` — nothing more; CSS vars handle everything else | MUI exposes `flexItem`, `light`, `textAlign`, `children` — props for concerns that should be CSS |
| **Correct vertical element** | RAC `Separator` renders `<div role="separator" aria-orientation="vertical">` for vertical — valid HTML + correct ARIA | MUI renders `<hr>` for vertical dividers — `<hr>` is block-level and semantically wrong vertically; invalid HTML5 |
| **Explicit `decorative` contract** | `decorative` prop with JSDoc explaining when to use it — consumers make an intentional, documented choice | MUI has no `aria-hidden` shortcut; adding it requires `slotProps` digging |
| **CSS-var inset overrides** | Consumer overrides `--inset-start` / `--inset-end` inline or from a parent rule — no new prop, no component fork needed | MUI: changing inset distance requires `sx` or `styled(Divider)` — runtime CSS-in-JS involved |
| **Zero runtime CSS** | Single DOM node; `@layer kafui` BEM class; `--outline-variant` from token layer — no JS on any code path | MUI `Divider` still runs through `styled(…)`  + Emotion class generation |
| **Auto dark mode** | `--outline-variant` uses `light-dark()` — no extra rule, no `createTheme({ palette: { mode: 'dark' } })` | MUI dark mode needs a separate theme with `palette.mode = 'dark'`; applied at root |

---

## Actionable TODO Checklist

### CSS (`@kafui/styles`)

- [ ] Create `packages/styles/src/components/_divider.css`
- [ ] Wrap all rules in `@layer kafui { … }`
- [ ] Define `.divider` base with component-scoped vars: `--inset-start: 0px; --inset-end: 0px`
- [ ] Reset `<hr>` browser defaults: `border: none; margin: 0; padding: 0`
- [ ] Set line color: `background: var(--outline-variant)`
- [ ] `.divider--horizontal`: `display: block; block-size: 1px; inline-size: auto; margin-inline-start: var(--inset-start); margin-inline-end: var(--inset-end)`
- [ ] `.divider--vertical`: `display: inline-block; inline-size: 1px; block-size: auto; align-self: stretch; margin-block-start: var(--inset-start); margin-block-end: var(--inset-end)`
- [ ] `.divider--inset`: `--inset-start: 72px` (aligns with list text after avatar + padding)
- [ ] `.divider--middle-inset`: `--inset-start: 16px; --inset-end: 16px`
- [ ] Verify `<hr>` default browser styles fully reset (Chrome/Firefox/Safari add `margin: 8px 0; border: 1px inset`)
- [ ] No dark-mode override needed — `--outline-variant` flips via `light-dark()` in the token layer

### React Component (`@kafui/react`)

- [ ] Create `Divider.tsx` wrapping RAC `<Separator>`
- [ ] Pass `orientation` to RAC `Separator` prop; it selects element type and sets `aria-orientation` automatically
- [ ] When `decorative={true}`, add `aria-hidden="true"` to rendered element
- [ ] Compose BEM class string: `divider divider--{variant} divider--{orientation}`
- [ ] Export as named export `Divider` from `packages/react/src/index.ts`

### Accessibility
- [ ] Verify `<hr>` renders for horizontal (valid HTML5 void element)
- [ ] Verify `<div role="separator" aria-orientation="vertical">` renders for vertical (not `<hr>`)
- [ ] Test VoiceOver (macOS Safari): horizontal separator announced as "separator" between sections
- [ ] Test NVDA (Windows Chrome): vertical separator announced correctly in a toolbar row
- [ ] Test `decorative={true}` — confirm AT skips element entirely
- [ ] Test `decorative={false}` (default) — confirm AT announces separator

### Testing
- [ ] Unit: correct BEM class combinations for all `variant` × `orientation` values (3 × 2 = 6 combos)
- [ ] Unit: `decorative={true}` → rendered element has `aria-hidden="true"`
- [ ] Unit: `decorative={false}` (default) → no `aria-hidden` attribute
- [ ] Unit: vertical → rendered element is not `<hr>`; has `role="separator"`; has `aria-orientation="vertical"`
- [ ] Unit: horizontal → rendered element is `<hr>` (no explicit role needed)
- [ ] Unit: `--inset` class → `--inset-start: 72px` applied (CSS custom property assertion)
- [ ] Unit: `--middle-inset` class → both `--inset-start` and `--inset-end: 16px`
- [ ] a11y: axe-core — horizontal and vertical, decorative and structural
- [ ] Visual regression: full-width, inset, middle-inset; horizontal + vertical in flex row; light + dark; LTR + RTL (confirm inset flips)

### Open Questions
- [ ] **Default inset for `"inset"` variant:** Current default is 72 dp (aligns with list text column after avatar+padding). M3 also uses 56 dp (icon-only list). Should the default be configurable? Resolution proposal: keep 72 dp default (most common, safest); consumer overrides `--inset-start: 56px` via CSS for icon-only contexts.
- [ ] **Vertical divider in non-flex contexts:** `align-self: stretch` only works in flex/grid. Document that vertical dividers require a flex/grid parent. Consider adding a dev warning if the computed height is 0.
- [ ] **`<List divided>` integration:** The `List` component's `divided` prop injects `<Divider variant="inset">` automatically. Ensure `Divider` is importable as an internal dependency of `List` without a circular reference (same package, internal import is fine).
- [ ] **Inset for video thumbnail rows:** List items with `--leading-video` (100 dp width + 16 dp padding = 116 dp total). If a list mixes video thumbnails with icons, the inset distance must match the widest leading element. Consider adding `--leading-video` as a documented inset override: `style="--inset-start: 116px"`.
