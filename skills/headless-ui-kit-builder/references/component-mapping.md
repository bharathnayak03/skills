# Component Mapping

Use this file when deciding which parts of the UI kit should be native primitives and which should wrap Headless UI.

## Core Rule

Use Headless UI when the component's value comes from behavior and accessibility state management. Stay native when the component's value is mostly styling and the browser already provides correct semantics.

## Usually Native

- `Button`
- `IconButton`
- `Card`
- `Pill`
- `Spinner`
- simple layout helpers

These usually need project styling, loading states, icon placement, and surface variants more than they need a behavior library.

## Usually Headless UI

- `Dialog`
- `DropdownMenu`
- `Select` with `Listbox`
- `Combobox`
- `Tabs`
- `Switch`
- `RadioCards` with `RadioGroup`
- `Disclosure`
- `Popover`

These components benefit from managed focus, keyboard navigation, active and selected state, dismissal rules, and ARIA wiring.

## Conditional Primitives

- `Button`
- `Input`
- `Textarea`
- native `Select`
- `Field`
- `Label`
- `Description`

Headless UI offers form primitives for these. Use them when the codebase values:

- consistent `data-*` state hooks
- field-level disabled propagation
- shared labeling and description semantics
- alignment with a broader Headless UI-based form system

Stay native when:

- the project already has stable field wrappers
- the added abstraction does not remove real complexity
- the team wants to minimize wrapper depth

## Floating Panel Guidance

Menus, popovers, combobox options, and listbox options need extra care.

- Assume they may be portaled or otherwise detached from the trigger subtree.
- Reapply surface classes and CSS variables on the panel itself if styling depends on inherited tokens.
- Give panels explicit width rules when they should match the trigger.
- Verify stacking against surrounding fields, sticky bars, and backdrops.

## Recommended First Milestone

1. Foundation primitives:
   `Button`, `IconButton`, `Card`, `Pill`, `Field`, `TextInput`, `Textarea`
2. Interactive wrappers:
   `Dialog`, `DropdownMenu`, `Select`, `Combobox`, `Switch`, `RadioCards`, `Tabs`, `Disclosure`, `Popover`
3. Documentation surface:
   Storybook stories and source-level verification
4. Migration:
   move a small real flow only after the primitive API is stable

