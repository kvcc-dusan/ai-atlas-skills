# Final validation checklist — code against the Figma screenshot

Adapted from Figma's own "Implement Design" skill, which treats this as a required,
named step rather than an implicit assumption. Run this against the screenshot captured
in Phase A, step 3 — not from memory of the layer tree.

The reason this exists as an explicit step: a model generating code has no innate
visibility into what that code will actually render as. "It compiles and looks
plausible" is not the same claim as "it matches the design," and treating them as
equivalent is exactly the silent-failure pattern this whole skill exists to avoid.

## Checklist

- [ ] **Layout** — spacing, alignment, and sizing behavior match. Specifically:
      re-check any node that was Fill/Hug sizing rather than Fixed — these are the
      values most likely to have been silently hardcoded anyway.
- [ ] **Typography** — font, size, weight, and line height match the grounded
      variable values, not an eyeballed approximation.
- [ ] **Colors** — every fill/stroke traces back to a grounded variable or an
      explicitly flagged unresolved value. No color should be present in the generated
      code that wasn't in the grounded set or explicitly noted as ungrounded.
- [ ] **Interactive states** — anything the Figma file shows as a variant (hover/press
      equivalent, disabled, selected) has a corresponding SwiftUI state handled, even if
      only as a placeholder.
- [ ] **Accessibility** — Dynamic Type will scale the text, icon-only controls have
      labels, and nothing depends on color alone to convey state.
- [ ] **Assets resolve** — every `Image("...")` call references an asset that either
      already exists in `.xcassets` or was actually fetched via an MCP asset tool in
      this session. A reference to an asset that was only *named*, never fetched or
      confirmed to exist, is a broken build waiting to happen — flag it, don't let it
      pass silently.
- [ ] **No invented components** — every DS component call was confirmed against
      either a Code Connect mapping or an actual search of the codebase, not against
      what a component "should probably be called."
- [ ] **Compiles** — if a build/compile check is available in the working environment,
      use it. A visibly broken build is a better outcome than a plausible-looking
      snippet nobody actually tried to compile.

## What to do when something fails this checklist

Say so plainly in the validation notes section of the output — don't quietly fix it and
present the final version as if it was correct on the first pass. Silently
self-correcting hides exactly the kind of mismatch a reviewer would want to know about
(e.g. "the DS token for this spacing didn't have an exact match, I used the nearest
existing token — confirm that's acceptable" is useful information; presenting the
adjusted value with no note is not).
