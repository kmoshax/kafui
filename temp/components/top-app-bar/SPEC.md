# Top App Bar

The top app bar displays information and actions relating to the current screen. It sits at the top of the screen and provides consistent, scannable navigation and contextual actions. M3 category: **Navigation → Top App Bar**.

---

## Anatomy / parts

| BEM element | Description |
|---|---|
| `.top-app-bar` | Root `<header>` container; full-width horizontal bar |
| `.top-app-bar__row` | The fixed-height 64 dp inner flex row containing leading, inline headline (when applicable), and trailing |
| `.top-app-bar__leading` | Leading slot — navigation icon button (back, close, hamburger) |
| `.top-app-bar__headline` | Title/headline text; may be in `__row` (small/center-aligned) or in `__content` (medium/large resting) |
| `.top-app-bar__trailing` | Trailing slot — row of up to 3 icon buttons; NOT wrapped in `role="toolbar"` |
| `.top-app-bar__content` | Expanded lower section (medium/large only); contains `__headline`; collapses on scroll |

State layer is **not** on the bar container — only on the individual icon buttons within.

---

## Variants

| Variant | Resting height | Scrolled height | Headline position (resting) | Headline typescale | `variant` prop |
|---|---|---|---|---|---|
| `small` | 64 dp | 64 dp | Inline in `__row`, `inline-start` | `title-large` | `"small"` (default) |
| `center-aligned` | 64 dp | 64 dp | Inline in `__row`, centered | `title-large` | `"center-aligned"` |
| `medium` | 112 dp | 64 dp | In `__content` (48 dp tall) below `__row` | `headline-small` | `"medium"` |
| `large` | 152 dp | 64 dp | In `__content` (88 dp tall) below `__row` | `headline-medium` | `"large"` |

`center-aligned` and `small` never have a `__content` section. On medium/large, the headline text is present in BOTH the `__content` section (expanded) and a second `.top-app-bar__headline--collapsed` clone inside `__row` that is invisible at rest and fades in when scrolled, so the screen reader announcement never duplicates (`aria-hidden` on the collapsed clone).

---

## States

### On-scroll behaviour

| Scroll state | Visual change |
|---|---|
| At top (not scrolled) | Resting height; background `--surface`; no shadow |
| Scrolled — any variant | Background `--surface-container`; `box-shadow` from `--elevation-2`; `medium`/`large` collapse `__content` to height 0 |

The modifier `.top-app-bar--scrolled` drives everything via CSS. The component does not mount its own scroll listener. Two consumer patterns are supported:

1. **Controlled** — consumer passes `isScrolled` prop; component adds/removes `.top-app-bar--scrolled`.
2. **Scroll-sentinel (opt-in)** — consumer passes `scrollTarget` prop; component mounts a single `IntersectionObserver` on a zero-height sentinel `<div>` pinned to the top of `scrollTarget` and derives `isScrolled` internally.

No other scroll-detection API. No `position` prop on the component.

### Bar-level modifier classes

| Modifier | Meaning |
|---|---|
| `.top-app-bar--scrolled` | Triggers background, shadow, collapse |
| `.top-app-bar--small` / `--center-aligned` / `--medium` / `--large` | Variant shape |

---

## Design tokens

All tokens are **unprefixed system roles** consumed via `@layer kafui`.

### Color
```css
.top-app-bar {
  --bg: var(--surface);
  background: var(--bg);
}
.top-app-bar--scrolled {
  --bg: var(--surface-container);
}
```

| Role | CSS custom property |
|---|---|
| Resting background | `--surface` |
| Scrolled background | `--surface-container` |
| Leading icon | `--on-surface` |
| Trailing icons | `--on-surface-variant` |
| Headline text | `--on-surface` |

### Typography

| Variant | Token set |
|---|---|
| `small` / `center-aligned` | `--title-large-font`, `--title-large-size`, `--title-large-weight`, `--title-large-line-height` |
| `medium` (expanded headline) | `--headline-small-font`, `--headline-small-size`, `--headline-small-weight`, `--headline-small-line-height` |
| `large` (expanded headline) | `--headline-medium-font`, `--headline-medium-size`, `--headline-medium-weight`, `--headline-medium-line-height` |
| Collapsed inline headline (medium/large, scrolled) | `--title-large-font`, `--title-large-size`, `--title-large-weight`, `--title-large-line-height` |

