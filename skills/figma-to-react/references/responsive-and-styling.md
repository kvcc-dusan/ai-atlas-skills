# Responsive breakpoints & styling-token representation

Two ambiguity dimensions the SwiftUI sibling of this skill never had to deal with. Both
are project- or file-specific judgment calls that must be resolved explicitly — in
pre-flight (styling) or in Phase A step 2 (breakpoints) — never guessed per screen.

## Responsive breakpoints

A Figma frame is normally one fixed width. Tailwind prefixes (`sm:`, `md:`, `lg:`) or
media queries describe *several* widths. The gap between those two facts is where
invented behavior sneaks in — LogRocket's published worked example of this exact
workflow needed a manual correction on its first pass (an element that should have been
hidden on mobile wasn't). Decision table:

| Situation in the Figma file | What to do |
|---|---|
| Sibling frames per breakpoint (`Desktop` / `Tablet` / `Mobile`, or width-suffixed duplicates) | Ground **each frame separately** via `figma-grounding` — metadata, variables, sizing modes per frame. Derive the responsive classes from the grounded diff, never from eyeballing two screenshots. |
| One frame, and the user asked for responsive output | Implement the designed width; every other-viewport decision is an **assumption**, listed in step 6, phrased as a question where it matters ("should the sidebar collapse below `md`? Not designed — I stacked it, flag if wrong"). |
| One frame, no responsive ask | Implement as designed at that width; note "responsive behavior undesigned" once in assumptions and move on. |
| Frames exist but disagree with each other (same element, incompatible treatments) | Surface the conflict — don't pick silently. |

Breakpoint values themselves come from the project (Tailwind config `screens`, an
existing tokens file), not from Tailwind's defaults by reflex — check pre-flight.

## Token representation (the Tailwind ambiguity)

The same grounded color has at least three valid spellings in a Tailwind project:

| Spelling | Example | When it's canonical |
|---|---|---|
| Config-based token class | `bg-primary-500` | Project defines tokens in `tailwind.config.*` / `@theme` — the usual DS-driven answer |
| Arbitrary value | `bg-[#0047AB]` | Escape hatch; canonical almost nowhere as a steady state — acceptable only as an explicitly flagged "no token matched" |
| CSS custom property | `bg-[var(--color-primary)]` | Projects whose tokens live in CSS variables (theming, runtime switching, non-Tailwind consumers) |

Which one is correct is **entirely project-specific** and is pinned once in pre-flight /
the project context file — not re-decided per screen. Rules of application:

- A grounded variable name (`color/primary/500` from `get_variable_defs`) maps to the
  project's token in its canonical representation. The variable *name* is the bridge —
  match on it, not on the resolved hex.
- A grounded raw value with **no matching token** never gets silently promoted to the
  nearest token (proximity is not a match — `figma-grounding`'s rule) and never gets
  silently hardcoded: emit the exact value in the escape-hatch spelling with an inline
  `// TODO: no token for #XXXXXX` (or the non-Tailwind equivalent).
- Non-Tailwind projects have the same three-way question in different clothes (CSS
  Modules value vs. `var(--token)` vs. a styled-components theme key) — the pre-flight
  answer covers it identically.

## Non-Tailwind styling approaches

The AutoLayout table in SKILL.md shows Tailwind utilities as shorthand for CSS
properties. Translation is mechanical (`flex flex-col gap-4 p-6` ≡
`display:flex; flex-direction:column; gap:…; padding:…`) — what is *not* mechanical is
where those declarations live (module file, styled component, global sheet) and how
tokens are referenced there. Both come from pre-flight. Never emit a second styling
system alongside the project's existing one "just for this screen."
