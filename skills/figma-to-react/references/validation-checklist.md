# Final validation checklist — code against the Figma screenshot

Adapted from Figma's own "Implement Design" skill, which treats this as a required,
named step rather than an implicit assumption. Run this against the screenshot(s)
captured in Phase A, step 3 — one per breakpoint frame if several exist — not from
memory of the layer tree.

The reason this exists as an explicit step: a model generating code has no innate
visibility into what that code will actually render as. "It typechecks and looks
plausible" is not the same claim as "it matches the design," and treating them as
equivalent is exactly the silent-failure pattern this whole skill exists to avoid.

## Checklist

- [ ] **Layout** — spacing, alignment, and sizing behavior match. Specifically:
      re-check any node that was Fill/Hug sizing rather than Fixed — these are the
      values most likely to have been silently hardcoded anyway — and confirm no
      content container inherited the frame's canvas height as a CSS `height`.
- [ ] **Typography** — font family, size, weight, and line height match the grounded
      variable values, not an eyeballed approximation.
- [ ] **Colors** — every color traces back to a grounded variable expressed in the
      project's canonical token representation (see `responsive-and-styling.md`), or is
      an explicitly flagged unresolved value. No color in the generated code that
      wasn't in the grounded set or flagged.
- [ ] **Styling approach** — no second styling system snuck in: no Tailwind classes in
      a CSS Modules project, no inline styles where the project has a convention.
- [ ] **Responsive claims** — every responsive class/media query traces to a grounded
      breakpoint frame or to a listed assumption. Nothing responsive appears that was
      neither designed nor flagged.
- [ ] **Interactive states** — anything the Figma file shows as a variant (hover,
      pressed, disabled, selected, focus) has a corresponding handled state, even if
      only as a placeholder — and focus is *visible*.
- [ ] **Accessibility** — semantic elements for interactive things (`button`/`a`, not
      clickable divs), inputs labelled, images have `alt`, nothing conveys state by
      color alone, keyboard reachability plausible.
- [ ] **List keys** — every `.map()` has a stable key from the data.
- [ ] **Identifiers** — no prop or component name collides with React/DOM-reserved
      names (`key`, `ref`, `children`, `className`, `style`) or JS reserved words —
      the Figma-layer-name trap.
- [ ] **Assets resolve** — every import/URL references an asset that either already
      exists in the project or was actually fetched via an MCP asset tool this session.
      An asset that was only *named*, never fetched, is a broken build waiting to
      happen — flag it, don't let it pass.
- [ ] **No invented components** — every DS import was confirmed against a Code
      Connect mapping or an actual codebase search, not against what a component
      "should probably be called."
- [ ] **Typechecks / lints** — run `tsc --noEmit` and the project's linter if the
      environment allows; if a dev server is available, render the component and
      compare against the screenshot directly. A visibly broken build beats a
      plausible-looking snippet nobody tried to compile.

## What to do when something fails this checklist

Say so plainly in the validation notes — don't quietly fix it and present the final
version as if it was correct on the first pass. Silently self-correcting hides exactly
the kind of mismatch a reviewer would want to know about ("the DS token for this
spacing had no exact match, I used the escape-hatch spelling with a TODO — confirm
that's acceptable" is useful information; the adjusted value with no note is not).
