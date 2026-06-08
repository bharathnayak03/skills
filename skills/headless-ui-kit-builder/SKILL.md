---
name: headless-ui-kit-builder
description: Build or retrofit a reusable UI kit for React applications using Headless UI, shared design tokens, wrapper primitives, Storybook coverage, and incremental migration guidance. Use when Codex needs to create a design-system layer, standardize inconsistent UI primitives, add accessible Headless UI wrappers, document a component kit, or plan a migration from one-off UI patterns to shared primitives.
---

# Headless UI Kit Builder

Inspect the existing frontend before proposing primitives. Extract the app's current visual language, token sources, accessibility patterns, and repeated UI shapes. Prefer additive work that fits the codebase over a full rewrite unless the user explicitly asks for replacement.

## Workflow

1. Audit the current UI layer.
2. Define the smallest useful primitive set.
3. Map Headless UI only where state management and accessibility are non-trivial.
4. Build a shared token and styling layer.
5. Add documentation and verification surfaces.
6. Plan or execute migration in controlled slices.

## Audit The Existing UI

- Search for existing buttons, inputs, modals, menus, tabs, pills, cards, and customer or admin shells.
- Identify the real runtime token sources such as CSS variables, Tailwind theme extensions, utility classes, or shared style helpers.
- Separate surfaces if the app has materially different contexts such as admin and customer experiences.
- Identify existing Storybook, visual testing, or route-level preview infrastructure before inventing a new review flow.
- Prefer reading actual components and CSS over inferring a design system from page screenshots alone.

## Define The Primitive Set

Start with the smallest primitive layer that removes duplication and supports the real product surfaces.

- Always define a public entry point such as `src/components/ui-kit/index.*`.
- Prefer foundation primitives first: `Button`, `IconButton`, `Card`, `Pill`, `Field`, `TextInput`, `Textarea`, loading states.
- Add interactive wrappers next: `Dialog`, `DropdownMenu`, `Select`, `Combobox`, `Switch`, `RadioCards`, `Tabs`, `Disclosure`, `Popover`.
- Leave domain components such as `OrderCard` or `TableTile` out of the initial kit unless the user explicitly asks for a product-level library.
- Keep migration compatibility in mind. If the codebase already has `Button` or `Modal`, decide whether to wrap, replace internally, or add a parallel kit first.

Read [references/component-mapping.md](references/component-mapping.md) when choosing which primitives should use Headless UI and which should stay native.

## Use Headless UI Deliberately

Use Headless UI for components where keyboard behavior, focus management, open state, selection state, roving focus, dismissal, or ARIA wiring are easy to get wrong.

- Prefer Headless UI for `Dialog`, `Menu`, `Listbox`, `Combobox`, `Tabs`, `Switch`, `RadioGroup`, `Disclosure`, and `Popover`.
- A native `<button>` is acceptable for a general design-system button if the project only needs semantic button behavior plus project-specific loading and styling.
- If the project values consistent `data-*` state hooks across all primitives, using Headless UI `Button`, `Input`, `Textarea`, `Select`, and related form primitives can be reasonable.
- When using Headless UI APIs, fetch the current docs with Context7 instead of relying on memory. This matters because wrapper structure, root `as` behavior, and floating-panel behavior change over time.

## Build The Token Layer

- Reuse the app's existing tokens first. Do not invent a second theme system unless the user asked for a rebrand.
- Put semantic surface decisions in shared helpers, not inline styles spread across every primitive.
- For multiple surfaces, expose a small explicit API such as `surface="admin" | "customer"` or equivalent.
- Use CSS variables for surface-specific values so portaled overlays and floating panels can be restyled explicitly when they stop inheriting from their trigger container.
- Keep primitives scoped and additive. Avoid leaking UI-kit styles into unrelated legacy components.

## Reduce Future Token Cost

Treat the UI kit as a compression layer for future prompts and future code edits.

- Prefer primitive names that encode intent clearly enough that future requests can refer to them without re-describing behavior.
- Build small, stable prop APIs such as `variant`, `size`, `tone`, `surface`, `open`, and `onClose` instead of ad hoc booleans and style overrides.
- Centralize repeated decisions such as radii, spacing, hover states, focus states, and brand-color behavior so later work can reference one primitive instead of re-specifying visual details.
- Add a public barrel export and a predictable folder structure so future agents can orient quickly without reading unrelated page code.
- Use Storybook stories as a compact behavioral reference. A future request can say “match the customer button from the UI kit stories” instead of restating all visual states.
- Prefer migration toward the UI kit in product code once the primitives are stable. The token savings only materialize when the kit becomes the default vocabulary of the app.

In practice, this reduces token cost in later work by shrinking three things:

- prompt size:
  the user can ask for `Dialog`, `Select`, or `Button surface="customer"` instead of describing styling and behavior from scratch
- exploration cost:
  future agents can inspect `ui-kit/index.*`, the token layer, and the stories instead of reading many pages to infer conventions
- output size:
  code changes become smaller because new screens compose existing primitives instead of re-implementing controls and states

## Storybook And Verification

- Add Storybook stories if the repo already uses Storybook or if the user asked for a UI kit review surface.
- Show both default and edge states: disabled, loading, focus, open, error, selected, destructive, empty, branded variants.
- Keep the story layer close to how product code will use the primitives. Do not create decorative demo-only APIs.
- Add source-level guards when useful: expected primitive files, public exports, token CSS markers, and story coverage.
- Verify with the repo's real commands. For UI kit work this commonly includes lint, unit or source-level tests, and Storybook build.

Read [references/delivery-checklist.md](references/delivery-checklist.md) for rollout order, story coverage, and verification expectations.

## Migration Strategy

Choose one of these approaches based on the codebase:

- Additive first:
  Create `ui-kit/`, document it, and migrate pages later. Use this when the app is already shipping and broad churn would be risky.
- Compatibility bridge:
  Keep legacy component names but replace their internals with UI kit primitives. Use this when there are many call sites and the public API is stable enough.
- Domain-first:
  Build high-level domain components only after the primitive contract has proved itself in product code.

Prefer incremental migrations by feature or surface. Avoid cross-app rewrites unless the user explicitly asks for them.

## Implementation Heuristics

- Keep APIs boring and predictable. The kit should reduce future prompt size by becoming the default vocabulary of the codebase.
- Prefer one obvious prop shape over many aliases.
- Optimize naming and exports for reuse by other agents. If a future request still has to describe the primitive in prose every time, the kit is not compressing context well enough.
- Preserve existing visual language unless the user asked for a redesign.
- Treat floating overlays as a separate styling problem. Portaled menus, popovers, listboxes, and combobox panels often need explicit surface classes, explicit width rules, and explicit z-index handling.
- Keep Storybook examples accurate. If a story reveals a broken stacking or inheritance model, fix the primitive rather than masking it in the story.
