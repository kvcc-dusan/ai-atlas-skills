# Code Connect for React — reading and authoring reference

Consult this when `get_code_connect_map` returns a hit for a React target, or when the
narrower task is authoring mappings. Full docs: developers.figma.com/docs/code-connect/react/

## Status note (verified July 2026 — this changed since earlier research)

Figma now labels the `figma.connect()` **parser API** for React the *Legacy Integration
Guide*. The recommended way to author new Code Connect integrations is **template
files** — framework-agnostic TypeScript files that explicitly define the emitted code
(developers.figma.com/docs/code-connect/template-files/). Both remain supported:

- **Reading** a `get_code_connect_map` hit is identical either way — you get the real
  component name and prop mapping; use it exactly as given.
- **Authoring** new mappings: prefer template files for a fresh integration; use the
  legacy parser API only to match a client repo that already uses it. Don't mix formats
  within one repo without flagging it.

## Legacy React parser API (still the installed base in most client repos)

Everything under `@figma/code-connect/react`:

| Helper | Maps | Notes |
|---|---|---|
| `figma.connect(Component, url, {...})` | The component itself | Second signature exists for native elements |
| `figma.string("Label")` | Text property → `string` | |
| `figma.boolean("Disabled")` | Boolean prop/variant | Can map `true`/`false` to two different JSX outputs |
| `figma.enum("Variant", {...})` | Variant property → code value | Supports nested references |
| `figma.slot("Content")` | Composable freeform sub-area | |
| `figma.instance("Icon")` | Nested instance-swap prop | Generic typing supported |
| `figma.children("Row")` | Nested instances by **layer name**, not prop name | Wildcards: `figma.children('Icon*')` |
| `figma.nestedProps("Header", {...})` | Surface a nested instance's props at parent level | |
| `figma.textContent("Title")` | A child text layer's content | |
| `figma.className([...])` | Compose a class string from mapped properties | Filters out `undefined`/empty — the Tailwind-native helper |

**Icons** have three connection patterns: as JSX elements, as React components, or as
string IDs. When a parent needs a connected icon's resolved props rather than just
rendering it, use the `getProps`/`render` helpers.

**Variant → separate components:** when one Figma component (e.g. `Button` with a
`Type` variant) is several React components (`PrimaryButton`, `SecondaryButton`), write
one `figma.connect` per variant combination via the `variant` option — don't force one
connection to cover all of them. A variant combination with no connection defined is a
gap: surface it in the plan's gap list, never fall back to the nearest-defined variant.

## Reading discipline (applies regardless of authoring format)

- A `get_code_connect_map` hit is authoritative for **component identity and prop
  mapping** — reproduce the mapped call as given, including `figma.className`-composed
  class strings; the author chose that composition deliberately.
- An **empty** map on a component you believe is mapped may be a scoping issue:
  `clientFrameworks`/`clientLanguages` requested with the wrong labels (e.g. only
  `swiftui` when the mapping is under `react`), or the wrong MCP server entirely
  (Framelink has no Code Connect at all — see `failure-modes.md`). Confirm before
  concluding "no mapping exists."

## What this does NOT solve

Code Connect resolves *component identity and prop mapping* — not spacing/color values
(that's `get_variable_defs`, per `figma-grounding`), not interaction/behavior (that's
the Assumptions step), and not which styling approach the project uses (that's
pre-flight). One input among several, not a substitute for grounding.