### Elevation
- Resting: `--elevation-0`
- Scrolled: `--elevation-2`

### Motion
- Background + shadow transition: `transition-duration: var(--duration-short4)`, `transition-timing-function: var(--easing-standard)`
- `__content` collapse (medium/large): `transition-duration: var(--duration-medium2)`, `transition-timing-function: var(--easing-emphasized)`
- Reduced motion: `@media (prefers-reduced-motion: reduce)` — instant, no transition

### Shape
No border radius — bar spans full viewport width.

### Component-internal vars (inside `.top-app-bar {}`)
```css
.top-app-bar {
  --h: 64px;        /* bar row height; never changes */
  --content-h: 0px; /* overridden per variant: medium=48px, large=88px */
  --bg: var(--surface);
}
```

---

## CSS structure (in `@layer kafui`)

```css
@layer kafui {
  .top-app-bar {
    --h: 64px;
    --content-h: 0px;
    --bg: var(--surface);

    display: flex;
    flex-direction: column;
    width: 100%;
    background: var(--bg);
    box-shadow: var(--elevation-0);
    transition:
      background-color var(--duration-short4) var(--easing-standard),
      box-shadow       var(--duration-short4) var(--easing-standard);
  }

  .top-app-bar--medium  { --content-h: 48px; }
  .top-app-bar--large   { --content-h: 88px; }

  .top-app-bar--scrolled {
    --bg: var(--surface-container);
    box-shadow: var(--elevation-2);
  }

  .top-app-bar__row {
    display: flex;
    align-items: center;
    height: var(--h);
    padding-inline: 4px;
    flex-shrink: 0;
  }

  .top-app-bar__leading {
    width: 48px;
    height: 48px;
    display: flex;
    align-items: center;
    justify-content: center;
    flex-shrink: 0;
    color: var(--on-surface);
  }

  .top-app-bar__headline {
    flex: 1;
    color: var(--on-surface);
    font: var(--title-large-weight) var(--title-large-size) / var(--title-large-line-height) var(--title-large-font);
    padding-inline-start: 4px;
    overflow: hidden;
    text-overflow: ellipsis;
    white-space: nowrap;
  }

  .top-app-bar--center-aligned .top-app-bar__headline {
    text-align: center;
    padding-inline: 0;
  }

  .top-app-bar__trailing {
    display: flex;
    align-items: center;
    margin-inline-start: auto;
    color: var(--on-surface-variant);
  }

  /* medium / large expanded content area */
  .top-app-bar__content {
    overflow: hidden;
    max-block-size: var(--content-h);
    padding-inline: 16px;
    padding-block-end: 28px;
    transition:
      max-block-size var(--duration-medium2) var(--easing-emphasized),
      padding       var(--duration-medium2) var(--easing-emphasized);
  }

  .top-app-bar--scrolled .top-app-bar__content {
    max-block-size: 0;
    padding-block-end: 0;
  }

  /* expanded headline typography */
  .top-app-bar--medium .top-app-bar__content .top-app-bar__headline {
    font: var(--headline-small-weight) var(--headline-small-size) / var(--headline-small-line-height) var(--headline-small-font);
    white-space: normal;
  }

  .top-app-bar--large .top-app-bar__content .top-app-bar__headline {
    font: var(--headline-medium-weight) var(--headline-medium-size) / var(--headline-medium-line-height) var(--headline-medium-font);
    white-space: normal;
  }

  /* collapsed inline headline (medium/large only, hidden at rest) */
  .top-app-bar__headline--collapsed {
    opacity: 0;
    pointer-events: none;
    transition: opacity var(--duration-medium2) var(--easing-emphasized);
  }

  .top-app-bar--scrolled .top-app-bar__headline--collapsed {
    opacity: 1;
  }

  /* RTL */
  [dir="rtl"] .top-app-bar__trailing {
    margin-inline-start: 0;
    margin-inline-end: auto;
  }

  /* Reduced motion */
  @media (prefers-reduced-motion: reduce) {
    .top-app-bar,
    .top-app-bar__content,
    .top-app-bar__headline--collapsed {
      transition: none;
    }
  }
}
```

