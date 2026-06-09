# Icon Button — TODO

## MUI equivalent
`@mui/material/IconButton` (non-toggle). For toggle: `@mui/material/ToggleButton` (designed for groups, heavyweight for a solo icon toggle) or `IconButton` + manual `useState` + manual `aria-pressed`.

MUI IconButton key reference: single visual style (no M3 variants), `color` prop accepts palette keys (`primary`, `secondary`, `default`, `inherit`, `error`, `info`, `success`, `warning`) — not M3 roles, no `selectedIcon` concept, `aria-label` optional in types, Emotion CSS-in-JS, HTML `disabled` attribute used.

---

## Beat-MUI opportunities

### 1. Four M3 variants with full toggle color matrices — MUI has zero
**What we do:** four variants (`standard`, `filled`, `filled-tonal`, `outlined`), each with correct unselected and selected color token pairs, all expressed as CSS modifier classes + `data-selected` attribute selectors. No runtime logic — CSS does all the work.

**Why it wins:** MUI has exactly one visual style for `IconButton`. Getting a "filled" icon button in MUI means writing `sx={{ bgcolor: 'primary.main', … }}` and manually tracking the selected state in JS. That is 20+ lines of application code for something that should be 1 prop. Our API: `<ToggleIconButton variant="filled" />` — done.

**Task:** Implement all four variant modifier classes with both unselected and selected color pairs in CSS. Storybook story with a 4×2 grid (variant × selected/unselected) for visual validation.

### 2. `<ToggleIconButton>` as a first-class component — correct `aria-pressed` without effort
**What we do:** `<ToggleIconButton>` builds on RAC `ToggleButton`, which automatically manages `aria-pressed`, `isSelected` controlled/uncontrolled patterns, and `onChange`. Consumers get correct toggle semantics for free.

**Why it wins:** MUI has no toggle icon button component. The standard MUI pattern is `<IconButton onClick={() => setState(!state)} aria-pressed={state}>` — 3 separate concerns manually wired. This is done incorrectly in the wild more often than not (`aria-pressed` is forgotten, or set to a non-boolean string, or the state initialization is wrong). RAC's `ToggleButton` handles all of it; we just style it.

**Task:** Export `ToggleIconButton` from the package root. Write a RTL test asserting that `aria-pressed` changes from `"false"` to `"true"` on press without any consumer-side state management (uncontrolled case).

### 3. `selectedIcon` prop — icon swap without state management overhead
**What we do:** `selectedIcon?: string` — when `data-selected` is set and `selectedIcon` is provided, the component renders `selectedIcon` sprite instead of `icon`. No consumer state needed to track which icon to show.

**Why it wins:** the M3 "filled icon on select" pattern (e.g. `heart` → `heart-filled`, `bookmark` → `bookmark-filled`) is universally needed but universally reimplemented. In MUI, consumers track the icon name in their own state: `const [icon, setIcon] = useState('heart'); /* toggle in onClick */`. This state is a duplicate of `aria-pressed` — it represents the same truth twice. Our `selectedIcon` prop derives the icon from the controlled/uncontrolled selection state — single source of truth.

**Task:** Implement `selectedIcon` rendering logic: `const currentIcon = isSelected && selectedIcon ? selectedIcon : icon`. The `<Icon>` component receives this value. Write a Storybook story with `icon="bookmark"` and `selectedIcon="bookmark-filled"` demonstrating the swap.

### 4. `aria-label` required at the type level — compile-time a11y
**What we do:** `'aria-label': string` is non-optional in both `IconButtonProps` and `ToggleIconButtonProps`.

**Why it wins:** MUI's `aria-label` is `string | undefined` — an icon button without an accessible name is a WCAG Level A failure (1.1.1), but MUI will compile and ship it without warning. We make it a compile error. This is especially important for icon buttons because 100% of them are icon-only — there is no other text label to fall back to.

**Task:** Verify both type signatures have `'aria-label': string` as required. Add a custom ESLint rule or an a11y Storybook test that fails if `aria-label` is absent. Add to the component's JSDoc: _"aria-label is always required because icon buttons have no visible text label"_.

### 5. Touch target via CSS `::before` — no DOM wrapper needed
**What we do:** for `xs` (28 dp) and `sm` (32 dp) sizes, a `::before` pseudo-element with negative `inset` extends the hit area to 48 dp minimum. The visual container stays small.

**Why it wins:** MUI adds a ripple wrapper `<span>` inside icon buttons and relies on the icon button's `padding` for touch target size — this expands the visual container as well as the touch target. For Expressive small sizes, you cannot have a 28 dp visible button with 48 dp padding in MUI without custom CSS. Our `::before` approach is invisible to layout engines and requires zero extra DOM.

