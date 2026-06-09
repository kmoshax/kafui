# Carousel

A horizontally scrolling container for browsing a collection of related items. M3 category: **Containment → Carousel**. Items resize dynamically as they enter and leave the focal viewport area. Scroll is driven by native CSS scroll-snap; no auto-play. Four M3 layout strategies differ in item sizing and the number of visible items.

---

## Anatomy / parts

| BEM element | Description |
|---|---|
| `.carousel` | Root; `overflow-x: scroll; scroll-snap-type`; also the `container-type: inline-size` anchor for container queries |
| `.carousel__track` | Inner semantic `<ul>` / `role="list"` holding all items; `display: flex; gap` |
| `.carousel__item` | Individual item (`<li>`); carries `scroll-snap-align`; resizes per layout via CSS custom properties |
| `.carousel__item-inner` | Inner content wrapper; `overflow: hidden; border-radius: inherit` |
| `.carousel__item-media` | Image/video region within item |
| `.carousel__item-label` | Text overlay or caption below media |
| `.carousel__prev-button` | Optional previous-item navigation button (hidden on coarse-pointer / touch) |
| `.carousel__next-button` | Optional next-item navigation button (hidden on coarse-pointer / touch) |
| `.carousel__indicators` | Optional dot/pill indicator row |
| `.carousel__indicator-dot` | Single dot/pill; `.carousel__indicator-dot--active` on current item |
| `.state-layer` | Shared state-layer utility on interactive items |

---

## Layout variants

| Layout | `layout` prop | Item width strategy | Visible items | Snap behavior | Use case |
|---|---|---|---|---|---|
| Hero | `"hero"` | Active item ≈ 85–90% container width; flanking items peek at ~5–7% each side | 1 focal + partial flanks | `scroll-snap-align: center; scroll-snap-type: x proximity` | Single spotlight item with context |
| Multi-browse | `"multi-browse"` | First item ≈ 60%; subsequent ≈ 30%; last partial | 2–4 | `scroll-snap-align: start; scroll-snap-type: x proximity` | Emphasised lead + browse thumbnails |
| Uncontained | `"uncontained"` | Fixed width (e.g. 240 dp); uniform; no resizing | 2–5+ | `scroll-snap-align: start; scroll-snap-type: x proximity` | Uniform content tiles |
| Full-screen | `"full-screen"` | 100cqw × 100cqh | 1 | `scroll-snap-align: start; scroll-snap-stop: always; scroll-snap-type: x mandatory` | Immersive product/image gallery |

---

## Item resizing behavior

In `hero` and `multi-browse` layouts, off-focal items render smaller. This is purely CSS-driven:

### Strategy A — Scroll-driven animation (progressive enhancement)
```css
@supports (animation-timeline: scroll()) {
  .carousel__item {
    animation: carousel-resize both linear;
    animation-timeline: view(inline);
    animation-range: entry 0% cover 50%;
  }
  @keyframes carousel-resize {
    from { width: var(--item-w-inactive); }
    to   { width: var(--item-w-active); }
  }
}
```
No JS, no `IntersectionObserver`, zero frame-by-frame work. Supported in Chrome 115+.

### Strategy B — IntersectionObserver fallback
For Firefox / Safari: a `useCarousel` hook observes items via `IntersectionObserver` and toggles `.carousel__item--active` / `.carousel__item--inactive` classes. CSS `transition` animates `width` on class change.

```css
.carousel__item { width: var(--item-w-inactive); transition: width var(--duration-medium2) var(--easing-emphasized); }
.carousel__item--active { width: var(--item-w-active); }
```

### CSS custom properties for item sizing (set per layout on `.carousel`)
| Property | Hero | Multi-browse | Uncontained |
|---|---|---|---|
| `--item-w-active` | 85% | 60% | 240px (fixed) |
| `--item-w-inactive` | 35% | 30% | 240px (unchanged) |

These are component-internal vars declared on `.carousel` and can be overridden per consumer.

---

## States

| State | Element | Description |
|---|---|---|
| Default | Item | Neutral surface; no state layer |
| Hovered | Interactive item | `--state-hover` (8%) × `on-surface` state layer |
| Focused | Interactive item | `--state-focus` (10%) state layer + focus ring |
| Pressed | Interactive item | `--state-pressed` (10%) state layer |
| Active (focal) | Item | Full-width rendering; `--active` modifier |
| Inactive (off-focal) | Item | Reduced width; `--inactive` modifier |
| Scrolling | Track | Smooth; native momentum; scrollbar hidden |

