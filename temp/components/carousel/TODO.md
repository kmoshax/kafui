# Carousel — TODO

## MUI equivalent

**None in MUI core.** MUI v5/v6 ships no Carousel. Community options (`react-material-ui-carousel`, `nuka-carousel`, `embla-carousel`) require separate installs, add JS-driven scroll simulation, and approximate zero M3 spec fidelity. kafUI ships a carousel that is part of M3 and part of the design system — not an afterthought.

**This is the single biggest gap in MUI's ecosystem.** Every item below is an opportunity to make kafUI the default answer for React M3 carousels.

---

## Beat-MUI opportunities

### 1. Ship all four M3 layouts — no community package needed
MUI's absence means every MUI team reaching for a carousel is gluing together an external package. kafUI ships `hero` / `multi-browse` / `uncontained` / `full-screen` out of the box, all matching the M3 spec's item-sizing, scroll-snap, and surface tokens.

**Tasks:**
- [ ] Create `carousel.css` inside `@layer kafui { … }`
- [ ] Root `.carousel`: `display: flex; overflow-x: scroll; scroll-snap-type: x proximity; scrollbar-width: none; overscroll-behavior-x: contain; container-type: inline-size`
- [ ] Cross-browser scrollbar hide: `.carousel::-webkit-scrollbar { display: none }`
- [ ] `.carousel--hero`: `--item-w-active: 85%; --item-w-inactive: 35%; padding-inline: 5%`; snap `center`
- [ ] `.carousel--multi-browse`: `--item-w-active: 60%; --item-w-inactive: 30%`; snap `start`; first-child emphasis via `:first-child` if desired
- [ ] `.carousel--uncontained`: `--item-w-active: 240px; --item-w-inactive: 240px`; uniform, no resizing
- [ ] `.carousel--full-screen`: `scroll-snap-type: x mandatory`; item `width: 100cqi; height: 100cqb; scroll-snap-stop: always; border-radius: var(--corner-none)`
- [ ] All token names unprefixed: `--corner-medium`, `--corner-none`, `--corner-full`, `--surface-container-low`, `--elevation-0`, `--elevation-1`, etc.

### 2. Zero-JS item resizing with CSS scroll-driven animation — community carousels use `requestAnimationFrame`
Every JS carousel library resizes items in a scroll event handler or `requestAnimationFrame` loop. kafUI uses `animation-timeline: view(inline)` — the browser does the math natively, off the main thread. The `IntersectionObserver` fallback keeps it working in Firefox/Safari.

**Tasks:**
- [ ] `@supports (animation-timeline: scroll())` block: `.carousel--hero .carousel__item, .carousel--multi-browse .carousel__item { animation: carousel-item-resize linear both; animation-timeline: view(inline); animation-range: entry 0% cover 40%; transition: none; }`
- [ ] `@keyframes carousel-item-resize { from { width: var(--item-w-inactive) } to { width: var(--item-w-active) } }`
- [ ] Strategy B (fallback): `.carousel__item { width: var(--item-w-inactive); transition: width var(--duration-medium2) var(--easing-emphasized) }` + `.carousel__item--active { width: var(--item-w-active) }`
- [ ] `useCarousel` hook: `IntersectionObserver` tracking active item; toggles `--active`/`--inactive` classes; detects `CSS.supports('animation-timeline', 'scroll()')` to skip IO when Strategy A available
- [ ] Data attribute `data-scroll-driven` on root when Strategy A is active (lets CSS select the right branch)
- [ ] `@media (prefers-reduced-motion: reduce)`: `transition: none` on `.carousel__item`; `scroll-behavior: auto` on keyboard scroll

### 3. Native-quality keyboard navigation with roving tabindex — most carousels break keyboard
Community carousels typically have no keyboard support or bolt on basic arrow-key handlers that don't manage focus. kafUI uses RAC `ListBox` (listbox model) for correct roving tabindex, `aria-selected`, and keyboard semantics out of the box, or a `useCarousel` hook for the region model.

**Tasks:**
- [ ] `Carousel` root: listbox model uses RAC `ListBox`; region model uses `<section>`/`<div role="region">` + `useCarousel` hook
- [ ] `useCarousel` hook: keyboard handler on track for `ArrowLeft`/`ArrowRight` + `Home`/`End`; RTL-aware (`document.documentElement.dir`); `scrollTo({ behavior: 'smooth' })` on normal motion; `scrollTo({ behavior: 'auto' })` under reduced-motion
- [ ] `Carousel.Item` (listbox): RAC `ListBoxItem`; roving tabindex managed by RAC
- [ ] `Carousel.Item` (region): `<li>` with manual roving tabindex via hook
- [ ] Focus ring: `outline` with `--outline` color, 2 dp offset
- [ ] Navigation buttons: RAC `Button`; `aria-controls` + `aria-label="Previous"/"Next"`; hidden by CSS on `pointer: coarse`

