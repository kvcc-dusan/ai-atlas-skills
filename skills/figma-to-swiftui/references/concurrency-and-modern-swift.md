# Concurrency and modern Swift — generation rules

Confirm the project's concurrency mode in pre-flight before writing any ViewModel —
code that's correct under Swift 5 mode can fail to compile under Swift 6 strict
concurrency, and the fix isn't cosmetic once a project has switched.

## If the project runs Swift 6 strict concurrency

- **Mark ViewModels `@MainActor`.** UI state must be updated on the main actor; marking
  the whole ViewModel main-actor-isolated makes every property and method main-actor by
  default instead of requiring per-member annotation.
- **Wrap async work in `Task { }` and `await` inside it**, keeping state updates on the
  main actor implicitly via the `@MainActor` annotation. Do not manually dispatch with
  `DispatchQueue.main.async` — that pattern predates structured concurrency and fights
  the compiler's isolation checking rather than working with it.
- **Use `nonisolated` deliberately, only for genuine background work** (e.g. a pure
  computation with no UI-state access), and hop back explicitly via `MainActor.run { }`
  or by calling back into a `@MainActor` method — never assume isolation is implicit.
- **Views using `@StateObject`/`@ObservedObject` must themselves be `@MainActor`** under
  Swift 6 — if the View isn't already isolated, adding a `@MainActor` ViewModel to it
  can surface new compiler errors at the View layer, not just the ViewModel. Flag this
  as a likely follow-up if the View wasn't already generated with that in mind.
- Prefer the `@Observable` macro (Observation framework) over legacy
  `ObservableObject`/`@Published` for new ViewModels when the project's minimum iOS
  version supports it (iOS 17+) — confirm the version floor in pre-flight before
  assuming this is available.

## If the project is still on Swift 5 / not strict-concurrency-clean

- Don't add `@MainActor`/strict-isolation annotations speculatively — they can conflict
  with an existing non-strict codebase's patterns and create friction the project isn't
  ready for. Match whatever pattern the project's existing ViewModels already use
  (check a real example in the codebase, don't assume).
- If the project context file doesn't say either way, this is exactly the kind of gap
  pre-flight should have caught — ask rather than guessing which mode applies.

## Why this matters enough to be its own file

A ViewModel that "looks right" but doesn't compile under the project's actual
concurrency mode is a worse outcome than one that's obviously incomplete — it costs a
compile-error debugging cycle instead of an honest TODO. This is the SwiftUI-specific
instance of the same principle the rest of this skill enforces everywhere else: an
unverified assumption presented with confidence is the failure mode to design against,
not merely a stylistic nice-to-have.
