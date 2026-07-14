# figma-to-swiftui — known failure signatures

v2 — this skill inherits every signature in
`figma-grounding/references/failure-modes.md`; that table isn't repeated here. This
file covers failures specific to the SwiftUI codegen layer, updated with patterns
reported by real teams running comparable design-to-code workflows (Halodoc's public
FigmaToSwiftUI case study, monday.com's design-system-MCP writeup, Figma's own
"Implement Design" skill Q&A).

## Silent quality failures

| Signature | Cause | What it means |
|---|---|---|
| Generated code calls a component (`PrimaryButton`, `DSCard`, `AppTextField`) that doesn't exist anywhere in the target project | Model pattern-matched a plausible-sounding DS component name instead of confirming against the real catalogue / Code Connect map / an actual codebase search | Treat any component name not confirmed via `get_code_connect_map` or a real `grep` of the codebase as invented until proven otherwise — the single most commonly reported failure across every team writeup we found |
| Component search finds *a* valid pattern in the codebase, but not the one the design system actually recommends | Large/legacy codebases accumulate multiple ways of doing the same thing; agentic search can surface an older or non-canonical pattern that still "works" | Prefer whatever the project's context file names as canonical over whatever a codebase search turns up first — if they conflict, surface the conflict rather than picking silently |
| Generated `View`/`ViewModel` uses a state-management pattern the project doesn't actually use | Pre-flight was skipped, or no context file was found and the model defaulted to a commonly-seen pattern instead of asking or generating one | If pre-flight didn't run, treat any generated architecture choice as unverified regardless of how idiomatic it looks |
| Generated ViewModel doesn't compile under the project's actual Swift concurrency mode | Concurrency mode wasn't confirmed in pre-flight; code was written assuming Swift 5-style patterns in a Swift 6 strict-concurrency project (or vice versa) | Concurrency mode is a pre-flight-blocking question, not a style preference — see `references/concurrency-and-modern-swift.md` |
| A literal width/height from the Figma canvas ends up hardcoded in `.frame()` | Sizing mode (hug/fill/fixed) was grounded but ignored during AutoLayout translation | Cross-check every `.frame(width:height:)` in generated output against the grounded sizing mode |
| Image reference compiles-looking (`Image("product_card_bg")`) but the asset was never actually fetched or confirmed to exist | Asset was *named* based on the Figma layer, but no download tool was available/attempted in this session, and that gap wasn't stated | Always state explicitly which images still need manual export vs. which were actually fetched — see `references/design-rationale.md` on why this isn't promised as fully automatic |
| Localization keys added to the wrong file, or skipped entirely | Project context file didn't list the target module's path, and the gap wasn't surfaced before writing keys | If the path wasn't confirmed, the skill should have stopped and asked |
| Component variant silently mapped to the "closest" defined case | A Figma variant combination has no matching Code Connect definition; nearest case picked instead of surfacing the gap | Same discipline as `figma-grounding`'s "never treat proximity as a match" |
| **Code presented as done without ever being checked against the screenshot** | Phase C (validation) was skipped, or treated as implicit rather than an explicit pass | This is the highest-severity failure in this file — "compiles and looks plausible" and "matches the design" are different claims; a model has no innate visibility into its own rendered output, so equating the two is exactly the confident-guess failure this whole skill exists to prevent |

## Compile-time traps (loud, but easy to walk straight into)

| Signature | Cause | What it means |
|---|---|---|
| Generated `View` fails to compile with a redeclaration/conformance error on `body`, `id`, or another protocol-required member | A Figma layer literally named "Body," "Content," "Frame," or similar got mapped straight to a Swift property/method name without checking it against the type's existing members (including protocol-inherited ones) | Always check a Figma-derived identifier against every member already on that type before using it — this is a compile error, not a style nitpick, and it's specifically likely here because Figma layer names and common Swift/SwiftUI member names overlap more than generic variable-naming collisions would |
| Nested groups with visibly different spacing get flattened into one `VStack(spacing:)` with a single averaged value | Gaps were read as one number per container instead of per sibling-pair at every nesting level | Reproduce each distinct gap as its own stack's `spacing:`/`.padding()` — a compromise value is wrong for both the tight and the loose sections it was averaged from |
| A block that should be multiple styled `Text` spans renders as one flat-styled paragraph | Model treated a multi-style text run as a single string with one style, missing that different words/lines had different colors, weights, or opacity in the source | Use `Text` concatenation to preserve per-span styling instead of collapsing to the dominant or first-seen style |

## Screen-complexity failure mode

Very large or deeply-nested frames (many component variants, mixed scroll directions,
heavy nesting) can produce an MCP call that pulls too much context, a mapping table
that gets unreadably long, or generation that needs a second pass to actually compile.
If a single frame is clearly outgrowing one pass: split it into logical sections
(header, list, footer) and run the skill once per section rather than forcing one call
to cover the whole screen.

## Debugging order

1. Confirm `figma-grounding` actually ran and its values are being used, not re-guessed.
2. Confirm the project context file was found and read, or was generated via pre-flight
   — if skipped, every architecture/concurrency/component assumption downstream is
   suspect.
3. Check component names against the real codebase before trusting that a generated
   component call will compile.
4. Confirm Phase C (validation against the screenshot) actually happened and wasn't
   skipped as "obviously fine."
5. If a screen is complex, split it into sections rather than debugging one giant
   generation pass.
