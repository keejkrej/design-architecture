# Application design architecture research

Research date: 2026-10-11. Sources below are first-party repositories, actual source files, and maintainer documentation. These are widely adopted projects rather than a ranked list: GitHub stars measure repository interest, not design quality or user adoption. Source paths describe the branches inspected today and can change. Recommendations are inferences from the implementations, not claims that these projects have proven agent productivity gains.

## The strongest application references

| Project | Purpose / relevant stack | Architecture worth studying |
| --- | --- | --- |
| [T3 Code](https://github.com/pingdotgg/t3code) | Agent control surface, TypeScript React web app plus desktop/mobile surfaces | App-local controls with semantic geometry, typed variants, and shared cross-platform palette data |
| [Cal.diy](https://github.com/calcom/cal.diy) | Scheduling; TypeScript web app and embedded scheduling | Semantic tokens shared across app and shadow DOM; separate UI, features, and platform atoms |
| [OpenCode](https://github.com/anomalyco/opencode) | Coding agent; TypeScript Solid web/desktop surfaces | Packaged UI with focused exports, adjacent CSS/stories, semantic token aliases, separate session UI |
| [Twenty](https://github.com/twentyhq/twenty) | CRM; TypeScript React | Explicit primitive / composition distinction, token generation, public entry points and scoped themes |
| [Plane](https://github.com/makeplane/plane) | Project management; TypeScript React frontend, Python backend | Shared primitives plus product blocks; documented surface semantics; migration adapters |
| [Dub](https://github.com/dubinc/dub) | Link attribution platform; TypeScript React/Next.js | Shared UI and Tailwind configuration with semantic backgrounds, borders and content colors |

License scope matters if copying implementation: T3 Code and OpenCode identify MIT licenses; Cal.diy is the MIT community edition, while the commercial Cal.com product is now distinct. Plane identifies AGPL-3.0. Twenty and Dub have mixed repository license arrangements, so inspect the specific package/file and enterprise boundary rather than treating every directory as permissively reusable. This is an architecture survey, not a license audit. [T3 Code repository](https://github.com/pingdotgg/t3code), [OpenCode UI manifest](https://github.com/anomalyco/opencode/blob/dev/packages/ui/package.json), [Cal.diy README](https://github.com/calcom/cal.diy/blob/main/README.md), [Cal.com maintainer explanation](https://cal.com/blog/cal-diy-open-source-to-closed-source), [Plane repository](https://github.com/makeplane/plane), [Twenty repository](https://github.com/twentyhq/twenty), [Dub repository](https://github.com/dubinc/dub).

## Concrete evidence and lessons

### Twenty: the clearest explicit hierarchy

Twenty's UI README distinguishes **primitives**, including compound interactive controls such as Select, Menu and Dialog, from **shared components**, which compose them into presets such as MainButton, LightButton, Dropdown and Toaster. The distinction is responsibility, not number of DOM elements. Both layers accept application data, labels and callbacks. Imports expose that distinction: `twenty-ui/primitives/input`, `twenty-ui/components/settings`, and `twenty-ui/theme`. This gives agents a searchable public vocabulary without requiring knowledge of internal paths. [UI README](https://github.com/twentyhq/twenty/blob/main/packages/twenty-ui/README.md).

`MainButton.tsx` imports the primitive `Button`, resolves a default variant using button-group context, and adds preset styles. This is a useful small composition: it owns a recurring visual decision without owning CRM business logic. [MainButton implementation](https://github.com/twentyhq/twenty/blob/main/packages/twenty-ui/src/components/input/MainButton/MainButton.tsx).

`design-tokens/designTokens.ts` aggregates color, background, text, spacing, font, animation and control-specific tokens. `scripts/generateThemeTokens.ts` derives light/dark CSS, CSS variable references, TypeScript types and constants from that source and has a `--check` mode for stale generated output. One canonical source prevents CSS and TypeScript representations drifting. Its README also documents theme scoping via `applyToRoot={false}` and public CSS variable overrides. [Token source](https://github.com/twentyhq/twenty/blob/main/packages/twenty-ui/design-tokens/designTokens.ts), [generator](https://github.com/twentyhq/twenty/blob/main/packages/twenty-ui/scripts/generateThemeTokens.ts), [theme contract](https://github.com/twentyhq/twenty/blob/main/packages/twenty-ui/README.md).

Caveat: the library calls itself alpha. Adopt the layering principle; assess API stability before making it a foundation dependency.

### Plane: product compositions and controlled migration

The current `preview` branch describes `@plane/blocks` as composite components and Plane-specific building blocks built on `@makeplane/propel` primitives. Exports cover auth, empty states, charts, layout, property selects and virtual lists. This boundary explicitly permits product knowledge in blocks while keeping foundational controls independently owned. [Blocks package manifest](https://github.com/makeplane/plane/blob/preview/packages/blocks/package.json).

Its empty-state action adapter imports `Button` from `@makeplane/propel/components/button` and maps legacy public variants and icon props to the newer control. The comment explains preserving roughly 150 callers during migration and eventual adapter removal. This is concrete technical-debt management: a single temporary translation boundary instead of scattering compatibility fixes through screens. [Migration adapter](https://github.com/makeplane/plane/blob/preview/packages/blocks/src/empty-state/action-button.tsx).

`packages/tailwind-config/AGENTS.md` defines canvas, surfaces and layers with correct and incorrect examples. The sole root canvas and sibling surface rules give agents architectural meaning for color tokens, rather than a bare color list. These specific rules belong to Plane's visual language; the transferable lesson is to document semantic usage and exceptions. [Agent-facing surface rules](https://github.com/makeplane/plane/blob/preview/packages/tailwind-config/AGENTS.md).

### OpenCode: explicit package API and adjacent evidence

`@opencode/ui` exposes focused subpaths such as `@opencode/ui/button`, styles, tokens, theme and icons. Internally it groups actions, forms, overlays, navigation, layout, data display and feedback. Button code, CSS and Storybook stories are colocated. The button wraps Kobalte's behavior and sets typed size/variant data attributes, leaving state styling in its adjacent CSS. [Manifest and public exports](https://github.com/anomalyco/opencode/blob/dev/packages/ui/package.json), [button](https://github.com/anomalyco/opencode/blob/dev/packages/ui/src/actions/button/button.tsx), [button stories](https://github.com/anomalyco/opencode/blob/dev/packages/ui/src/actions/button/button.stories.tsx).

`styles/tokens/theme.css` aliases raw palettes to roles for background, text, icon, border, overlay and state. It also contains agent-purpose colors, illustrating that domain semantics can be layered on a shared language. Some colors remain literal values: this is useful source evidence, not an idealized claim of perfect token discipline. The UI's `AGENTS.md` gives concrete typography rules, including why tight line boxes clip descenders. [Semantic tokens](https://github.com/anomalyco/opencode/blob/dev/packages/ui/src/styles/tokens/theme.css), [UI agent instructions](https://github.com/anomalyco/opencode/blob/dev/packages/ui/AGENTS.md).

### Cal.diy: one language across app and embed

`packages/config/theme/tokens.css` explicitly serves both the web app and embedded elements' shadow DOM. It maps semantic backgrounds, text, borders, status and visualization colors to Tailwind variables, defines radii and motion, and initializes variables on `:root, :host`. This is a valuable cross-application example: consistency needs deliberate delivery into isolated rendering roots. [Shared token CSS](https://github.com/calcom/cal.diy/blob/main/packages/config/theme/tokens.css).

The existing shared button owns typed variants, loading behavior, icons, tooltip support and button/link rendering. Source also has separate `packages/coss-ui`, `packages/ui`, and platform atom controls, so a mature repository can contain multiple generations of abstraction. Do not reproduce that history as a target architecture: document the canonical entry point and transition path. [Button implementation](https://github.com/calcom/cal.diy/blob/main/packages/ui/components/button/Button.tsx), [coss-ui package](https://github.com/calcom/cal.diy/blob/main/packages/coss-ui/package.json), [platform button](https://github.com/calcom/cal.diy/blob/main/packages/platform/atoms/src/components/ui/button.tsx).

### T3 Code and Dub: bounded APIs and geometry

T3 Code's button uses Base UI rendering helpers plus class-variance-authority for typed variants and sizes. It explicitly reserves exported `buttonVariants` for other UI modules; application code should render `Button`. That prevents callers from cloning visual recipes onto arbitrary elements. Its central CSS documents semantic geometry variables so sidebars, palettes, tooltips and toolbars do not drift. [Button contract](https://github.com/pingdotgg/t3code/blob/main/apps/web/src/components/ui/button.tsx), [geometry and theme CSS](https://github.com/pingdotgg/t3code/blob/main/apps/web/src/index.css).

Dub's `packages/ui/src/button.tsx` centralizes variants/loading/icon behavior and imports utilities from `@dub/utils`; its shared Tailwind theme separates background, border and content roles with light/dark values. This demonstrates a practical monorepo seam, but some button branches still use raw neutral/red utilities. Shared packages alone do not ensure complete semantic styling. [Button](https://github.com/dubinc/dub/blob/main/packages/ui/src/button.tsx), [theme](https://github.com/dubinc/dub/blob/main/packages/tailwind-config/themes.css).

## Project constraint

The chosen architecture must always use shadcn/ui or a shadcn-derived framework/native implementation. The surveyed applications provide architectural lessons; their existing component stacks are not proposed replacements for shadcn. See [the architecture](design-architecture.md) for the React, Vue, Svelte, Solid and React Native mapping.

## Synthesis for avoiding technical debt

Recommended inference: keep shared tokens, interactive primitives and reusable presentation patterns stable; keep product workflows and data ownership in feature modules. Promote a pattern once genuinely repeated behavior and visual rules are understood. Supply typed variants, focused exports, examples/stories, usage rules and generated-token checks. Treat migration adapters as explicit temporary boundaries with removal criteria. Avoid a universal screen component full of product switches, and avoid letting each agent invent a second button or spacing scale. Shared language should govern geometry, type, interaction, state and elevation while allowing each app to choose its own layout and domain compositions.
