# Card

A contained surface that groups related content and actions about a single subject. M3 category: **Containment → Cards**. Cards may be static (display-only) or interactive (clickable as a whole, or with individual action controls). They never scroll internally.

---

## Anatomy / parts

| BEM element | Description |
|---|---|
| `.card` | Root container; carries variant surface, shape, elevation, state layer |
| `.card__media` | Optional leading image or video region; typically full-width at the top |
| `.card__header` | Optional header zone containing avatar, headline, subhead, trailing action icon |
| `.card__avatar` | Leading avatar/icon inside header |
| `.card__headline` | Primary title text; maps to `title-large` typescale |
| `.card__subhead` | Secondary descriptor; maps to `body-medium` in `on-surface-variant` |
| `.card__action-icon` | Trailing icon button in header (e.g. more-vert) |
| `.card__content` | Main body text area; maps to `body-medium` |
| `.card__actions` | Bottom bar for action buttons / icon buttons; `display: flex; gap` |
| `.state-layer` | Full-cover absolutely-positioned overlay for hover/focus/pressed tints (interactive cards only); shared utility class |

All sub-elements are optional. The minimum valid card is just `.card` with any direct children.

---

## Variants

| Variant | `variant` prop | Surface token | Elevation | Border |
|---|---|---|---|---|
| Elevated | `"elevated"` | `--surface-container-low` | level 1 resting → level 2 hover | none |
| Filled | `"filled"` | `--surface-container-highest` | level 0 | none |
| Outlined | `"outlined"` | `--surface` | level 0 | 1 dp `--outline-variant` |

Default variant: `"elevated"`.

---

## States

Applies only when the card is **interactive** (has `onPress`, `href`, `isSelected`, or `isDraggable`).

| State | State-layer opacity token | Elevation delta |
|---|---|---|
| Enabled | 0% | Resting elevation |
| Hovered | `--state-hover` (0.08) × `on-surface` | +1 level (elevated: level 1 → 2) |
| Focused | `--state-focus` (0.10) × `on-surface` | Resting elevation |
| Pressed | `--state-pressed` (0.10) × `on-surface` | Resting elevation |
| Dragged | `--state-dragged` (0.16) × `on-surface` | `--elevation-4` (all variants) |
| Disabled | surface at 12% `on-surface` overlay; content at 38% `on-surface` | `--elevation-0` |

Static (non-interactive) cards have no state layer and no hover/focus/pressed behavior.

---

## Interactive sub-modes

| Sub-mode | Description | RAC primitive |
|---|---|---|
| Clickable card | Entire card surface is one press target | RAC `Button` (no `href`) or RAC `Link` (with `href`) as root |
| Selectable card | Card acts as a checkbox/radio; `aria-checked`, `role="checkbox"` or `role="option"` | `Checkbox` or `ListBoxItem` depending on context |
| Draggable card | Card can be reordered; `role="listitem"` in a drag-and-drop list; dragged state layer | RAC `useDraggableItem` |

These sub-modes are mutually exclusive. Consumers select via props; the root element type is inferred automatically.

---

## Design tokens

All token names reference the unprefixed system roles from `@kafui/styles`. Component-internal variables are declared inside the `.card` block.

### Color
| Role | Token |
|---|---|
| Elevated surface | `--surface-container-low` |
| Filled surface | `--surface-container-highest` |
| Outlined surface | `--surface` |
| Border (outlined) | `--outline-variant` |
| State layer color | `--on-surface` (at `--state-hover/focus/pressed/dragged` opacity) |
| Headline text | `--on-surface` |
| Subhead / body | `--on-surface-variant` |
| Disabled surface overlay | `--on-surface` at 12% |
| Disabled text | `--on-surface` at 38% |

### Shape
- Corner: `--corner-medium` (12 dp) for all variants at all interactive states.
- Applied via logical `border-start-start-radius` etc. for RTL correctness.
- `__media` clips to top corners only when at the top of the card.

### Elevation
| Slot | Token |
|---|---|
| Elevated resting | `--elevation-1` |
| Elevated hover | `--elevation-2` |
| Filled/Outlined resting | `--elevation-0` |
| Dragged (all variants) | `--elevation-4` |

Implemented via `box-shadow` mapped to each level's M3 shadow spec.

### Typography
- Headline: `--title-large-size`, `--title-large-weight`, `--title-large-line-height`
- Subhead: `--body-medium-size`, `--body-medium-weight`, `--body-medium-line-height`
- Body / supporting text: `--body-medium-size`, `--body-medium-weight`, `--body-medium-line-height`
- Action button labels: `--label-large-size` etc. (handled by Button component)

