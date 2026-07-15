# Project context template — Figma → React

Fill this in once per client codebase and save it as `CLAUDE.md` / `AGENTS.md` (or fold
it into whichever rules file your IDE already uses) at the repo root. This is the file
`figma-to-react`'s pre-flight step looks for — without it, the skill either bootstraps
one from a `create_design_system_rules`-style MCP call (with
`clientLanguages="typescript"`, `clientFrameworks="react"`) plus a codebase scan, or
asks a grouped question and proceeds on a stated default. Filling this in once removes
the need for either.

Modeled on Figma's own published "design system rules" template — React is the case
that template was primarily written for.

---

## Framework & build

- Framework flavor: `[Vite SPA / Next.js App Router / Next.js Pages / Remix / other]`
- If Next.js: server-component default? `["use client" only where state/effects require it — confirm]`
- TypeScript strictness: `[strict / notable deviations]`
- Component file convention: `[e.g. src/components/FeatureName/ComponentName.tsx, named exports]`

## Styling

- Approach: `[Tailwind / CSS Modules / styled-components / vanilla CSS custom properties / mixed — if mixed, which is canonical for new code]`
- If Tailwind — canonical token representation: `[config classes (bg-primary-500) / CSS custom properties (bg-[var(--x)]) — arbitrary values allowed only as flagged escape hatch?]`
- Token source of truth: `[tailwind.config.* / @theme block / tokens.css / theme object — path]`
- Breakpoint set: `[e.g. Tailwind defaults / custom screens map — list them]`
- What must never be hardcoded: `[e.g. colors, spacing — always tokens; radii may be literal for one-offs]`

## Design system

- Component library location: `[e.g. src/components/ui/, packages/design-system/]`
- Is Code Connect set up for this library? `[yes/no — if yes, get_code_connect_map is authoritative for component identity; note whether mappings use template files or the legacy React parser API]`
- Component catalogue (if Code Connect isn't set up — list what exists):
  - `[ComponentName]` — `[props signature, e.g. Button({ variant, size, onClick, children })]`
  - `[ComponentName]` — `[props signature]`
- Icon system: `[icon component set / sprite / library package — never inline fresh SVG if one exists]`

## State & data

- State management: `[local state only / Zustand / React Query / Redux — when each applies]`
- Data-fetching convention: `[e.g. React Query hooks per feature, server components fetch]`

## Assets

- Static asset location(s): `[src/assets / public/ — which for what]`
- Image component/pattern: `[plain <img> / next/image / project wrapper]`
- Animation library in use: `[none / framer-motion / lottie-react — package name]`

## Localization

- i18n library and message-file path(s): `[e.g. react-i18next, src/locales/<lang>/<ns>.json — or "none, plain strings"]`
- Key naming convention: `[e.g. feature.element.role]`

## Accessibility & quality bar

- Standards beyond default WCAG expectations: `[list]`
- Lint/typecheck commands the skill should run in Phase C: `[e.g. pnpm tsc --noEmit && pnpm lint]`
- Anything this project intentionally does differently from generic React best
  practice, and why: `[document the why — arbitrary-looking rules get silently
  "corrected" by a model unless the reasoning is stated]`

---

## Keeping this current

Review this file when the styling approach changes, the design system gets a major
version bump, or Code Connect mappings are added/removed. A stale file is worse than no
file — it produces confidently wrong assumptions instead of an explicit question.