---

## Design tokens

All names are unprefixed system roles from `@kafui/styles`. Component-internal vars declared on `.carousel`.

### Color
- Item surface: `--surface-container-low`
- Item label text: `--on-surface`
- Item label background (scrim): `color-mix(in oklab, var(--surface) 80%, transparent)`
- Nav button surface: `--surface-container-high`
- Nav button icon: `--on-surface-variant`
- Indicator active: `--primary`
- Indicator inactive: `--surface-variant`

### Shape
- Item corner: `--corner-medium` (12 dp); full-screen layout: `--corner-none` (0)
- Nav button: `--corner-full` (circular)

### Elevation
- Active hero item: `--elevation-1`
- Inactive item: `--elevation-0`

### Typography
- Item label: `--label-large-size`, `--label-large-weight`, `--label-large-line-height`
- Item sublabel: `--body-small-size`, `--body-small-weight`, `--body-small-line-height`

### Motion
- Scroll-driven resize (Strategy A): `linear` timing; driven by scroll position — no duration token needed.
- Fallback class-change resize (Strategy B): `--easing-emphasized` + `--duration-medium2`
- Focus change: `--easing-standard` + `--duration-short4`
- `@media (prefers-reduced-motion: reduce)`: disable all item size transitions; disable `scroll-behavior: smooth`; scroll still works.

### Scroll snap
| Layout | `scroll-snap-type` | `scroll-snap-align` | `scroll-snap-stop` |
|---|---|---|---|
| Hero | `x proximity` | `center` | (default) |
| Multi-browse | `x proximity` | `start` | (default) |
| Uncontained | `x proximity` | `start` | (default) |
| Full-screen | `x mandatory` | `start` | `always` |

### CSS structure (illustrative)

```css
@layer kafui {
  .carousel {
    /* component-internal vars — overridable per consumer */
    --item-w-active: 85%;
    --item-w-inactive: 35%;
    --item-gap: 8px;

    display: flex;
    overflow-x: scroll;
    scroll-snap-type: x proximity;
    scrollbar-width: none;
    overscroll-behavior-x: contain;
    container-type: inline-size;
    gap: var(--item-gap);
    /* use padding-inline for peek effect in hero layout */
  }
  .carousel::-webkit-scrollbar { display: none; }

  .carousel--hero {
    --item-w-active: 85%;
    --item-w-inactive: 35%;
    padding-inline: 5%;         /* exposes flanking item peek */
  }
  .carousel--multi-browse {
    --item-w-active: 60%;
    --item-w-inactive: 30%;
  }
  .carousel--uncontained {
    --item-w-active: 240px;
    --item-w-inactive: 240px;
  }
  .carousel--full-screen {
    scroll-snap-type: x mandatory;
  }

  .carousel__track {
    display: contents; /* items are direct flex children of .carousel */
  }

  .carousel__item {
    flex: 0 0 auto;
    width: var(--item-w-inactive);
    scroll-snap-align: start;
    border-radius: var(--corner-medium);
    overflow: hidden;
    transition: width var(--duration-medium2) var(--easing-emphasized);
    background: var(--surface-container-low);
  }
  .carousel--hero .carousel__item { scroll-snap-align: center; }
  .carousel--full-screen .carousel__item {
    width: 100cqi; /* container-query inline */
    height: 100cqb;
    scroll-snap-stop: always;
    border-radius: var(--corner-none);
  }

  /* Strategy A: scroll-driven (progressive enhancement) */
  @supports (animation-timeline: scroll()) {
    .carousel--hero .carousel__item,
    .carousel--multi-browse .carousel__item {
      animation: carousel-item-resize linear both;
      animation-timeline: view(inline);
      animation-range: entry 0% cover 40%;
      transition: none; /* let scroll-driven handle it */
    }
    @keyframes carousel-item-resize {
      from { width: var(--item-w-inactive); }
      to   { width: var(--item-w-active); }
    }
  }

  /* Strategy B: class-based fallback */
  .carousel__item--active { width: var(--item-w-active); }
  .carousel__item--inactive { width: var(--item-w-inactive); }

  /* Active hero item gets subtle elevation */
  .carousel--hero .carousel__item--active { box-shadow: var(--elevation-1); }

  .carousel__item-inner {
    overflow: hidden;
    border-radius: inherit;
    height: 100%;
  }
  .carousel__item-media img,
  .carousel__item-media video {
    display: block;
    width: 100%;
    height: 100%;
    object-fit: cover;
  }

  .carousel__item-label {
    position: absolute;
    bottom: 0;
    inset-inline: 0;
    padding-block: 8px;
    padding-inline: 12px;
    background: color-mix(in oklab, var(--surface) 80%, transparent);
    color: var(--on-surface);
  }

  .carousel__prev-button,
  .carousel__next-button {
    position: absolute;
    top: 50%;
    transform: translateY(-50%);
    background: var(--surface-container-high);
    border-radius: var(--corner-full);
    color: var(--on-surface-variant);
  }
  .carousel__prev-button { inset-inline-start: 8px; }
  .carousel__next-button { inset-inline-end: 8px; }

  /* Hide nav buttons on touch/coarse pointer */
  @media (pointer: coarse) {
    .carousel__prev-button,
    .carousel__next-button { display: none; }
  }

  /* RTL: flip directional icons via logical property — icon sprite handles this */
  [dir="rtl"] .carousel__prev-button .icon,
  [dir="rtl"] .carousel__next-button .icon {
    transform: scaleX(-1);
  }

  .carousel__indicators {
    display: flex;
    gap: 4px;
    justify-content: center;
  }
  .carousel__indicator-dot {
    width: 8px;
    height: 8px;
    border-radius: var(--corner-full);
    background: var(--surface-variant);
    transition: background var(--duration-short2) var(--easing-standard);
  }
  .carousel__indicator-dot--active {
    background: var(--primary);
  }

  @media (prefers-reduced-motion: reduce) {
    .carousel__item { transition: none; }
    .carousel__indicator-dot { transition: none; }
    /* scroll-driven animation also halted via user-agent reduced-motion handling */
  }
}
```