**Task:** Implement `::before` touch-target extension for `--size-xs` and `--size-sm`. Write a Storybook accessibility story that overlays the touch-target area visually (using a dev utility). Playwright test: simulate a tap 10 dp outside the `xs` container edge and assert the button receives the press event.

### 6. Static CSS for selected-state colors — MUI needs JS theme + `sx`
**What we do:** all selected-state color rules are in CSS, driven by RAC's `data-selected` attribute. E.g. `.icon-button--filled[data-selected] { --_bg: var(--primary); --_fg: var(--on-primary); }`. No JS involved.

**Why it wins:** in MUI, achieving the `filled` icon button selected state requires either: (a) an `sx` prop that reads theme colors (`sx={{ bgcolor: selected ? 'primary.main' : 'action.selected' }}`), or (b) a `styled()` component with template-literal props. Both compute styles at runtime, inject style tags, and require the theme context. Ours is a static CSS file — parsed once by the browser, cached indefinitely.

**Task:** Implement all selected-state CSS rules per variant. Add a dark-mode visual test: verify `filled-tonal` selected in dark mode uses the correct dark-scheme `secondary-container` value (derived from `light-dark()` in tokens).

### 7. Expressive width modes — `narrow`/`wide` with a single class
**What we do:** `.icon-button--narrow` and `.icon-button--wide` BEM modifiers override `inline-size` via `calc(var(--_sz) * factor)`, producing rectangular pill shapes from the same square token.

**Why it wins:** MUI has no concept of icon button width modes. Implementing a "wide" icon button touch in MUI requires `sx={{ width: '56px', height: '40px', borderRadius: '50%' }}` — magic numbers hardcoded in application code, disconnected from any token system. Our modifiers scale off `--_sz`, so changing the size also changes the width proportionally.

**Task:** Implement `.icon-button--narrow` and `.icon-button--wide` with the ratio-based `calc`. Verify proportions at `xs`, `md`, and `xl` sizes. Add the width-mode transition under `@media (prefers-reduced-motion: no-preference)`.

### 8. `outlined` selected state: `inverse-surface` — spec-correct, not guesswork
**What we do:** `.icon-button--outlined[data-selected]` sets `--_bg: var(--inverse-surface)` and `--_fg: var(--inverse-on-surface)`, and removes the border. This is the exact M3 token mapping for outlined toggle selected state.

**Why it wins:** this is a subtle but important M3 detail that MUI (which has no `outlined` icon button variant) completely ignores, and that most hand-rolled implementations get wrong — they typically use `primary` as the selected background, which is off-spec and can fail contrast requirements in both light and dark schemes. `inverse-surface` is a high-contrast, scheme-aware token designed exactly for this use case.

**Task:** Implement the `outlined[data-selected]` CSS rule. Verify contrast ratios in both light and dark via Storybook a11y panel. Note: in dark mode, `inverse-surface` is a light color — confirm the button is visually coherent (light container + dark icon).

---

## Checklist

- [ ] **[Beat #1]** Four variant classes + selected color pairs; 4×2 Storybook grid story.
- [ ] **[Beat #2]** Export `ToggleIconButton`; RTL test for automatic `aria-pressed` management.
- [ ] **[Beat #3]** `selectedIcon` prop rendering logic; Storybook story with icon swap demo.
- [ ] **[Beat #4]** Non-optional `aria-label`; ESLint/Storybook a11y check; JSDoc note.
- [ ] **[Beat #5]** `::before` touch target for `xs`/`sm`; Playwright hit-area test.
- [ ] **[Beat #6]** Static CSS selected-state rules; dark-mode visual test for `filled-tonal`.
- [ ] **[Beat #7]** `narrow`/`wide` modifiers with ratio-based `calc`; width-mode transition.
- [ ] **[Beat #8]** `outlined[data-selected]` with `inverse-surface`; contrast verification.
- [ ] State-layer element driven by `data-hovered`, `data-focused`, `data-pressed`.
- [ ] All variants: elevation level 0 only — no box-shadow on icon buttons.
- [ ] Focus ring on `:focus-visible` only; no ring on mouse press.
- [ ] Document `ToggleButton` sets `aria-pressed` automatically — do NOT set it manually.
- [ ] Confirm `--surface-variant` is defined in `_TOKENS.md` (cross-cutting, raise with lead).
- [ ] Expressive size modifiers `--size-xs` through `--size-xl` with correct square dimensions.
- [ ] `standard` disabled: transparent background (no container tint), only icon tinted.
