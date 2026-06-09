# kafUI — Material 3 Component Index

Master catalog of every M3 component kafUI targets, the React Aria primitive each
builds on, and the kafUI compound surface. ✦ = Material 3 Expressive (2025).

Each component has a `SPEC.md` (anatomy/variants/states/tokens/a11y/API) and a
`TODO.md` (concrete ways kafUI **beats MUI** — DX/fidelity/perf/theming wins, in
the shadcn/HeroUI spirit) in `temp/components/<slug>/`.

See [`_TOKENS.md`](./_TOKENS.md) for the single-source-color OKLCH palette system
that every component's `SPEC.md` references by **plain role name** (`primary`,
`on-surface`, `outline`…). All CSS lives in `@layer kafui`; the brand name appears
only on that layer, never on a token or class.

## Actions
| Component | Slug | React Aria base | kafUI compound surface |
|---|---|---|---|
| Common button | `button` | `Button` | `<Button variant>` |
| FAB | `fab` | `Button` | `<Fab size color>` |
| Extended FAB | `extended-fab` | `Button` | `<ExtendedFab>` |
| Icon button | `icon-button` | `ToggleButton` / `Button` | `<IconButton variant toggle>` |
| Segmented button | `segmented-button` | `ToggleButtonGroup` | `<SegmentedButtonGroup>` / `<SegmentedButton>` |
| Button group ✦ | `button-group` | `ToggleButtonGroup` / `Group` | `<ButtonGroup>` |
| Split button ✦ | `split-button` | `Group` + `MenuTrigger` | `<SplitButton>` / `.Menu` |
| FAB menu ✦ | `fab-menu` | `MenuTrigger` + `Button` | `<FabMenu>` / `.Item` |

## Communication
| Component | Slug | React Aria base | kafUI compound surface |
|---|---|---|---|
| Badge | `badge` | (presentational) | `<Badge>` wrapper |
| Progress indicator | `progress-indicator` | `ProgressBar` | `<ProgressIndicator variant>` |
| Snackbar | `snackbar` | `ToastRegion` / `Toast` | `<SnackbarRegion>` + `queue` |
| Tooltip | `tooltip` | `TooltipTrigger` / `Tooltip` | `<Tooltip>` / rich `<Tooltip variant="rich">` |
| Loading indicator ✦ | `loading-indicator` | `ProgressBar` | `<LoadingIndicator>` |

## Containment
| Component | Slug | React Aria base | kafUI compound surface |
|---|---|---|---|
| Bottom sheet | `bottom-sheet` | `Modal` + `Dialog` | `<BottomSheet>` (modal/standard) |
| Card | `card` | (pressable container) | `<Card>` / `.Media` `.Header` `.Content` `.Actions` |
| Carousel | `carousel` | custom (listbox/scroll-snap) | `<Carousel>` / `.Item` |
| Dialog | `dialog` | `Modal` + `Dialog` + `DialogTrigger` | `<Dialog>` / `.Title` `.Content` `.Actions` |
| Divider | `divider` | `Separator` | `<Divider variant orientation>` |
| List | `list` | `ListBox` / `GridList` | `<List>` / `.Item` |
| Side sheet | `side-sheet` | `Modal` + `Dialog` | `<SideSheet>` (modal/standard) |

## Navigation
| Component | Slug | React Aria base | kafUI compound surface |
|---|---|---|---|
| Bottom app bar | `bottom-app-bar` | `Toolbar` | `<BottomAppBar>` |
| Navigation bar | `navigation-bar` | `<nav>` + links (`aria-current`) | `<NavigationBar>` / `.Item` |
| Navigation drawer | `navigation-drawer` | `<nav>` (+`Modal` when modal) | `<NavigationDrawer>` / `.Item` |
| Navigation rail | `navigation-rail` | `<nav>` + links | `<NavigationRail>` / `.Item` |
| Search | `search` | `SearchField` + `Autocomplete` | `<SearchBar>` → `<SearchView>` |
| Tabs | `tabs` | `Tabs`/`TabList`/`Tab`/`TabPanel` | `<Tabs>` / `.List` `.Tab` `.Panel` |
| Top app bar | `top-app-bar` | region/`Toolbar` | `<TopAppBar variant>` |
| Toolbar ✦ | `toolbar` | `Toolbar` | `<Toolbar variant>` |