---

## Interaction & accessibility

### Scroll / swipe
- Native horizontal scroll (touch swipe, mouse wheel, trackpad).
- `overflow-x: scroll; scrollbar-width: none` hides scrollbar visually; scrolling remains functional.
- `overscroll-behavior-x: contain` prevents page scroll bleed.

### Keyboard

Two semantic models; keyboard behavior differs:

**Listbox model** (RAC `ListBox`):
- Arrow keys move focus between items (roving tabindex managed by RAC).
- `Home`/`End` jump to first/last item.
- `Space` selects focused item; `Enter` activates (navigate or expand).
- Focused item scrolls into view automatically.

**Region model** (custom hook):
- The `.carousel__track` has `tabindex="0"` and handles `ArrowLeft`/`ArrowRight` to scroll to the previous/next snap point via `scrollTo({ behavior: 'smooth' })`.
- `Home`/`End` scroll to first/last item.
- In RTL: `ArrowLeft`/`ArrowRight` directions are logically swapped.

### Focus management
- **Listbox model**: roving tabindex managed by RAC `ListBox`; active item has `tabindex="0"`, others `tabindex="-1"`.
- Focus ring: `outline` on the item using `--outline`, offset 2 dp.
- Nav buttons are standard focusable `<button>` elements; pressing scrolls to the adjacent item and updates the active item.

### ARIA

| Model | Root role | Item role | When to use |
|---|---|---|---|
| Listbox | `role="listbox"` | `role="option"` | Items are selectable (image picker, product chooser); `aria-selected` on active item |
| Region | `role="region"` + `aria-label` | `role="listitem"` inside `role="list"` | Items are browsable content (cards, articles); no selection semantics needed |

- `aria-label` or `aria-labelledby` required on the root in all cases.
- `aria-roledescription="carousel"` on root for additional screen-reader context.
- Nav buttons: `aria-label="Previous"` / `"Next"`; `aria-controls` pointing to track `id`.
- Indicators: `role="tablist"` with `aria-label`; each dot is `role="tab"` + `aria-selected`; clicking scrolls to item.
- `aria-live="polite"` visually-hidden announcer div: announces "Item N of M" on active item change.

### Reduced motion
- Item resizing transitions disabled; items render at fixed size.
- Scroll behavior still works; `scroll-behavior: auto` replaces `smooth` on keyboard scroll.
- Nav button presses scroll without animation.