---

## Interaction & accessibility

**Landmark role:**
- Root element is `<header>` — implicit `banner` landmark. When multiple `<header>` elements exist, add `aria-label` to disambiguate (e.g. `aria-label="Page header"`).

**Headline:**
- Default: `<span>` (purely visual; page already has an `<h1>` in content area).
- When it IS the primary heading: `headingLevel` prop renders `<h1>`–`<h6>` with the same font override. Default is `undefined` (span).

**Leading icon button:**
- `aria-label` must describe the action: `"Navigate back"`, `"Open navigation menu"`, `"Close"`. Never a generic `"Menu"` unless it opens a drawer.
- Uses RAC `Button` with `onPress`.

**Trailing actions:**
- Individual `aria-label` per icon button (`"Search"`, `"More options"`, `"Share"`).
- Max 3 per M3 spec; overflow goes into a More-options menu.
- **NOT wrapped in `role="toolbar"`** — M3 trailing actions in the top app bar are not a toolbar in the ARIA sense (no arrow-key roving expected).

**Collapsed headline (medium/large):**
- The `__headline--collapsed` clone inside `__row` has `aria-hidden="true"` — the real accessible headline remains in `__content` (even when `max-block-size: 0`). Screen readers read the content-area heading; the collapsed visual copy is purely decorative.

**Focus order:** Leading icon → trailing icons in DOM order → main content. Bar is not in tab order except via its interactive children.

**RTL:** Leading/trailing slots swap visually via logical properties. Directional icons (back arrow, hamburger) mirrored via `:dir(rtl)` selector.

**Reduced motion:** `__content` collapse and background cross-fade are instant.

---

## Proposed kafUI React API

```tsx
// No direct react-aria-components equivalent for the shell.
// Icon buttons within slots use RAC Button.

type TopAppBarVariant = 'small' | 'center-aligned' | 'medium' | 'large';
type HeadingLevel = 1 | 2 | 3 | 4 | 5 | 6;

interface TopAppBarProps {
  variant?: TopAppBarVariant;           // default: 'small'
  headline?: React.ReactNode;           // title text or element
  headingLevel?: HeadingLevel;          // renders <h{n}>; default: undefined (span)
  leading?: React.ReactNode;            // leading icon button slot
  trailing?: React.ReactNode;           // trailing icon button(s) slot
  // Scroll control — two patterns, mutually exclusive:
  isScrolled?: boolean;                 // controlled; pass true when scrolled
  scrollTarget?: Element | null;        // opt-in sentinel: component mounts IntersectionObserver
  onScrolledChange?: (scrolled: boolean) => void; // fires when sentinel changes
  'aria-label'?: string;                // forwarded to <header>; recommended when multiple headers
  className?: string;
  // children intentionally omitted — use named slots
}
```

**BEM classes emitted:**
- `.top-app-bar` (always)
- `.top-app-bar--small` / `--center-aligned` / `--medium` / `--large`
- `.top-app-bar--scrolled` when scrolled
- `.top-app-bar__row`, `__leading`, `__headline`, `__headline--collapsed` (medium/large only), `__trailing`, `__content` (medium/large only)

**React Aria base:** No RAC primitive for the shell. Leading/trailing icon buttons use RAC `Button`. The shell renders as `<header>` with plain structural `<div>` children.

**Scroll sentinel:** When `scrollTarget` is provided, a `useScrollSentinel(scrollTarget)` hook is invoked internally (same hook exported for reuse by `BottomAppBar` and `Toolbar`). The hook creates a zero-height `<div>` appended to `scrollTarget` and returns `isScrolled: boolean` via `IntersectionObserver`. If `isScrolled` prop is also passed, it takes precedence (controlled wins).
