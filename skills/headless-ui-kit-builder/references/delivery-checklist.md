# Delivery Checklist

Use this file when implementing or reviewing a Headless UI-based UI kit.

## Rollout Order

1. Inspect existing primitives and token sources.
2. Add the dependency if the project does not already have `@headlessui/react`.
3. Create the token and styling layer.
4. Build the public primitive entry point.
5. Implement foundation primitives.
6. Implement Headless UI wrappers.
7. Add Storybook or another review surface.
8. Run verification.
9. Migrate production screens later unless the user explicitly asked for an immediate migration.

## Story Coverage

Each story group should show real states, not just happy-path screenshots.

- foundations:
  surface comparison, token usage, branded variants
- buttons:
  primary, secondary, ghost, soft, danger, loading, disabled, icon-only
- forms:
  default, help text, error, selected, open, empty results, customer or branded variant if applicable
- overlays:
  menu open, popover open, dialog open, destructive footer
- navigation:
  tab selection, disclosure open and closed

## Verification

At minimum:

- run lint
- run the relevant test suite
- build Storybook if the project uses it

When the repository supports it, also add:

- source-level tests for expected primitive files and public exports
- visual tests or screenshots for overlay stacking and floating-panel behavior

## Common Failure Modes

- Root Headless UI components render as `Fragment`, so wrapper props cannot be forwarded.
- Floating panels lose inherited surface tokens and appear transparent or mismatched.
- Dialog backdrop and panel stacking are inverted or ambiguous.
- Storybook demos do not reflect real product layout constraints.
- The UI kit duplicates an existing token system instead of codifying it.

## Review Standard

The kit is not done when the components merely compile. It is done when:

- the primitive API is coherent
- the stories reflect real usage
- overlays stack correctly
- branded and multi-surface styling works
- the kit clearly reduces the need to re-specify UI behavior in future tasks
