# kafUI — Deep Review & Consolidation

Review of all **37 M3 component specs** (`temp/components/<slug>/{SPEC,TODO}.md`)
plus the two contract docs (`_TOKENS.md`, `INDEX.md`). Done by 10 parallel
sub-agents (≈3–4 components each), then a single consolidation pass that enforced
cross-component consistency and rewrote the contracts.

Three goals per component: (1) deep-review the SPEC against M3 for fidelity +
strongest modern API; (2) reframe TODOs from "differences from MUI" into concrete
**ways kafUI beats MUI** (shadcn/HeroUI spirit); (3) fix the CSS naming.

---

## 1. The CSS naming reset (the biggest mechanical change)

The specs had **prefix soup** *and* an internal inconsistency — some components
used `.kafui-button` + `--md-sys-color-*`, others `.md-chip` + `md.sys.color.*`.
Both systems are gone. The rule now, applied to all 37 + the token contract:

| Concern | Before | After |
|---|---|---|
| Seed color | `--md-source` | `--source` |
| System color role | `--md-sys-color-on-surface-variant` | `--on-surface-variant` |
| Reference palette | `--md-ref-neutral-20` | `--ref-neutral-20` |
| Shape | `--md-sys-shape-corner-full` | `--corner-full` |
| State opacity | `--md-sys-state-hover-state-layer-opacity` | `--state-hover` |
| Elevation | `--md-sys-elevation-level3` | `--elevation-3` |
| Motion | `--md-sys-motion-easing-emphasized` | `--easing-emphasized` |
| Typescale | `--md-sys-typescale-label-large-size` | `--label-large-size` |
| Block class | `.kafui-button` / `.md-chip` | `.button` / `.chip` |
| State layer | `.kafui-state-layer` | `.state-layer` |
| Component-internal var | (often re-prefixed) | scoped inside block: `.card { --pad: 16px }` |

**Collision safety lives on `@layer`, not on names.** Everything is wrapped in
`@layer kafui` (ordered `reset, tokens, base, components, utilities`). Generic
class names like `.card`/`.list`/`.menu` are safe because layered styles always
lose specificity to unlayered author CSS — so consumers can override with a plain
`.card { … }`. The brand name appears **only** on the layer.

The M3-role→token **mapping tables** kept in a few specs (e.g. `badge`,
`progress-indicator`) — `| md.sys.color.error | --error | … |` — are retained on
purpose: they're provenance/anchor docs, not soup.

---

## 2. Ratified cross-cutting decisions

These were surfaced repeatedly by the workers and are now locked in `INDEX.md`'s
*Cross-cutting API conventions* and `_TOKENS.md`:

1. **`variant` vs `color`.** `variant` changes anatomy/structure (Button, Card,
   TextField); `color` is used only where anatomy is identical and the role is the
   sole axis (FAB, Toolbar). This resolves the "why does FAB use `color` but Button
   uses `variant`" tension cleanly.
2. **Label model.** Item label = `children`; group label = `<Label>` child; helper
   text = `<Text slot="description">`; errors = `<FieldError>` + `isInvalid` +
   `validate`. The deprecated `validationState` was removed from `text-field` (it
   was the single worst correctness bug found). `errorMessage` is `React.ReactNode`.
3. **Canonical field anatomy.** `text-field` owns the container/label/supporting/
   error CSS; `date-picker` and `time-picker` input variants compose `.text-field`
   classes instead of cloning them.
4. **State via `data-*`, not className toggling.** RAC attributes drive all
   hover/press/focus/selected styling. Density via `data-density="-2"` (kills the
   confusing double-dash `--density--2`).
5. **State-layer implementation.** Default = `::before` pseudo (no extra node);
   real `.state-layer` child only when a block has independently focusable
   descendants that must stack above it (documented exception: `list`).
6. **Compound vs named export.** Dot-notation (`Menu.Item`) when the part has no
   standalone use; named export (`ButtonGroupItem`) when it does.
7. **Shared modal contract.** Dialog + modal bottom/side sheet share RAC
   `ModalOverlay isDismissable` (scrim-tap/Escape/focus-trap/scroll-lock/return-
   focus) and a single `.scrim` partial tinted with `color-mix(in srgb,
   var(--scrim) 32%, transparent)`. No `-rgb` split-channel hacks (incompatible
   with OKLCH-derived tokens).
8. **Shared hooks** in `@kafui/react`: `useScrollSentinel(target, { mode:
   'position' | 'direction' })` powers top-app-bar elevation/collapse,
   bottom-app-bar/toolbar hide-on-scroll, and search-bar elevation;
   `useResponsiveVariant` powers sheet `promoteTo`; `useExtendedFabCollapse`.
9. **Navigation semantics.** bar/rail/drawer render `<nav>` + links +
   `aria-current`; **Tabs are `role="tablist"`, not a `<nav>`** — a deliberate,
   documented distinction.
10. **Dark-mode elevation.** No separate tint overlay; the M3-2024
    `--surface-container-*` tiers carry tonal elevation, `--elevation-N` is shadow
    only. Resolves the open FAB/Card/Dialog question once.
11. **Tokens added to the contract** that components needed but were missing:
    `--surface-variant`, the explicit motion ladder (`--duration-*`/`--easing-*`),
    and a `--z-*` stacking tier (`--z-modal`, `--z-snackbar`, `--z-tooltip`, …).
12. **Browser baseline** set (Chrome 120+/FF 121+/Safari 17.4+ — OKLCH relative
    color, `light-dark()`, `@layer`, logical props). `:has()` and scroll-driven
    effects degrade to a React-set `data-*` attribute / `useScrollSentinel`.
