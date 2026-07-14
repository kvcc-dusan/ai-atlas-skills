# Code Connect for SwiftUI — annotation cheat sheet

Reference for reading/writing Code Connect mappings when `get_code_connect_map` returns
a hit. Consult this before assuming how a Figma property should resolve to a Swift
component call.

## Requirements

Code Connect must be added as a package dependency for SwiftUI mappings to resolve at
all:

```swift
dependencies: [
    .package(url: "https://github.com/figma/code-connect", from: "1.0.0"),
],
targets: [
    .target(
        name: "ExampleTarget",
        dependencies: [.product(name: "Figma", package: "code-connect")]
    )
]
```

If `get_code_connect_map` returns empty for a component you know is mapped, this is one
possible cause — confirm the package is actually present before concluding "no mapping
exists."

## Property wrapper reference

| Wrapper | Maps | Example |
|---|---|---|
| `@FigmaString` | A Figma text property to a `String` | `@FigmaString("Text Content") var label: String` |
| `@FigmaBoolean` | A Figma boolean property to a `Bool` | `@FigmaBoolean("Disabled") var disabled: Bool` |
| `@FigmaEnum` | A Figma variant property to a Swift enum/value | `@FigmaEnum("Variant", mapping: ["Primary": .primary, "Secondary": .secondary])` |
| `@FigmaInstance` | A nested instance-swap property to a child component | `@FigmaInstance("Icon") var icon: Icon` |
| `@FigmaChildren` | Nested instances **not** bound to an instance-swap prop — takes the *layer name*, not a Figma prop name | `@FigmaChildren(layers: ["Header", "Row"]) var contents: AnyView?` |

`@FigmaEnum` and `@FigmaBoolean` also support `hideDefault: true` — suppress a modifier
entirely when the mapped value equals its default, rather than emitting a no-op call.

## Variant mapping (one Figma component → multiple Swift components)

When a single Figma component (e.g. `Button` with a `Type` variant) is actually
represented by separate Swift types (`PrimaryButton`, `SecondaryButton`,
`DangerButton`), map each Swift type to a specific variant combination rather than
trying to force one connection to cover all of them:

```swift
struct PrimaryButton_connection: FigmaConnect {
    let component = PrimaryButton.self
    let variant = ["Type": "Primary"]
    let figmaNodeUrl: String = "https://..."
    var body: some View { PrimaryButton(title: "Text") }
}
```

If `get_code_connect_map` hands back a node with a variant combination that has no
connection defined for it, that's a gap — surface it in Step 2b of the main skill, don't
fall back to the nearest-defined variant silently.

## Conditional modifiers

`figmaApply`/`elseApply` map a Figma property to a modifier chain rather than an
initializer argument:

```swift
@FigmaEnum("Type", mapping: ["Primary": true])
var isPrimary: Bool = false

var body: some View {
    MyComponent()
        .figmaApply(isPrimary) { $0.tint(.blue) } elseApply: { $0.backgroundColor(.clear) }
}
```

If the generated Code Connect snippet you're given uses this pattern, reproduce the
modifier chain as-is rather than converting it back into initializer arguments — the
Code Connect author chose modifiers deliberately, usually because the underlying
component's API requires it.

## What this does NOT solve

Code Connect resolves *component identity and prop mapping* — it does not resolve
spacing/color values (that's `get_variable_defs`, per `figma-grounding`) and it does not
tell you about interaction/behavior (that's the Assumptions step). Treat it as one input
among several, not a complete substitute for grounding.