### 4. Correct ARIA semantics — community carousels fail WCAG 2.1 regularly
`aria-roledescription="carousel"`, `aria-live="polite"` item announcer, proper `listbox`/`region` choice, `aria-controls` on nav buttons — none of these exist in typical community carousels.

**Tasks:**
- [ ] Root: `aria-roledescription="carousel"` always; plus `role="listbox"` or `role="region"` per `semantics` prop
- [ ] `aria-label` or `aria-labelledby` required (TypeScript enforces one is present; runtime dev warning if both absent)
- [ ] Visually-hidden live region: `aria-live="polite"` div announces "Item N of M" on active item change
- [ ] Indicator dots: `role="tablist"` + `aria-label`; each dot `role="tab"` + `aria-selected`; clicking scrolls to item
- [ ] Nav buttons: `aria-controls={trackId}` wired via `useId`

### 5. RTL and dark mode zero-config — community carousels break in RTL
Logical CSS properties + `[dir="rtl"]` icon flip covers RTL. `light-dark()` tokens cover dark mode. Neither requires any consumer code.

**Tasks:**
- [ ] `inset-inline-start`/`end` for nav button positioning
- [ ] RTL keyboard handler: `ArrowLeft` → forward, `ArrowRight` → back when `dir="rtl"`
- [ ] `[dir="rtl"] .carousel__prev-button .icon, [dir="rtl"] .carousel__next-button .icon { transform: scaleX(-1) }`
- [ ] Dark mode: verify `--surface-container-low`, `--surface-container-high`, `--primary`, `--surface-variant` all flip under `color-scheme: dark`
- [ ] Write RTL visual test and dark mode visual test

### 6. Container-query sizing — works inside any layout, not just full-viewport
Full-screen layout uses `100cqi`/`100cqb` so it fills its _container_, not the viewport. This means the carousel works inside a dialog, a panel, or a sidebar without modification.

**Tasks:**
- [ ] `.carousel { container-type: inline-size; }` on root
- [ ] `.carousel--full-screen .carousel__item { width: 100cqi; height: 100cqb; }`
- [ ] Test: full-screen carousel inside a 600px-wide dialog — should fill the dialog, not the viewport

---

## Styles (`@kafui/styles`) summary checklist
- [ ] `carousel.css` — all rules in `@layer kafui { … }`
- [ ] Component-internal vars on `.carousel`: `--item-w-active`, `--item-w-inactive`, `--item-gap`
- [ ] All system token names unprefixed: `--corner-medium`, `--corner-full`, `--corner-none`, `--surface-container-low`, `--surface-container-high`, `--surface-variant`, `--on-surface`, `--on-surface-variant`, `--primary`, `--outline`, `--elevation-0`, `--elevation-1`, `--easing-emphasized`, `--duration-medium2`, `--duration-short2`, `--state-hover`, `--state-focus`, `--state-pressed`
- [ ] No `kafui-` prefix on any BEM class
- [ ] `@layer kafui` provides collision safety

## React (`@kafui/react`) summary checklist
- [ ] `Carousel` compound: `Carousel.Item`, `Carousel.Item.Media`, `Carousel.Item.Label` all exported
- [ ] `useCarousel` hook: IO-based active tracking, keyboard scroll, RTL-aware, Strategy A detection
- [ ] Both listbox and region semantic models supported via `semantics` prop
- [ ] `aria-label` / `aria-labelledby` TypeScript enforcement (one required)
- [ ] Live region announcer in DOM
- [ ] Nav buttons hidden by CSS on coarse-pointer; always in DOM for keyboard users on mouse

## Testing
- [ ] Unit: active item class toggled correctly by `IntersectionObserver` callback (Strategy B)
- [ ] Unit: `ArrowLeft`/`Right` keyboard moves focus and scrolls
- [ ] Unit: `Home`/`End` jump to first/last item
- [ ] Unit: RTL — `ArrowLeft`/`Right` directions swap
- [ ] A11y: listbox mode — `role="listbox"` + `role="option"` present; `aria-selected` updates
- [ ] A11y: region mode — `role="region"` + `role="list"` + `role="listitem"` present
- [ ] A11y: nav buttons have `aria-controls` pointing to track id
- [ ] A11y: live region announces "Item N of M" on change
- [ ] A11y: focus ring visible on keyboard navigation
- [ ] Visual: hero layout — active item wider than inactive
- [ ] Visual: full-screen layout — fills container (not viewport)
- [ ] Visual: dark mode — all surface tokens correct
- [ ] Visual: RTL — nav buttons on correct sides; icons flipped
- [ ] Reduced motion: no resize animation; scroll still functional
- [ ] Performance: no scroll event handler in Strategy A path (verify via DevTools)
