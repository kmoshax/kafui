# @kafui/react

React Material 3 components for kafUI.

- Built on **React Aria** (`react-aria-components`) for behavior + accessibility.
- **Zero styling in JSX** — components render BEM classNames; all visuals come
  from `@kafui/styles`.
- **Sprite-backed `<Icon name>`** — no per-icon JS modules.
- **i18n & RTL ready** — React Aria `I18nProvider`, `@internationalized/date`,
  logical props; `onPress` (not `onClick`) everywhere.

## Usage
```tsx
import "@kafui/styles";
import { Button } from "@kafui/react";

<Button variant="filled" onPress={save}>Save</Button>;
```

> Status: scaffolded. Component implementations land per the specs in
> `temp/components/<slug>/SPEC.md`.
