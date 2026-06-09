# kafUI Token System (`@kafui/styles`)

How the M3 design-token system maps onto a CSS-first, single-source-color
pipeline. Every component `SPEC.md` references roles by their **plain, unprefixed
name** (`primary`, `on-surface`, `outline`…); this file is the contract for how
those roles become real CSS custom properties.

## 0. Principles
- **No prefix soup.** Tokens are bare M3 role names: `--primary`, `--on-primary`,
  `--surface`, `--outline`. No `--md-sys-color-*`, no `--kafui-*`. The brand name
  lives **only** on the `@layer` name (§9), never on a token or a class.
- **One source color in, full M3 palette out.** Authors set `--source` (any valid
  color). The whole tonal system derives from it at runtime via OKLCH
  relative-color syntax — no build step, no JS, no Sass.
- **`light-dark()` for scheme.** Each role is defined once as
  `light-dark(<light>, <dark>)`; flipping `color-scheme` switches the whole UI.
- **Everything overridable.** Any generated role can be hard-overridden by simply
  redeclaring its custom property at any scope (`:root`, a theme class, a subtree).
- **Logical, not physical.** Spacing/border tokens use logical directions so RTL
  is automatic.

## 1. Reference palette — OKLCH from one source
M3 builds tonal palettes (tones 0–100) per key color (primary, secondary,
tertiary, neutral, neutral-variant, error). We approximate this with OKLCH:
lightness `L` is the tone axis, chroma `C` and hue `H` derive from the source.
The raw ramp is named `--ref-<key>-<tone>` (reference palette — internal plumbing,
not consumed directly by components).

```css
@layer kafui.tokens {
  :root {
    /* The single input. Everything below derives from it. */
    --source: #6750a4;                 /* default M3 baseline seed */

    /* Key hues, rotated off the source (M3 secondary≈source, tertiary +60°). */
    --hue-primary:  oklch(from var(--source) l c h);
    --hue-tertiary: oklch(from var(--source) l c calc(h + 60));

    /* Tonal stops: fix chroma/hue from source, sweep L for each tone. */
    --ref-primary-40: oklch(from var(--source) 0.40 c h);
    --ref-primary-80: oklch(from var(--source) 0.80 c h);
    --ref-primary-90: oklch(from var(--source) 0.90 c h);
    --ref-primary-10: oklch(from var(--source) 0.10 c h);
    /* …neutral tones reduce chroma toward ~0.005 for surfaces/backgrounds… */
    --ref-neutral-98: oklch(from var(--source) 0.98 0.005 h);
    --ref-neutral-6:  oklch(from var(--source) 0.06 0.005 h);
  }
}
```

> Note: `oklch(from … )` relative syntax exposes the channels `l c h`; the snippet
> shows intent. The real stylesheet enumerates the full M3 tone ladder
> (0,10,20,30,40,50,60,70,80,90,95,98,99,100) per key color. This is generated
> mechanically — no component `SPEC.md` ever hardcodes a hex.

## 2. System color roles — `light-dark()`, unprefixed
M3 maps tones to roles differently per scheme (e.g. `primary` = tone 40 in light,
tone 80 in dark). `light-dark()` encodes both at once. Component specs reference
these names verbatim (`var(--primary)`, `var(--on-surface-variant)`).