### State layer
- `--state-hover`: 0.08
- `--state-focus`: 0.10
- `--state-pressed`: 0.10
- `--state-dragged`: 0.16

### Motion
- Elevation transition on hover: `--easing-standard` + `--duration-short4`
- State layer fade: `--duration-short2`

### CSS structure (illustrative)

```css
@layer kafui {
  .card {
    /* component-internal vars */
    --radius: var(--corner-medium);
    --pad-block: 12px;
    --pad-inline: 16px;

    position: relative;
    border-start-start-radius: var(--radius);
    border-start-end-radius: var(--radius);
    border-end-start-radius: var(--radius);
    border-end-end-radius: var(--radius);
    overflow: hidden;
    transition:
      box-shadow var(--duration-short4) var(--easing-standard);
  }

  .card--elevated {
    background: var(--surface-container-low);
    box-shadow: var(--elevation-1);
  }
  .card--filled {
    background: var(--surface-container-highest);
    box-shadow: var(--elevation-0);
  }
  .card--outlined {
    background: var(--surface);
    border: 1px solid var(--outline-variant);
    box-shadow: var(--elevation-0);
  }

  /* state layer lives on the shared .state-layer utility class */
  .card .state-layer {
    color: var(--on-surface);
  }
  .card[data-hovered] .state-layer { opacity: var(--state-hover); }
  .card[data-focus-visible] .state-layer { opacity: var(--state-focus); }
  .card[data-pressed] .state-layer { opacity: var(--state-pressed); }
  .card--dragging .state-layer { opacity: var(--state-dragged); }

  .card[data-hovered].card--elevated { box-shadow: var(--elevation-2); }
  .card--dragging { box-shadow: var(--elevation-4); }

  .card__media {
    overflow: hidden;
    /* top corners only when positioned at top; clips image to card shape */
    border-start-start-radius: var(--radius);
    border-start-end-radius: var(--radius);
  }
  .card__media img,
  .card__media video {
    display: block;
    width: 100%;
    aspect-ratio: 16 / 9;
    object-fit: cover;
  }

  .card__header {
    display: grid;
    grid-template-columns: auto 1fr auto;
    align-items: center;
    gap: 8px;
    padding-inline: var(--pad-inline);
    padding-block: var(--pad-block);
  }

  .card__actions {
    display: flex;
    flex-wrap: wrap;
    gap: 8px;
    padding-inline: var(--pad-inline);
    padding-block: var(--pad-block);
  }

  @media (prefers-reduced-motion: reduce) {
    .card { transition: none; }
    .card .state-layer { transition: none; }
  }
}
```

---

## Interaction & accessibility

### Keyboard
- Clickable card: `Enter` or `Space` activates (handled by RAC `Button`/`Link`).
- Selectable card: `Space` toggles selection; `Enter` navigates (link variant).
- Draggable card: per WAI-ARIA drag-and-drop pattern (`Space` to grab, arrows to reorder, `Enter`/`Escape` to drop/cancel).

### Focus
- Interactive card: focus ring on root element via `data-focus-visible`; ring uses `--outline` color, offset 3 dp.
- Individual action buttons inside `__actions` remain independently focusable regardless of card clickability.
- When the card is clickable as a whole **and** contains action buttons, action buttons call `e.continuePropagation()` — or better, they are placed outside the card's `Button`/`Link` root so there is no nesting of interactive elements.

### Nested interactives — important constraint
M3 allows cards with both a clickable surface and internal action buttons. The safest implementation:
- The `Card` root is a non-interactive `<div>` with a full-surface `<Link>`/`<Button>` overlay (`position: absolute; inset: 0`) behind the content (`z-index: 0`).
- Action buttons in `__actions` sit above the overlay (`z-index: 1; position: relative`) and are independently focusable/pressable.
- This avoids invalid `<button>` inside `<button>` nesting and is the pattern used by Google's own M3 web component.

### ARIA
- Static card: no required role; use `role="article"` or `role="region"` if contextually appropriate.
- Clickable card: root overlay is `role="button"` or `role="link"`; `aria-label` required when card has no descriptive heading.
- Selectable card: `role="checkbox"` + `aria-checked`; or `role="option"` inside `role="listbox"`.
- Draggable card: `role="listitem"` inside `role="list"`; describe drag affordance via `aria-describedby`.
- `<img>` in media: must have `alt`; decorative images use `alt=""`.
- `__headline` renders as a heading element at the appropriate level; level is configurable, not hardcoded.

