# A design architecture for agents, multiple apps, and native mobile

Research date: 2026-10-11. This document combines primary-source research with an architectural recommendation. Observations about projects are cited; proposed structure and rules are recommendations, not evidence that any particular repository has measured better AI-agent outcomes. See [application source findings](app-research.md) for the detailed app survey.

## Recommendation

Build one design language on **shadcn as the required foundation**, with a canonical token source, explicit control and pattern contracts, and framework/platform-specific rendering. Give coding agents a small, searchable public surface with working examples. Give runtime UI-generating models a narrower, validated catalog backed by those same renderers.

Consistency has four parts: visual roles, interaction behavior, composition rules, and product terminology. Sharing only colors leaves the other three open to drift. Sharing every screen creates coupling between products. The useful seam is the reusable design decision: a control's states, a form field's label/error relationship, a toolbar's hierarchy, or an empty state's recovery action.

For React web and a React DOM desktop app, share the UI implementation. For React Native, share tokens, applicable presentation logic and semantic contracts, then implement native controls and layouts. Platform adaptations are part of the design system and should have examples and tests.

## Required foundation: shadcn across frameworks and platforms

This is an explicit project constraint: all UI implementations build on shadcn/ui or an appropriate shadcn-derived framework/native port. Framework changes do not authorize adopting an unrelated design system. Research references inform architecture; they are not alternative component foundations.