```css
@layer kafui.tokens {
  :root { color-scheme: light dark; }

  :root {
    --primary:               light-dark(var(--ref-primary-40), var(--ref-primary-80));
    --on-primary:            light-dark(#fff,                   var(--ref-primary-20));
    --primary-container:     light-dark(var(--ref-primary-90),  var(--ref-primary-30));
    --on-primary-container:  light-dark(var(--ref-primary-10),  var(--ref-primary-90));
    --inverse-primary:       light-dark(var(--ref-primary-80),  var(--ref-primary-40));

    /* secondary, tertiary families follow the same pattern */
    --secondary-container:    light-dark(var(--ref-secondary-90), var(--ref-secondary-30));
    --on-secondary-container: light-dark(var(--ref-secondary-10), var(--ref-secondary-90));

    --surface:               light-dark(var(--ref-neutral-98), var(--ref-neutral-6));
    --on-surface:            light-dark(var(--ref-neutral-10), var(--ref-neutral-90));
    --surface-variant:       light-dark(var(--ref-nv-90),      var(--ref-nv-30));
    --on-surface-variant:    light-dark(var(--ref-nv-30),      var(--ref-nv-80));
    --outline:               light-dark(var(--ref-nv-50),      var(--ref-nv-60));
    --outline-variant:       light-dark(var(--ref-nv-80),      var(--ref-nv-30));

    /* surface-container tiers (M3 2024 surface model — see §6) */
    --surface-container-lowest:  light-dark(#fff,                  var(--ref-neutral-4));
    --surface-container-low:     light-dark(var(--ref-neutral-96), var(--ref-neutral-10));
    --surface-container:         light-dark(var(--ref-neutral-94), var(--ref-neutral-12));
    --surface-container-high:    light-dark(var(--ref-neutral-92), var(--ref-neutral-17));
    --surface-container-highest: light-dark(var(--ref-neutral-90), var(--ref-neutral-22));

    --error:                 light-dark(var(--ref-error-40),   var(--ref-error-80));
    --on-error:              light-dark(#fff,                  var(--ref-error-20));
    --inverse-surface:       light-dark(var(--ref-neutral-20), var(--ref-neutral-90));
    --inverse-on-surface:    light-dark(var(--ref-neutral-95), var(--ref-neutral-20));
    --scrim:                 #000;       /* tinted at use site, see §10 */
    --shadow:                #000;
  }
}
```

**Override example** (a whole tenant theme = one line):
```css
.theme-brand { --source: oklch(0.55 0.18 145); }   /* re-derives everything */
.theme-fixed { --primary: #0b57d0; }                /* pin a single role */
```

## 3. State layers — `--state-*` opacities + `.state-layer`
M3 overlays the relevant `on-*` color at a fixed opacity.

```css
@layer kafui.tokens {
  :root {
    --state-hover:   0.08;
    --state-focus:   0.10;
    --state-pressed: 0.10;   /* selection controls also 0.10 */
    --state-dragged: 0.16;
  }
}
```

**Implementation convention (uniform across the library):**
- **Default = pseudo-element.** Interactive blocks render the state tint via a
  `::before` (or `::after`) pseudo, no extra DOM node:
  ```css
  @layer kafui.components {
    .button::before {
      content: ""; position: absolute; inset: 0; border-radius: inherit;
      background: currentColor; opacity: 0; transition: opacity var(--duration-short2);
      pointer-events: none;
    }
    .button[data-hovered]::before { opacity: var(--state-hover); }
    .button[data-pressed]::before { opacity: var(--state-pressed); }
  }
  ```
- **Real `.state-layer` child only when** the block contains independently
  focusable descendants that must stack above the tint (e.g. a list row with a
  trailing `<Switch>`/`<Checkbox>`). Those components document the exception in
  their SPEC.

React Aria's `data-hovered` / `data-pressed` / `data-focus-visible` attributes
drive the opacity — never JS className toggling.

## 4. Shape — `--corner-*` (logical)
```css
@layer kafui.tokens {
  :root {
    --corner-none: 0;
    --corner-extra-small: 4px;
    --corner-small: 8px;
    --corner-medium: 12px;
    --corner-large: 16px;
    --corner-extra-large: 28px;
    --corner-full: 9999px;
  }
}
```
Use `border-start-start-radius` etc. (logical) so shape mirrors in RTL. The
Expressive shape-morph-on-press is a transition between two corner tokens; for
components that rest at `--corner-full` (pill), the morph **dips toward
`--corner-large`** on press and returns, since morphing full→full is a no-op.

## 5. Typography — `--<role>-<prop>`
Roles `display/headline/title/body/label` × `large/medium/small`, each a bundle of
`font / weight / size / line-height / tracking`, e.g. `--label-large-size`,
`--body-medium-weight`. Applied with a single utility class (e.g.
`.typescale-label-large`) or by referencing the individual props.

