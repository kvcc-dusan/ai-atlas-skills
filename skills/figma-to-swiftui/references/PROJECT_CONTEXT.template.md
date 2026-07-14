# Project context template — Figma → SwiftUI

Fill this in once per client codebase and save it as `CLAUDE.md` / `AGENTS.md` (or fold
it into whichever rules file your IDE already uses) at the repo root. This is the file
`figma-to-swiftui`'s pre-flight step looks for — without it, the skill either bootstraps
one from a `create_design_system_rules`-style MCP call plus a codebase scan, or asks a
grouped question and proceeds on a stated default. Filling this in once removes the
need for either.

Modeled on Figma's own published "design system rules" template, adapted for a native
SwiftUI target instead of React/Tailwind.

---

## Architecture

- State-management / architecture pattern: `[e.g. plain MVVM / TCA / @Observable-based]`
- Dependency injection approach: `[e.g. environment objects, a DI container, manual init]`
- Navigation pattern: `[e.g. NavigationStack with a Router type, coordinator pattern]`
- View/ViewModel file convention: `[e.g. ScreenNameView.swift + ScreenNameViewModel.swift, one per screen]`

## Swift & concurrency

- Minimum iOS / Swift version: `[e.g. iOS 17+, Swift 5.10]`
- Swift 6 strict concurrency enabled? `[yes / no / partial-per-module]`
- If yes: ViewModels must be `[@MainActor / actor-isolated how?]`
- Preferred async pattern: `[Task { } + async/await — confirm no legacy completion-handler style expected]`

## Design system

- Component library location: `[e.g. Packages/DesignSystem/Sources/]`
- Is Code Connect set up for this library? `[yes/no — if yes, get_code_connect_map is authoritative for component identity]`
- Component catalogue (if Code Connect isn't set up — list what exists):
  - `[ComponentName]` — `[Swift signature, e.g. DSButton(title:style:action:)]`
  - `[ComponentName]` — `[Swift signature]`
- Token namespace: `[e.g. DS.Spacing.md, Theme.Colors.primary]`
- What must never be hardcoded: `[e.g. colors, spacing — always tokens; corner radius may be literal for one-off shapes]`

## Assets

- `.xcassets` catalogue location(s): `[path(s)]`
- Async image loading: `[project's existing component/pattern, or "use AsyncImage"]`
- Lottie or other animation library in use: `[yes/no, package name]`

## Localization

- `Localizable.strings` path(s) per module: `[module → path mapping]`
- Key naming convention: `[e.g. snake_case, screen_element_role]`
- Languages maintained: `[e.g. en (primary), + list of others]`

## Accessibility & quality bar

- Any accessibility standards beyond default WCAG/HIG expectations: `[list]`
- SwiftLint config location: `[path, or "not used"]`
- Anything this project intentionally does differently from generic SwiftUI best
  practice, and why: `[document the why — arbitrary-looking rules get silently
  "corrected" by a model unless the reasoning is stated]`

---

## Keeping this current

Review this file when the architecture changes, the design system gets a major
version bump, or Code Connect mappings are added/removed. A stale file is worse than
no file — it produces confidently wrong assumptions instead of an explicit question.
