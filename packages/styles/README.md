# @kafui/styles

CSS-first Material 3 styling for kafUI.

- **One source color → full palette.** Set `--md-source` and the entire M3 tonal
  system is derived at runtime with OKLCH relative-color syntax. No build step.
- **Light/dark built in** via `light-dark()` + `color-scheme`.
- **RTL built in** via CSS logical properties.
- **BEM component classes** (`.kafui-button`, `.kafui-button--filled`,
  `.kafui-button__icon`), organized with cascade layers so consumer overrides
  win without `!important`.

## Usage
```ts
import "@kafui/styles";          // reset + tokens + components
```
```ts
import "@kafui/styles/tokens";   // tokens only
```

## Theming
```css
:root        { --md-source: #6750a4; }       /* re-derive everything */
.brand-theme { --md-source: oklch(0.55 0.18 145); }
.pinned      { --md-sys-color-primary: #0b57d0; }  /* override one role */
```

See `temp/components/_TOKENS.md` for the full token architecture.