## 6. Elevation & motion
**Elevation** uses the M3 2024 tonal surface model: tonal "lift" comes from the
`--surface-container-*` tier (not a separate dark-mode tint overlay), and
`--elevation-N` supplies the matching `box-shadow` pair only.
```css
@layer kafui.tokens {
  :root {
    --elevation-0: none;
    --elevation-1: 0 1px 2px rgb(0 0 0 / .30), 0 1px 3px 1px rgb(0 0 0 / .15);
    --elevation-2: 0 1px 2px rgb(0 0 0 / .30), 0 2px 6px 2px rgb(0 0 0 / .15);
    --elevation-3: 0 4px 8px 3px rgb(0 0 0 / .15), 0 1px 3px rgb(0 0 0 / .30);
    --elevation-4: 0 6px 10px 4px rgb(0 0 0 / .15), 0 2px 3px rgb(0 0 0 / .30);
    --elevation-5: 0 8px 12px 6px rgb(0 0 0 / .15), 0 4px 4px rgb(0 0 0 / .30);
  }
}
```
**Motion** — explicit ladder (components reference these names):
```css
@layer kafui.tokens {
  :root {
    --easing-standard:               cubic-bezier(0.2, 0, 0, 1);
    --easing-emphasized:             cubic-bezier(0.2, 0, 0, 1);
    --easing-emphasized-decelerate:  cubic-bezier(0.05, 0.7, 0.1, 1);
    --easing-emphasized-accelerate:  cubic-bezier(0.3, 0, 0.8, 0.15);
    --duration-short2: 100ms;
    --duration-short3: 150ms;
    --duration-short4: 200ms;
    --duration-medium1: 250ms;
    --duration-medium2: 300ms;
    --duration-long2:  500ms;
  }
}
```
All animations are wrapped in `@media (prefers-reduced-motion: no-preference)`
(opt-in) so reduced-motion users get the static result by default.

## 7. Z-index tiers — `--z-*`
Stacking order is coordinated through tokens, never hardcoded per component.
```css
@layer kafui.tokens {
  :root {
    --z-nav: 100;
    --z-app-bar: 200;
    --z-drawer: 300;
    --z-modal-scrim: 400;
    --z-modal: 410;       /* dialog, modal bottom/side sheet */
    --z-snackbar: 500;
    --z-tooltip: 600;
  }
}
```

## 8. BEM convention — unprefixed
- Block: `.<component>` (e.g. `.button`, `.text-field`, `.chip`)
- Element: `.<component>__<part>` (e.g. `.button__icon`, `.dialog__actions`)
- Modifier: `.<component>--<variant|state>` (e.g. `.button--filled`)
- Shared utilities are bare too: `.state-layer`, `.scrim`.
- React Aria render props supply `data-*` state hooks; CSS targets those for
  hover/press/focus/selected rather than extra modifier classes wherever RAC
  exposes an attribute. Reserve modifiers for things RAC has no attribute for
  (e.g. `.button--icon-only`). Density is expressed via a `data-density`
  attribute (`data-density="-2"`), never a double-dashed `--density--2` modifier.

## 9. `@layer` order (collision safety)
The brand name appears **only** here. Generic class/token names are safe because
layered styles always lose specificity battles to unlayered author styles.
```css
@layer kafui.reset, kafui.tokens, kafui.base, kafui.components, kafui.utilities;
```
The entry stylesheet establishes this order first; `_scrim.css` and other shared
partials (in `kafui.base`) are imported before any component sheet that uses them.

## 10. RTL, dark mode & scrim (built in, zero config)
- **Dark:** consumer sets `color-scheme: dark` (or `light dark` + OS pref); every
  role flips via `light-dark()`. No `.dark` class duplication, no elevation tint
  overlay (the surface-container tiers carry tonal elevation).
- **RTL:** all directional CSS uses logical properties
  (`margin-inline-start`, `inset-inline-end`, `border-start-start-radius`,
  `padding-block`); `dir="rtl"` mirrors layout automatically. Off-canvas surfaces
  use the `translate` property (`translate: 100% 0`) which is writing-mode aware.
  Icons that encode direction (back/forward) flip via `:dir(rtl)` selectors.
- **Scrim:** shared `.scrim` utility, tinted at the use site with
  `background: color-mix(in srgb, var(--scrim) 32%, transparent)` (works with
  OKLCH-derived tokens — no `-rgb` split-channel variables).

## 11. Browser baseline
kafUI targets evergreen browsers that support OKLCH relative color, `light-dark()`,
`@layer`, and logical properties (≈ Chrome 120+, Firefox 121+, Safari 17.4+).
Progressive enhancements degrade gracefully:
- `:has()`-based sibling effects fall back to a React-set `data-*` attribute.
- Scroll-driven effects (`animation-timeline: view()`) fall back to a JS sentinel
  (`useScrollSentinel`) where the visual is load-bearing.
</content>
