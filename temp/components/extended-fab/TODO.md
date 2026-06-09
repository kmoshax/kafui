# Extended FAB — TODO

## MUI equivalent
`@mui/material/Fab` with text content as `children` (no dedicated Extended FAB component). Closest approximation: `<Fab variant="extended">icon text</Fab>`.

MUI Fab (extended) key reference: text + icon passed as `children` ReactNode, no `collapsed` prop, no dedicated scroll-collapse API, no `lowered` prop, `variant="extended"` switches layout but the same component handles both FAB and Extended FAB, Emotion CSS-in-JS.

---

## Beat-MUI opportunities

### 1. Dedicated `<ExtendedFab>` component — not a `variant` switch on a shared component
**What we do:** `<ExtendedFab>` is a standalone component, not `<Fab variant="extended">`. It has its own props, its own CSS class, and its own documentation.

**Why it wins:** MUI's `Fab` with `variant="extended"` is a structural/behavioral fork hidden behind a string prop — it changes the component's anatomy (adds a text node), sizing, and padding rules, but these are all baked into a single component definition. This means `Fab` must carry extended-FAB-specific logic even when rendering as circular. Our separation produces a smaller, more focused JS module for each; tree-shaking removes whichever you don't import.

**Task:** Create `packages/react/src/components/extended-fab/ExtendedFab.tsx` as a standalone component. Do not share a single RAC wrapper between `Fab` and `ExtendedFab`. Verify bundle size: importing only `Fab` should not include ExtendedFab CSS.

### 2. `label: string` as a compile-enforced accessible name
**What we do:** `label` is a required `string` prop. The component always sets `aria-label={label}` on the root button, whether collapsed or extended. The `label` value also renders as the visible `.extended-fab__label` text.

**Why it wins:** MUI passes `children` as ReactNode — no accessible name is ever automatically derived. If a consumer passes `children={<span>Create</span>}` and later adds `collapsed` behavior, the accessible name disappears. MUI's approach makes this a silent runtime failure. Ours makes it impossible — the string is always available for AT even when the label is visually hidden.

**Task:** Implement `aria-label={props.label}` unconditionally on the root button. Write a Jest/RTL test that renders the component in `collapsed` state and asserts `getByRole('button', { name: 'Create' })` still finds it.

### 3. Built-in CSS collapse animation — zero JS for animation
**What we do:** `.extended-fab--collapsed` modifier class triggers a CSS-only collapse: `inline-size` narrows to 56 px, `.extended-fab__label` fades to `opacity: 0` and `inline-size: 0`. Transitions use `--easing-emphasized` and `--duration-medium2`. Consumer only toggles a boolean prop.

**Why it wins:** MUI has no collapse concept at all — every codebase using MUI's Fab for an Extended FAB reinvents the collapse animation with inline styles, `styled()` overrides, or a custom motion library. These implementations are inconsistent, rarely respect `prefers-reduced-motion`, and couple scroll logic to animation logic. Our approach decouples all three: scroll detection (hook), state (controlled prop), and animation (CSS).

**Task:** Implement the CSS collapse/extend rules. Stagger the label opacity fade to start slightly earlier than width completion (label fades in the first 100 ms, width continues to 300 ms). Add a Storybook interactive story that demonstrates collapse via a "Simulate Scroll" button.

### 4. `useExtendedFabCollapse(threshold?)` hook — batteries included
**What we do:** a `useExtendedFabCollapse` hook in `@kafui/hooks` returns `{ collapsed: boolean }`. Internally it uses `IntersectionObserver` or a lightweight `scroll` event listener with the given threshold.

**Why it wins:** MUI consumers must always write this themselves, often badly (no debounce, wrong IntersectionObserver target, no cleanup). By shipping the hook, we ensure consistent behavior across the ecosystem and mean that the "full Extended FAB with scroll collapse" use case is a 5-line integration, not a 50-line one.