| Target | Starting implementation | Ownership |
| --- | --- | --- |
| React web / React DOM desktop | [shadcn/ui](https://ui.shadcn.com/docs) | Upstream React implementation; vendor into the owned shared UI package |
| Vue / Nuxt | [shadcn-vue](https://www.shadcn-vue.com/docs/introduction) | Framework port; align its tokens and public design roles |
| Svelte / SvelteKit | [shadcn-svelte](https://www.shadcn-svelte.com/docs) | Community port; align its tokens and public design roles |
| Solid / SolidStart | [Zaidan](https://zaidan.carere.dev/docs) | User-selected shadcn-based Solid implementation and registry; vendor and align its controls |
| React Native / Expo | [React Native Reusables](https://github.com/founded-labs/react-native-reusables) | shadcn-inspired native implementation; align tokens and native interaction rules |
| Other frameworks | A maintained shadcn-derived port, or an owned implementation of the same control contracts | Evaluate actual component coverage before adopting; do not assume a port exists |

These projects are separately maintained; they do not provide a single interchangeable package or identical APIs. shadcn-svelte explicitly describes itself as an unofficial port. Zaidan follows shadcn structure and CLI distribution while using Solid-native Kobalte and Corvu behavior primitives. [Zaidan introduction](https://zaidan.carere.dev/docs), [Zaidan source](https://github.com/carere/zaidan). React Native Reusables describes its goal as bringing shadcn/ui to React Native, rather than being the official React Native edition of shadcn/ui. Underlying behavior dependencies may differ without creating a second visual language.

Own the installed source in one shared package per renderer. Map a shared semantic token contract into each implementation's theme conventions, including paired foreground/background colors, border, input, ring and radius roles. Normalize approved variants and state meanings deliberately; similar upstream defaults do not guarantee cross-framework parity. Use framework-native composition and event conventions rather than forcing JSX or React hooks into Vue, Svelte or Solid.

For missing controls, extend the owned shadcn-derived system with a documented contract and examples. Specialist engines such as editors or charts can live behind an adapter whose surrounding controls and presentation follow the same language. Runtime generative UI also resolves through these owned shadcn-derived implementations.

## What the reference projects teach

| Reference | Observed approach | Architectural lesson |
| --- | --- | --- |
| T3 Code | Typed app controls; centralized geometry; documented restrictions on consuming button recipes | Put recurring decisions behind controls and semantic geometry names |
| Cal.diy | Token CSS delivered to both app roots and embedded shadow DOM | Make theme delivery explicit for each rendering environment |
| OpenCode | Solid UI package with focused exports, adjacent CSS/stories and semantic color roles | A well-defined public vocabulary transfers across frameworks |
| Twenty | Explicit primitives/compositions; token generation with a stale-output check | Responsibilities and generated artifacts make the hierarchy usable |
| Plane | Product blocks over shared controls; central migration adapter; agent-facing surface rules | Keep product knowledge at the right level and contain migrations |
| Dub | Shared UI and theme packages, with some literal styling still present | Packaging is useful but does not enforce consistency by itself |

Sources and specific file paths for all six observations are in [app-research.md](app-research.md). OpenCode uses Solid rather than React; Plane also has a Python backend. They belong in this survey for their TypeScript UI architecture. The sample covers recognizable projects across different product categories; it is not a popularity ranking or a visual-quality benchmark. Some repositories have mixed or commercial license boundaries.

Vercel provides three complementary references. Geist documents visual foundations, including color roles and coordinated typography styles. Its AI prototyping guidance recommends tokens, components, reusable blocks and registry distribution. Vercel's open-source chatbot demonstrates a concrete app implementation with shared semantic CSS and typed control variants. This supports a system of rules and reusable implementations rather than treating an attractive dashboard as the design system. The published Geist documentation does not by itself establish availability of an official public component package. [Geist colors](https://vercel.com/geist/colors), [Geist typography](https://vercel.com/geist/typography), [Vercel design-system guidance](https://vercel.com/blog/ai-powered-prototyping-with-design-systems), [chatbot theme source](https://github.com/vercel/chatbot/blob/c2f8235e1f3ea903ad8b7f61447c4f74164b5c58/app/globals.css), [chatbot button](https://github.com/vercel/chatbot/blob/c2f8235e1f3ea903ad8b7f61447c4f74164b5c58/components/ui/button.tsx).

AI Elements is also useful: it layers conversation, message, prompt-input and other AI-specific building blocks over shadcn conventions and inherits the existing theme. It illustrates how a product-category vocabulary can sit above foundational controls. Source installation still requires an update policy if multiple apps copy the code. [AI Elements](https://elements.ai-sdk.dev/).

## Layers: classify by responsibility

These are conceptual levels, not a requirement for six packages or for every screen to pass through every level.

| Level | Owns | Examples | Does not own |
| --- | --- | --- | --- |
| Foundations | Visual roles and scales, assets, motion, density, accessibility policy | `surface.canvas`, `text.secondary`, `space.3`, type roles, icon rules | Product workflow |
| Behavior foundations | Existing platform control mechanics | Focus management, overlay dismissal, press/selection behavior | Your brand's complete styling |
| Styled primitives | A coherent control or layout operation, with supported states | Button, Text, Field, Dialog, Stack, Separator | Fetching, router or product store |
| Patterns / compositions | A recurring presentation or interaction problem | EmptyState, SearchToolbar, SettingsSection, AsyncContent, FormActions | Booking rules or CRM permissions |
| Shells / templates | A recurring arrangement of regions | Workspace frame, detail view frame, settings frame, mobile tab frame | A universal app navigation model |
| Feature UI | Domain meaning and workflow | BookingEditor, DealPipeline, AgentRunPanel | Reinventing the shared visual language |

An accessible Select can be a primitive even though it contains many elements. A SettingsSection can be a composition even if its implementation is small: it removes repeated decisions about hierarchy, spacing and actions. A layout helper is worthwhile when it encodes a rule; wrapping every `div` with another prop vocabulary creates little value.

Headless behavior comes from the dependencies of the selected shadcn implementation/port or from narrowly scoped behavior adapters. Keep that implementation behind the owned shadcn-derived controls where practical. Extract your own platform-neutral behavior only where real consumers share it. DOM focus, native gestures and platform navigation are not automatically portable because they use React.

## Tokens: one source, several outputs

Use a canonical structured token source, with a schema and generation/check command. DTCG's published format defines token types and aliases; adopting that format is useful for interchange but does not supply your brand, semantic roles or platform adaptations. A typed TypeScript source is also a reasonable starting point if external interchange is unnecessary. Choose one authoritative source. [DTCG Format Module](https://www.designtokens.org/tr/2025.10/format/).

Distinguish:

1. **Reference values:** palettes, spacing scale, font families, durations.
2. **Semantic roles:** canvas, raised surface, muted text, destructive action, focus indicator.
3. **Control-specific decisions where needed:** field height, sidebar width, dialog elevation. Do not create a public token for every internal property.

Generate web CSS variables and typed native theme data from the same source. Components consume semantic roles. Feature layout can use the shared spacing scale directly; it need not invent a semantic alias for each margin. Product statuses map to design intents in domain code: for example, booking cancellation may map to neutral or warning according to the product's rules. Do not overload `success` as a brand accent or let a runtime model invent authoritative business status.

Keep brand, color scheme, density and platform separate in the model. Define supported combinations and their precedence once. Platform mappings can adjust typography metrics, touch geometry and elevation. A compact desktop control and a comfortable mobile control can express the same hierarchy with different dimensions.

Document geometry and surface usage alongside values: which region is the canvas, when a surface needs elevation, when a divider is enough, where captions go, and how actions align. These rules keep a scheduling app, an IDE and a CRM related without giving them identical page layouts.

## Cross-platform consistency

React Native styles use JavaScript objects and have differences from browser CSS. Its accessibility implementation also varies between iOS and Android. React Native explicitly supports platform-specific files. These are reasons to make renderer seams deliberate rather than gradually filling every shared control with platform conditionals. [React Native styling](https://reactnative.dev/docs/style), [accessibility](https://reactnative.dev/docs/accessibility), [platform-specific code](https://reactnative.dev/docs/platform-specific-code).

| Concern | Shared design contract | Web / React DOM desktop | React Native mobile |
| --- | --- | --- | --- |
| Type | Roles, hierarchy, weights, numeric formatting | CSS font metrics and zoom behavior | Loaded font files, native metrics and user font scaling |
| Color | Semantic intent, supported light/dark schemes | CSS variables | Resolved native theme values |
| Controls | Emphasis, labels, disabled/loading meanings | Semantic DOM, hover, keyboard and focus | Native press handling, accessibility state and touch area |
| Overlays | Selection intent, dismissal policy, result state | Popover or dialog where appropriate | Sheet, dialog or separate screen as specified |
| Navigation | Destination names and information hierarchy | Sidebar, breadcrumbs, shortcuts | Tabs, stacks and native back gestures |
| Collections | Data, sorting/filter semantics and actions | Dense table or grid | List/detail presentation and mobile-appropriate controls |
| Motion | Purpose, rhythm and reduced-motion policy | Browser animation implementation | Native animation implementation |

Shared does not imply exact prop parity. For a portable Button, common intent might include emphasis, disabled, pending and label. Web links still need link semantics; native navigation needs a different renderer. Do not make every native control accept DOM events or force every web control into mobile-only props.

For desktop, this recommendation assumes Electron/Tauri or another shell rendering the same React DOM UI. Desktop title bars, window state and OS actions belong in the shell/platform adapter. A native desktop toolkit needs its own renderer just as mobile does.

### Implementation strategy under the shadcn constraint

Use framework/platform renderers with shared foundations. For React DOM, start with shadcn/ui; for React Native, use React Native Reusables with its supported styling setup. Align both with generated tokens and the same semantic pattern contracts. NativeWind provides utility-based native styling, but similar class names do not make CSS and native behavior identical. [NativeWind](https://www.nativewind.dev/docs), [React Native Reusables](https://github.com/founded-labs/react-native-reusables).

Tamagui and Restyle are outside the selected architecture because the required foundation is shadcn. Do not introduce a universal UI system to maximize shared markup. Share framework-neutral data contracts and pure presentation logic; keep React hooks in React renderers and framework-specific reactivity in its own implementation.

Evaluate each selected port with a real vertical slice: a form with validation, an overlay, a collection with detail navigation, and light/dark mode. Include keyboard use on desktop and font scaling/back behavior on mobile. Check token, variant and state parity across renderers. A button gallery cannot reveal the important tradeoffs. Pin compatible versions and record upstream provenance during implementation; this report intentionally does not prescribe versions.

## Repository organization

An illustrative multi-product layout:

```text
packages/
  design-tokens/
    source/                  # canonical values, roles and platform mappings
    generated/               # CSS and typed native values; never hand-edited
  ui-headless/               # shared React presentation state; pure logic can serve other frameworks
  ui/
    primitives/              # owned shadcn/ui React DOM controls
    patterns/                # reusable presentation and interaction modules
    shell/                   # frames and chrome shared by relevant web/desktop apps
  ui-native/
    primitives/              # React Native Reusables, aligned to shared shadcn roles
    patterns/                # native adaptations of the applicable patterns
    shell/                   # native frames/navigation presentation
  ui-vue/                    # add only for a real Vue consumer; shadcn-vue source
  ui-svelte/                 # add only for a real Svelte consumer; shadcn-svelte source
  ui-solid/                  # add only for a real Solid consumer; owned Zaidan source
  generative-ui/             # add when runtime generation actually exists
    catalogs/                # validated semantic schemas, scoped by task/surface
    web/                     # adapters into ui and approved feature renderers
    native/                  # adapters into ui-native and native feature renderers
apps/
  scheduling/{web,desktop,mobile}/
  workspace/{web,desktop,mobile}/
docs/design/
  language.md                # principles, surfaces, geometry, type and state rules
  architecture.md            # dependency rules, ownership and evolution policy
```

Feature UI lives in product-owned feature modules, either alongside the app for a single consumer or in product packages when web/desktop/mobile actually share the domain. A booking pattern reused by two scheduling apps can become shared scheduling UI without becoming a foundation for every product.

Dependency direction: feature UI uses patterns/primitives; renderers use tokens and applicable headless behavior; foundation packages do not import product stores, routers or network clients. The generative renderer is an adapter over existing UI, not the new owner of all UI behavior. Add packages only when consumers, independent dependencies or ownership justify them. Small projects can use these same folders inside one package first.

Expose focused imports such as `@org/ui/button` and `@org/ui/empty-state`. Colocate implementation, examples/stories and behavioral tests for each public module. Keep raw styling recipes private if exposing them would allow callers to reproduce the control without its semantics.

## Make the system usable by coding agents

Coding agents create reviewed source changes; runtime generative models create ephemeral or persisted UI specifications. They need different degrees of freedom.

For coding agents, supply a compact entry document pointing to:

- The required shadcn foundation, selected port for the target, canonical import paths and component index.
- A decision sequence: reuse an existing pattern, compose approved controls locally, then propose a new shared pattern only with concrete repeated needs.
- One or two real usage examples per pattern, including an important non-happy state.
- Where fetching, domain state, router adaptation and platform behavior belong.
- The supported style overrides and process for an exception or new variant.
- The commands for token checks, type checking, focused behavior tests and visual review.

Generate searchable indexes and schema information from exports/metadata where possible. Keep explanatory guidance short and link to executable examples rather than maintaining another extensive copy of the implementation. shadcn's registry and MCP workflows demonstrate distribution/discovery capabilities for components and blocks. They help agents find the design system; they do not ensure that the resulting screen follows it. [shadcn registry](https://ui.shadcn.com/docs/registry), [shadcn MCP](https://ui.shadcn.com/docs/mcp).

For multiple apps owned together, consume shared packages so fixes propagate. Use registries to discover, prototype or deliberately vendor code into the shared system. Installing independent copies into every app creates update and divergence work. Across separate repositories, version packages, document migrations and test representative consumers before release. A registry can distribute blocks effectively, but copied code still needs ownership and reconciliation.

Enforce what can be enforced: no deep imports, no dependencies from foundations into features, no duplicated token definitions, and no unapproved literal visual values in shared controls. Permit app layout code and sanctioned exceptions. Avoid an absolute ban on CSS or arbitrary dimensions: diagrams, third-party editors and computed geometry may need them.

## Runtime generative UI: reuse the design system

| Approach | What the model supplies | Where appearance lives | Appropriate role |
| --- | --- | --- | --- |
| AI SDK tool-result UI | Tool choice/arguments; tool results populate authored UI | Normal application renderers | Known tasks such as a booking lookup or report |
| Vercel json-render | A specification using catalog components/actions | Registry maps types into your renderers | Flexible composition within a controlled vocabulary |
| Thesys OpenUI | Structured OpenUI Lang output using allowed components | Library definitions map nodes to your renderers | Another structured, streaming composition approach |
| Open Intelligent UI | Native structured UI or generated HTML/CSS/JS in a sandbox | Native renderers plus supplied sandbox styles | Custom interactive explanations and exploratory artifacts |

Sources: [AI SDK generative UI](https://ai-sdk.dev/docs/ai-sdk-ui/generative-user-interfaces), [json-render catalog](https://json-render.dev/docs/catalog), [json-render registry](https://json-render.dev/docs/registry), [OpenUI custom design-system example](https://www.openui.com/docs/openui-lang/examples/design-systems/shadcn), [Open Intelligent UI architecture](https://github.com/CopilotKit/OpenIntelligentUI/blob/f6e4388b26a64b9a0714943b08a1ce622b924eec/docs/architecture.md). OpenUI here means Thesys's framework, not the similarly named browser-standards initiative or other projects called OpenUI. Open Intelligent UI includes a Python agent backend; it is not an all-TypeScript application.

Prefer catalog entries that encode useful design decisions: `Metric`, `Comparison`, `ResultList`, `FormSection`, `TimeSeries`, and approved domain presentations. Add bounded layout primitives where needed. Exposing only low-level boxes leaves too much composition work to the model; exposing a universal app schema full of workflow switches creates a maintenance burden.

Each catalog entry should define validated props, allowed children/slots, relevant states, usage guidance and its supported platforms. Keep arbitrary style/class strings and executable callbacks out of the default runtime catalog. The host supplies theme, density, locale, permissions and action execution. Validate model output rather than assuming a framework's schema or prompt guarantees valid generation. Define useful fallbacks for unsupported types, invalid data and incomplete streams.

For web/native, a semantic catalog can target two registries. For example, `ResultList` may render as a table on desktop and list rows on mobile. Report unsupported capabilities explicitly: a DOM-heavy editor or a custom 3D artifact does not become native merely because its enclosing JSON is portable. Use platform-specific catalogs or capability filters where required.

Open Intelligent UI is a useful concrete reference for sandbox delivery: its design-system package exports theme CSS, SVG rules and form styles shared by the app and MCP output. Its custom sandbox CSS is architectural evidence for theme delivery, not a replacement foundation under the shadcn constraint. Any adopted sandbox route must receive styles derived from the owned shadcn token/control system. Its single-source test checks that one token definition appears in exactly one source file. That helps contain token duplication, but does not prove every form style or generated artifact is consistent. [Shared design-system source](https://github.com/CopilotKit/OpenIntelligentUI/blob/f6e4388b26a64b9a0714943b08a1ce622b924eec/packages/design-system/src/index.ts), [single-source test](https://github.com/CopilotKit/OpenIntelligentUI/blob/f6e4388b26a64b9a0714943b08a1ce622b924eec/packages/design-system/src/__tests__/single-source.test.ts).

For a sandbox, deliver a versioned stylesheet/token artifact and synchronize the host's selected color scheme explicitly. Host CSS does not automatically cross iframe boundaries. Keep the host's standard UI frame consistent and define where custom artifact styling may vary. Generated CSS can still override supplied rules, so this route offers weaker guarantees of visual consistency than a renderer using owned controls. Native support would require a separate renderer or an intentionally embedded web surface.

Persisted UI specs need a catalog version and compatibility policy. Keep temporary streaming state separate from completed saved output. Provide migrations or compatible readers when renaming types/props; package versioning alone cannot repair old saved specifications. Do not expose every internal control as a permanent wire contract.

## Avoid technical debt during evolution

| Failure mode | Preventive rule |
| --- | --- |
| Each agent creates another Button or modal recipe | Canonical exports, discovery examples and import restrictions |
| A shared pattern accumulates product booleans | Keep workflow in feature UI; use named slots or smaller patterns |
| Similar markup is generalized too early | Require shared behavior/meaning and actual independent consumers |
| An existing pattern cannot meet a legitimate need | Provide local composition and a documented variant/extension route |
| App-specific CSS changes control internals | Use supported variants/slots; move a recurring decision into its owner |
| Web and native themes diverge | Generate both and check generated output in CI |
| Universal controls accumulate platform switches | Move substantive behavior to renderer adapters |
| Copied registry code drifts across apps | Vendor once into the shared package or maintain explicit copy/update ownership |
| A migration leaves two permanent systems | Central compatibility adapter, owner, removal criteria and caller inventory |
| Saved AI UI breaks after a release | Version the catalog and verify migration/reader compatibility |

A shared module should remove repeated decisions. If its callers must still know all its implementation details or override most of its styles, its interface is too broad or the abstraction is in the wrong place. Removing it should make meaningful complexity reappear in several callers; otherwise it may be a pass-through.

Assign ownership to tokens, each public pattern and each renderer. Mark public patterns stable, experimental or deprecated in the catalog/docs. Record exceptions with a reason and an owner. When a shared visual rule changes, update representative app/platform examples in the same change.

## Verification and rollout

Use checks with different purposes:

1. **Structure:** dependency/import rules, generated-token freshness, public type surfaces and catalog validation.
2. **Behavior:** keyboard/focus/overlay behavior on DOM; press, back, keyboard avoidance and accessibility states on native. Test through public interfaces.
3. **Visual evidence:** representative patterns and screens in light/dark, relevant density modes, narrow widths, long/localized content, loading, empty and error states. Include native font scaling and both iOS/Android where supported. Maintain platform-specific baselines rather than expecting identical screenshots.
4. **Generation:** render a small fixture suite of valid/invalid/partial specs, replay saved-version fixtures, and periodically inspect actual model outputs across supported tasks. Schema success does not establish useful layout, correct data or design quality.

Start with an opinionated visual language and a small control set. Implement two deliberately different product screens plus one corresponding native screen. Share the recurring decisions revealed by those screens. Once that vocabulary works, expose a small runtime catalog backed by it. Measure drift and overrides before broadening the abstraction.

The target is a system where a scheduling screen, a coding workspace and a mobile result view use recognizable type, surfaces, controls, states and terminology, while each retains the interaction model its task and platform require.