### RTL
- `scroll-snap-type` and `overflow-x` are direction-neutral.
- `flex-direction: row` reverses under `dir="rtl"` natively.
- `ArrowLeft`/`ArrowRight` keyboard handlers swap direction in RTL (`dir` attribute read from `document.documentElement.dir`).
- Nav button icons flip via `[dir="rtl"] .icon { transform: scaleX(-1) }`.
- `inset-inline-start`/`end` positions nav buttons correctly in both directions.

---

## Proposed kafUI React API

```tsx
// Listbox model: ListBox + ListBoxItem from react-aria-components
// Region model: custom useCarousel hook + native scroll

type CarouselLayout = 'hero' | 'multi-browse' | 'uncontained' | 'full-screen';
type CarouselSemantics = 'listbox' | 'region';

interface CarouselProps {
  layout?: CarouselLayout;                     // default: 'multi-browse'
  semantics?: CarouselSemantics;               // default: 'region'
  selectedKey?: React.Key;                     // controlled; listbox mode
  defaultSelectedKey?: React.Key;
  onSelectionChange?: (key: React.Key) => void;
  showNavButtons?: boolean;                    // default: true (hidden by CSS on coarse-pointer)
  showIndicators?: boolean;                    // default: false
  'aria-label'?: string;
  'aria-labelledby'?: string;
  children: React.ReactNode;                   // Carousel.Item children
  className?: string;
}

interface CarouselItemProps {
  id: string | number;                         // required for roving tabindex / selection
  onPress?: (e: PressEvent) => void;
  href?: string;
  'aria-label'?: string;
  children: React.ReactNode;
  className?: string;
}

interface CarouselItemMediaProps {
  src: string;
  alt: string;                                 // required
  className?: string;
}

interface CarouselItemLabelProps {
  children: React.ReactNode;
  className?: string;
}

// Listbox model — selectable items
<Carousel layout="hero" semantics="listbox" aria-label="Featured products"
  selectedKey={selected} onSelectionChange={setSelected} showNavButtons>
  <Carousel.Item id="1">
    <Carousel.Item.Media src="/product1.jpg" alt="Product 1" />
    <Carousel.Item.Label>Product 1</Carousel.Item.Label>
  </Carousel.Item>
  <Carousel.Item id="2">…</Carousel.Item>
</Carousel>

// Region model — content browsing (default)
<Carousel layout="multi-browse" aria-label="Recent articles">
  <Carousel.Item id="a1">
    <Card>…</Card>
  </Carousel.Item>
</Carousel>
```

**BEM classes emitted:**
- `.carousel` (root)
- `.carousel--hero` / `--multi-browse` / `--uncontained` / `--full-screen`
- `.carousel__track`
- `.carousel__item`
- `.carousel__item--active` / `--inactive` (Strategy B fallback)
- `.carousel__item-inner`, `.carousel__item-media`, `.carousel__item-label`
- `.carousel__prev-button`, `.carousel__next-button`
- `.carousel__indicators`, `.carousel__indicator-dot`, `.carousel__indicator-dot--active`
- `.state-layer` on interactive items; RAC `data-hovered`, `data-focused`, `data-pressed`, `data-selected`

**CSS layer:** all rules in `@layer kafui { … }`. No `kafui-` prefix; the layer name provides collision safety.

**React Aria base:**
- Listbox model: `ListBox` + `ListBoxItem` — handle roving tabindex, ARIA roles, keyboard selection automatically.
- Region model: custom `useCarousel` hook (`IntersectionObserver` + keyboard scroll handler + RTL-aware direction flip).
- Nav buttons: RAC `Button`.
- Indicators: `role="tablist"` with RAC `Button` per dot.

**Deviations / notes:**
- No auto-play: M3 spec does not include it; auto-play is an accessibility anti-pattern.
- `display: contents` on `__track` makes items direct flex children of `.carousel`, avoiding an extra wrapper element and keeping the CSS simpler.
- Scroll-driven animation Strategy A is a progressive enhancement; `@supports` gates it; Strategy B fallback ensures behavior in all browsers.
- `full-screen` uses `100cqi`/`100cqb` (container-query inline/block) when inside a constrained container; the `.carousel` root declares `container-type: inline-size`.
- `semantics` prop is a runtime choice, not a CSS switch — it determines which RAC primitive is used as the root.
- `Carousel.Item.Media` and `Carousel.Item.Label` are nested on the `Item` namespace to signal ownership; implementation is simple functional components.