### Touch target
- Action buttons within `__actions`: minimum 48×48 dp touch targets.
- Clickable card: entire card surface is the touch target; no minimum card size enforced.

### Reduced motion
- Elevation transitions and state-layer fades suppressed via `@media (prefers-reduced-motion: reduce)`.

### RTL
- `__actions`: `gap` + logical `margin-inline-start` for leading avatar.
- `__header`: `grid-template-columns` direction is direction-neutral; text aligns correctly under `dir="rtl"`.
- `padding-inline`, `padding-block` throughout.

---

## Proposed kafUI React API

```tsx
// Static card: plain <div>
// Clickable card: overlay <Button>/<Link> inside a <div> root (see nested-interactives note)
// Selectable card: root renders as appropriate RAC primitive

type CardVariant = 'elevated' | 'filled' | 'outlined';

interface CardProps {
  variant?: CardVariant;            // default: 'elevated'
  onPress?: (e: PressEvent) => void; // makes card interactive; renders overlay Button
  href?: string;                    // renders overlay Link; implies interactive
  isSelected?: boolean;             // selectable mode; root emits aria-checked
  onSelectionChange?: (isSelected: boolean) => void;
  isDraggable?: boolean;            // enables drag-and-drop integration
  isDragging?: boolean;             // controlled dragging state (elevated state)
  isDisabled?: boolean;
  'aria-label'?: string;
  children: React.ReactNode;
  className?: string;
}

interface CardMediaProps {
  src: string;
  alt: string;                      // required; enforced by TypeScript
  aspectRatio?: string;             // CSS aspect-ratio value; default: '16/9'
  className?: string;
}

interface CardHeaderProps {
  avatar?: React.ReactNode;         // leading avatar/icon
  headline: React.ReactNode;        // required; primary title; rendered as heading element
  subhead?: React.ReactNode;
  headingLevel?: 2 | 3 | 4 | 5 | 6; // default: 3; must be overridable for document outline
  action?: React.ReactNode;         // trailing icon button slot
  className?: string;
}

interface CardContentProps {
  children: React.ReactNode;
  className?: string;
}

interface CardActionsProps {
  children: React.ReactNode;        // Button / IconButton children
  className?: string;
}

// Compound usage
<Card variant="outlined" onPress={handlePress} aria-label="Mountain landscape article">
  <Card.Media src="/hero.jpg" alt="Mountain landscape" />
  <Card.Header
    headline="Headline"
    subhead="Subhead"
    headingLevel={3}
    action={<IconButton name="more-vert" aria-label="More options" />}
  />
  <Card.Content>Supporting text for the card content.</Card.Content>
  <Card.Actions>
    <Button variant="filled">Action</Button>
    <Button variant="text">Action</Button>
  </Card.Actions>
</Card>
```

**BEM classes emitted:**
- `.card` (always)
- `.card--elevated` / `.card--filled` / `.card--outlined`
- `.card--interactive` when `onPress`, `href`, or selection/drag props present
- `.card--disabled`
- `.card--dragging`
- `.state-layer` (shared utility; rendered inside interactive card root)
- `.card__media`, `.card__header`, `.card__avatar`, `.card__headline`, `.card__subhead`, `.card__action-icon`, `.card__content`, `.card__actions`
- RAC data attributes on interactive overlay: `data-hovered`, `data-focused`, `data-pressed`

**CSS layer:** all rules live inside `@layer kafui { … }`. Class names are collision-safe through layer specificity alone — no `kafui-` prefix on BEM classes.

**React Aria base:**
- Clickable card with `href`: full-surface `Link` overlay from `react-aria-components`.
- Clickable card without `href`: full-surface `Button` overlay from `react-aria-components`.
- Static card: plain `<div>`.
- `Card.Media`, `Card.Header`, `Card.Content`, `Card.Actions`: simple functional components; no RAC primitives needed.

**Deviations / notes:**
- No `color` prop — variant encodes all color roles per M3.
- Overlay pattern for nested interactives avoids invalid HTML nesting and is more accessible than `stopPropagation` hacks.
- `Card.Header` exposes `headingLevel`; default `h3` is a reasonable assumption but must be overridable.
- `isInteractive` flag is removed: interactivity is inferred from `onPress`/`href`/`isSelected`/`isDraggable` presence. Explicit flag was redundant.