**Task:** Implement `useExtendedFabCollapse(options?: { threshold?: number; scrollRef?: RefObject<Element> })` in `packages/hooks`. Default threshold: 100 px of scroll past the page top. Export from the `@kafui/hooks` package root. Add Storybook + Jest test.

### 5. No-icon fallback warning — prevent silent broken collapse
**What we do:** in development mode (`process.env.NODE_ENV !== 'production'`), if `collapsed={true}` is set but no `icon` prop is provided, the component logs a `console.warn`.

**Why it wins:** MUI's Fab doesn't have this problem because it has no `collapsed` concept — but in a library that does, this edge case will trip up every consumer eventually. An early warning is far cheaper than debugging a collapsed FAB that shows nothing.

**Task:** Add a dev-only guard: `if (collapsed && !icon) console.warn('[ExtendedFab] collapsed=true requires an icon prop; ignoring collapse')`. Remove the `.extended-fab--collapsed` class when `collapsed && !icon` to prevent the broken visual state.

### 6. Correct staggered motion — M3 Expressive spec, not MUI's flat transition
**What we do:** the label fade completes in `--duration-short2` (~100 ms) while the container width transition takes `--duration-medium2` (~300 ms). This is the M3 Expressive motion specification for FAB collapse.

**Why it wins:** MUI, if this were implemented, would likely use a single `transition` with the same duration on all properties — the M3 stagger (label fades early, width follows) gives a more polished, intentional feel. The stagger makes the collapse read as "the label leaves first, then the container catches up" — consistent with M3's motion language.

**Task:** Implement two separate `transition` rules: one on `.extended-fab` for `inline-size` / `padding-inline` at `--duration-medium2`, and a separate `transition` on `.extended-fab__label` for `opacity` / `inline-size` at `--duration-short2`. Add a Storybook story with `@prefers-reduced-motion: no-preference` forced for reviewers.

### 7. Logical padding — auto RTL, no JS
**What we do:** `padding-inline-start: 16px; padding-inline-end: 20px;` (when icon present). These flip automatically under `dir="rtl"` — icon stays at inline-start.

**Why it wins:** MUI uses `paddingLeft`/`paddingRight` in inline styles or `ml`/`mr` in the `sx` prop — these are physical directions and do not flip in RTL. Every MUI RTL deployment requires manual style overrides or the `jss-rtl` / `stylis-rtl` plugin. Our CSS logical property approach needs zero RTL configuration.

**Task:** Use `padding-inline-start` and `padding-inline-end` in all CSS. Verify in a Storybook story with `dir="rtl"` applied to the container — icon should be on the right side without any extra code.

---

## Checklist

- [ ] **[Beat #1]** Standalone `ExtendedFab.tsx`; verify Fab import does not include ExtendedFab.
- [ ] **[Beat #2]** Unconditional `aria-label={label}`; RTL test asserting accessible name in collapsed state.
- [ ] **[Beat #3]** CSS collapse animation; Storybook interactive story with "Simulate Scroll" button.
- [ ] **[Beat #4]** `useExtendedFabCollapse` hook in `@kafui/hooks`; 5-line integration example in docs.
- [ ] **[Beat #5]** Dev-mode warning when `collapsed && !icon`; guard removes collapsed class.
- [ ] **[Beat #6]** Staggered motion — separate transition durations for label and container.
- [ ] **[Beat #7]** Logical padding; Storybook RTL story.
- [ ] Implement all four color variant modifier classes with correct token pairs.
- [ ] Implement `--lowered` elevation modifier.
- [ ] State-layer element driven by `data-hovered`, `data-focused`, `data-pressed`.
- [ ] `aria-disabled` via React Aria `isDisabled`; do NOT set HTML `disabled`.
- [ ] Elevation transition: `box-shadow` animated.
- [ ] Document no `size` prop — Extended FAB has exactly one size.
- [ ] Document no `href` support — intentional.
- [ ] Reduced motion: verify transitions are instant under `prefers-reduced-motion: reduce`.