## Selection
| Component | Slug | React Aria base | kafUI compound surface |
|---|---|---|---|
| Checkbox | `checkbox` | `Checkbox` / `CheckboxGroup` | `<Checkbox>` |
| Chip | `chip` | `TagGroup`/`Tag` · `ToggleButton` | `<ChipGroup>` / `<Chip variant>` |
| Date picker | `date-picker` | `DatePicker`/`Calendar`/`RangeCalendar` | `<DatePicker variant>` |
| Menu | `menu` | `Menu` / `MenuTrigger` | `<Menu>` / `.Item` `.Submenu` |
| Radio button | `radio-button` | `RadioGroup` / `Radio` | `<RadioGroup>` / `<Radio>` |
| Slider | `slider` | `Slider`/`SliderThumb`/`SliderTrack` | `<Slider>` (single/range) |
| Switch | `switch` | `Switch` | `<Switch>` |
| Time picker | `time-picker` | `TimeField` + custom dial | `<TimePicker variant>` |

## Text input
| Component | Slug | React Aria base | kafUI compound surface |
|---|---|---|---|
| Text field | `text-field` | `TextField`/`Label`/`Input`/`FieldError` | `<TextField variant>` |

---

**Totals:** 37 components — Actions 8, Communication 5, Containment 7,
Navigation 8, Selection 8, Text input 1. M3 Expressive additions: button-group,
split-button, fab-menu, loading-indicator, toolbar.

## Cross-cutting API conventions
These are library-wide invariants. A component deviates only with an explicit
justification in its own SPEC.

**Props & naming**
- **`variant` vs `color`.** `variant` carries an M3 named variant that changes
  *anatomy/structure* (e.g. `filled | outlined | text`); never split into boolean
  props. `color` is used **only** where anatomy is identical and the role is the
  sole difference (e.g. FAB, Toolbar). Both are closed string unions, never
  arbitrary palette names.
- **React Aria naming everywhere:** `onPress` not `onClick`; `isDisabled`,
  `isSelected`, `isInvalid`, `isReadOnly`; `onSelectionChange`, `onOpenChange`.
- **`onPress`** unifies pointer/keyboard/touch; `aria-disabled` keeps disabled
  controls focusable (never the bare HTML `disabled` attribute).

**Composition**
- **Compound components** mirror M3 anatomy (`Card.Media`, `Dialog.Actions`,
  `Menu.Item`). Use dot-notation sub-components when the part has **no** standalone
  use; use a named export (e.g. `ButtonGroupItem`) when it is independently usable.
  No monolithic prop bags, no `slotProps` soup.
- **Icons** via sprite-backed `<Icon name="..." />` — no per-icon JS modules.
- **Icon-only controls require `aria-label` at the type level** (discriminated
  union when a visible label may be absent).

**Forms & fields**
- **Item label = `children`.** Group label = RAC `<Label>` child (or `label` prop
  on the group). Helper text = `<Text slot="description">`. Errors = RAC
  `<FieldError>` driven by `isInvalid` + `validate`; **never** the deprecated
  `validationState`. `errorMessage`/error content is typed `React.ReactNode`.
- **Canonical field anatomy** (container + label + supporting-text + error) is
  defined once by `text-field`; `date-picker`/`time-picker` input variants compose
  its `.text-field` classes rather than duplicating CSS.

**State, styling & theming**
- **State via `data-*`, not className toggling.** RAC `data-hovered|pressed|
  focus-visible|selected|disabled` drive CSS. BEM modifiers only for what RAC has
  no attribute for. Density via `data-density="-2"` (no `--density--2`).
- **Zero inline styles in JSX** (the only exception is a single CSS custom property
  carrying a runtime value, e.g. `style={{ '--_value': pct }}`). All visuals live
  in `@kafui/styles` as static, zero-runtime CSS in `@layer kafui`.
- **Tokens** referenced by plain role name; one `--source` seed re-derives the
  palette; `light-dark()` + `color-scheme` for dark mode (no `.dark` duplication).

**Overlays & shared behavior**
- **Modal pattern** (dialog, modal bottom/side sheet) shares the RAC
  `ModalOverlay isDismissable` contract (scrim-tap + Escape + focus-trap +
  scroll-lock + return-focus) and the shared `.scrim` utility. Stacking via `--z-*`.
- **Shared hooks** in `@kafui/react`: `useScrollSentinel(target, { mode })` (app
  bars / search elevation+hide), `useResponsiveVariant` (sheet promotion),
  `useExtendedFabCollapse`.
- **`<nav>` + `aria-current` for navigation** (bar/rail/drawer render links);
  **Tabs are `role="tablist"`**, explicitly *not* a `<nav>` landmark.

**i18n/RTL** via React Aria `I18nProvider` + CSS logical properties; off-canvas
surfaces use the writing-mode-aware `translate` property; nothing hardcoded LTR.
