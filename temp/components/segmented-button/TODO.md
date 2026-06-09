# Segmented Button — TODO

## MUI Reference (what we are beating)

**MUI `ToggleButtonGroup` + `ToggleButton`** (`@mui/material/ToggleButtonGroup`)

MUI has a toggle-button group. It has no animated checkmark, no M3-correct radio semantics in
single-select, incorrect `aria-pressed` for all modes, Emotion-based styling, and a string `value`
prop that diverges from any RAC-based library. Every item below is a concrete way kafUI does it
better.

---

## Beat-MUI Opportunities

### 1. Correct ARIA semantics — MUI gets this wrong
**MUI**: uses `role="group"` + `role="button"` + `aria-pressed` for *both* selection modes — even
single-select, which should be `role="radiogroup"` + `role="radio"` + `aria-checked`.
**kafUI win**: RAC `ToggleButtonGroup` sets `role="radiogroup"` in `selectionMode="single"`,
`role="group"` in `"multiple"`. Screen readers announce the correct affordance ("radio group" vs
"group") and users understand whether they can select multiple items — without a single line of
aria wiring in the component.

**Tasks:**
- [ ] Validate RAC v1 emits `role="radiogroup"` for `selectionMode="single"`. If not, add
  `aria-roledescription` or a `role` override via RAC `slot` API and file an upstream issue.
- [ ] Validate multi-select emits `aria-pressed` or `aria-checked` consistently; document the
  decision in the component JSDoc.

### 2. Animated checkmark, CSS-only — MUI doesn't have one at all
**MUI**: no checkmark. Selection state is only communicated by fill color (not enough for
colorblind users and fails WCAG 1.4.1 non-color conveyed info).
**kafUI win**: the check icon is always in the DOM; `inline-size: 0 → 18px` + `opacity: 0 → 1`
transition is triggered by `[data-selected]` from RAC — zero JS, zero layout reflow, no state
toggles in React. Reduces re-renders to zero for the animation itself.

**Tasks:**
- [ ] Implement `.segmented-button__check-icon` with `inline-size` + `opacity` transition per SPEC.
- [ ] Test that the check icon is present in DOM (for AT enumeration) even when `opacity: 0`.
- [ ] Ensure icon-only segments: check replaces leading icon (leading-icon hidden via CSS when
  `[data-selected]` and no `children` text).
- [ ] Label+icon segments: check appears inline before the original icon (both visible).
- [ ] Add `@media (prefers-reduced-motion: reduce)` block that sets `transition: none` on both
  `__check-icon` and `.state-layer`.

### 3. CSS-first state layers — MUI injects Emotion on every interaction
**MUI**: Emotion injects ripple CSS on each interaction, generating new CSS classes per render.
**kafUI win**: state layer is a `::before` pseudo driven purely by RAC `data-hovered`,
`data-pressed`, `data-focus-visible` — no JS, no class mutation, no new CSS injected at runtime.
The `@layer kafui` layer ensures predictable specificity so consumers can override with a single
custom property redeclaration.

**Tasks:**
- [ ] Implement `.state-layer` (shared utility) as `::before` pseudo-element on
  `.segmented-button`: `content: ""; position: absolute; inset: 0; border-radius: inherit;
  background: currentColor; opacity: 0; transition: opacity 150ms`.
- [ ] Target RAC data-attributes: `[data-hovered] .state-layer { opacity: var(--state-hover) }`,
  `[data-focus-visible] .state-layer { opacity: var(--state-focus) }`,
  `[data-pressed] .state-layer { opacity: var(--state-pressed) }`.
- [ ] Verify state layer does NOT bleed outside pill border-radius (group `overflow: hidden` +
  `border-radius: var(--corner-full)` handles this).

### 4. `Set<Key>` selection model — MUI's string array is library-specific
**MUI**: `value: string | string[]` — diverges from any RAC-based selection pattern; consumers
who use RAC elsewhere (Tabs, Listbox, etc.) must maintain two mental models.
**kafUI win**: `onSelectionChange(keys: Set<Key>)` is identical to every other kafUI selection
component. Consumers destructure `[...keys][0]` for single-select or iterate for multi — one
pattern everywhere.

**Tasks:**
- [ ] `SegmentedButtonGroup` wraps RAC `ToggleButtonGroup`; forward `selectedKeys`,
  `defaultSelectedKeys`, `onSelectionChange`, `selectionMode`, `isDisabled` directly.
- [ ] Expose `onSelectionChange` (not `onChange`) to match RAC naming — avoids confusion for
  consumers already using RAC.
- [ ] TypeScript: `selectedKeys` / `defaultSelectedKeys` typed as `Iterable<Key>`;
  `onSelectionChange` typed as `(keys: Set<Key>) => void`.
- [ ] Dev-mode invariant: warn if `aria-label` and `aria-labelledby` are both absent.

### 5. `density` not `size` — M3-accurate, not MUI-approximated
**MUI**: `size="small"|"medium"|"large"` — implies different tap target; inconsistent with M3's
density model.
**kafUI win**: `density={0 | -1 | -2 | -3}` maps directly to M3's density spec (40/36/32/28 dp
height). The touch target stays at 48 dp via `min-block-size` regardless of density — no
accessibility regression at dense layouts.