13. **Shape morph at pill rest.** Components resting at `--corner-full` dip toward
    `--corner-large` on press and back (full→full was a no-op).

---

## 3. Top "beat-MUI" opportunities (library-wide)

Ranked by impact — these are the wins to lead the docs/marketing with:

1. **Zero-runtime CSS vs Emotion.** Every component ships static CSS in
   `@layer kafui`; no per-render style injection, no Emotion dependency, SSR-clean.
   Benchmarkable with `size-limit`. This is the whole-library thesis.
2. **One `--source` → full themed palette** via OKLCH `light-dark()`, vs MUI's
   `createTheme` + per-component `styleOverrides`. A tenant theme is one line.
3. **Free range date picker + non-Gregorian calendars.** `@mui/x-date-pickers-pro`
   gates range behind a paid license; kafUI's `DateRangePicker` is MIT in the same
   package, and Islamic/Hebrew/Japanese/Buddhist calendars work via `I18nProvider`
   with zero adapter setup. The single most commercially pointed win.
4. **Flat field API, automatic ARIA wiring.** RAC wires `aria-describedby`/
   `aria-invalid`/`aria-required` with no manual `id`/`htmlFor`; replaces MUI's
   `inputProps`/`InputProps`/`InputLabelProps`/`FormHelperTextProps` split — MUI's
   most-complained-about DX issue.
5. **Components MUI simply doesn't have**: Navigation Rail, M3 Carousel (CSS
   scroll-snap + `animation-timeline: view()`, no embla/swiper), rich Tooltip,
   bottom/side sheets with detent/snap-points, M3 Search bar→view, Toolbar/
   button-group/split-button/fab-menu (Expressive), and the M3-Expressive
   morphing Loading Indicator.
6. **Correct semantics out of the box**: `role="switch"` (MUI ships
   `role="checkbox"`), filter chips as `ToggleButton` not grid rows, segmented
   button as a real `radiogroup` in single-select (MUI gets this wrong),
   `menuitemradio`/`menuitemcheckbox`, real submenus, `aria-current` links.
7. **Built-in imperative snackbar queue** (`add()`→key, `close(key)`) replaces the
   notistack dependency entirely; `priority` maps to ARIA urgency, not visual color.
8. **Trigger-as-composition for overlays** (`DialogTrigger`) removes the
   `useState(false)` boilerplate every MUI dialog/drawer/menu carries.

---

## 4. Per-category highlights

- **Actions (button, fab, extended-fab, icon-button):** icon-only via optional
  `children` + auto modifier; FAB uses `color`; `<ToggleIconButton>` gets
  `aria-pressed` free + `selectedIcon` kills duplicated state; Expressive shape
  morph in CSS.
- **Button groups (segmented, button-group ✦, split ✦, fab-menu ✦):** correct
  selection ARIA; split-button is 1 import / ~8 lines vs MUI's ~70-line recipe;
  fab-menu keeps labels always-visible (fixes SpeedDial touch UX) with
  `role="menu"`.
- **Selection (checkbox, radio, switch, slider):** `children` label everywhere;
  type-safe slider range inferred from value type (no `isRange` footgun); CSS-only
  switch handle morph; SVG checkmark draw; `--state-focus` fixed to 0.10.
- **Chip/menu/list/divider:** chip normalized off the `.md-` prefix; 2-component
  list vs MUI's 7; first-class submenus; correct `<hr>`/`<div>` divider element.
- **Communication (badge, progress, snackbar, loading ✦):** positive `visible`
  prop; one progress component (not two); built-in snackbar queue; genuine M3
  Expressive morph with a `complete`/`onExitComplete` lifecycle.
- **Overlays (tooltip, dialog, bottom/side sheet):** unified rich-tooltip
  dispatch; detent/snap bottom sheet (MUI has none); `role="complementary"`
  standard side sheet; shared scrim + dismiss contract.
- **Fields/pickers (text-field, date, time):** the field-anatomy + ARIA-wiring +
  free-range story above; SSR-safe mode/locale handling.
- **App bars (top, bottom, toolbar ✦):** one `useScrollSentinel` mechanism, CSS-
  only medium/large collapse, vibrant Toolbar color variants with on-container
  icon correction, correct `role="toolbar"` placement.
- **Navigation (bar, rail, drawer, tabs):** shared animated active-indicator
  (transform, no reflow), `<nav>`+`aria-current` links, Tabs `keyboardActivation
  ="manual"` to avoid arrow-key data fetches.
- **Card/carousel/search:** compound Card (overlay pattern for nested
  interactives, the canonical compound template), scroll-driven carousel,
  search bar↔view as one component with on-scroll elevation.

---

## 5. Open decisions for a human call

Everything above is resolved; these are genuine product/scope choices, not
ambiguities:

- **Class-name branding (please confirm).** Consolidation dropped the brand from
  class names (`.button`, not `.kafui-button`), relying on `@layer kafui` for
  isolation — matching the `.btn { --radius }` example in the brief. If you want
  `.kafui-`-prefixed classes kept, this is the one decision that reverts cleanly.
- **`enableDrag` on Switch, `buffer` progress variant, side/bottom-sheet
  `promoteTo`** — currently scoped as opt-in / v-next; confirm they're out of v1.
- **Package layout.** Specs reference `@kafui/styles`, `@kafui/react`,
  `@kafui/hooks` — the `packages/` workspace should be scaffolded to match before
  implementation (separate task).
</content>