**Tasks:**
- [ ] Apply `--density-0` through `--density--3` modifier class on group from `density` prop.
- [ ] Use `--h` CSS custom property inside `.segmented-button-group` block; each modifier
  overrides just `--h`. Items inherit via `block-size: var(--h)`.
- [ ] Ensure `min-block-size: 48px` is on the interactive element (not the group) for correct
  48-dp touch area regardless of density.
- [ ] Storybook: show all density levels in one story with touch-target overlay.

### 6. Icon-only a11y requirement — MUI silently omits
**MUI**: no warning when an icon-only toggle button is rendered without an accessible name.
**kafUI win**: TypeScript discriminated union enforces `aria-label` when `children` is absent.
Zero runtime surprises; accessibility is a compile-time guarantee.

**Tasks:**
- [ ] Implement discriminated union type for `SegmentedButtonProps` (see SPEC).
- [ ] Runtime `console.error` in dev mode as a safety net in case TS is bypassed (e.g. plain JS).

### 7. RTL and reduced motion — zero config
**MUI**: RTL detection is JS-based (`theme.direction`); reduced motion has no built-in handling.
**kafUI win**: all directional CSS uses logical properties (`padding-inline`, `border-start-*`).
`dir="rtl"` on any ancestor flips layout automatically. `@media (prefers-reduced-motion: reduce)`
block in CSS zeroes all transition durations — no JS, no theme toggle.

**Tasks:**
- [ ] Audit all directional CSS rules in the segmented-button stylesheet; replace every physical
  property (`margin-left`, `border-top-left-radius`, etc.) with its logical equivalent.
- [ ] Add `@media (prefers-reduced-motion: reduce)` block per SPEC.
- [ ] Storybook: RTL story using `<I18nProvider locale="ar">` + `dir="rtl"`.

---

## Implementation Checklist

### Styles (`@kafui/styles`)

- [ ] `.segmented-button-group`: flex row, `border: 1px solid var(--outline)`,
  `border-radius: var(--corner-full)`, overflow hidden, inline-flex.
- [ ] Internal dividers: `border-inline-end: 1px solid var(--outline)` on all segments except
  `:last-child` (CSS, no JS needed).
- [ ] `.segmented-button`: flex, align-center, `block-size: var(--h)`, `min-block-size: 48px`,
  `padding-inline: var(--_pad-inline)` (default 12px; 8px for `--icon-only`).
- [ ] `[data-selected]` modifier: `background: var(--secondary-container)`;
  `color: var(--on-secondary-container)`.
- [ ] `[data-disabled]` modifier: `opacity: 0.38; pointer-events: none`.
- [ ] `--icon-only` modifier: `padding-inline: 8px`.
- [ ] Density modifiers (see Beat-MUI #5).
- [ ] Check icon (see Beat-MUI #2).
- [ ] State layer (see Beat-MUI #3).
- [ ] RTL audit (see Beat-MUI #7).
- [ ] Reduced motion (see Beat-MUI #2 and #7).

### React Component (`@kafui/react`)

- [ ] `SegmentedButtonGroup` wraps RAC `ToggleButtonGroup` (see Beat-MUI #4).
- [ ] Density class applied from `density` prop (see Beat-MUI #5).
- [ ] `SegmentedButton` wraps RAC `ToggleButton`; always renders `__check-icon` span in DOM.
- [ ] `--icon-only` class added in JS when no `children` text passed.
- [ ] `aria-label` / `aria-labelledby` dev-mode guard (see Beat-MUI #6).
- [ ] TypeScript discriminated union (see Beat-MUI #6).
- [ ] Export `SegmentedButtonGroupProps`, `SegmentedButtonProps` from package index.

### Tests

- [ ] Single-select: selecting segment B deselects segment A; `onSelectionChange` called with
  `Set` containing only `B`.
- [ ] Multi-select: A and B can be selected simultaneously; deselect works independently.
- [ ] Keyboard: arrow keys move focus; Space/Enter select; Tab exits group.
- [ ] Disabled segment: not focusable, not selectable; `aria-disabled="true"` present.
- [ ] ARIA roles: `role="radiogroup"` in single-select; correct per Beat-MUI #1.
- [ ] Check icon: `inline-size` is `0` when unselected; non-zero when selected.
- [ ] Density: correct class applied; `--h` custom property value matches spec.
- [ ] `aria-label` required: TypeScript error when icon-only + no `aria-label`.
- [ ] Snapshot: single-select, multi-select, all density levels.

### Documentation / Storybook

- [ ] Story: single-select, label-only segments (e.g. "Day / Week / Month").
- [ ] Story: single-select, icon-only segments with `aria-label`.
- [ ] Story: multi-select, icon + label segments.
- [ ] Story: disabled individual segments + fully disabled group.
- [ ] Story: all four density levels side by side with touch-target overlay.
- [ ] Story: RTL layout via `I18nProvider`.
- [ ] Docs callout: "Unlike MUI `ToggleButtonGroup`, kafUI uses correct radiogroup ARIA in
  single-select mode and includes an animated checkmark for non-color selection feedback."
