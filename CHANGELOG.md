# Changelog

All notable changes to skills in this repo are documented here, per skill, following [Keep a Changelog](https://keepachangelog.com/en/1.1.0/) format. Each skill is versioned independently (semver).

## [figma-to-react 1.0.0] - 2026-07-15 — self-tested, external review pending

### Added
- Initial release. Converts a grounded Figma frame into React + TypeScript components matched to the target project's real design system, Code Connect mappings, styling approach, and token representation — never generic JSX or invented components. Sibling of `figma-to-swiftui` (on its own branch), built on `figma-grounding`.
- Blocking pre-flight: MCP-server identity check (official vs. Framelink — different tool surface, no Code Connect), styling approach (Tailwind / CSS Modules / styled-components / vanilla — never assumed), Tailwind token-representation pin, framework flavor, breakpoint situation. Stated fallback default (React+TS+Tailwind+Vite) is labelled as a default, not a discovered fact.
- Three-phase procedure: ground+plan (mapping table confirmed before code), generate (AutoLayout→flexbox/grid, typed props, a11y, reserved-prop collision check), validate against the Figma screenshot per breakpoint frame.
- Responsive discipline: per-breakpoint frames grounded separately; single-frame designs get responsive behavior flagged as assumptions, never invented.
- references/: code-connect-react.md (template-files-first — the parser API is now Figma's "legacy" path — with the legacy React helpers documented for existing repos), failure-modes.md, responsive-and-styling.md, PROJECT_CONTEXT.template.md, validation-checklist.md, design-rationale.md.
- Companion command `/figmareact`.

Status: on branch `feat/figma-to-react`. Methodology steps 1–5 passed against a live Figma file (Portals, 5-case eval, answer key first, live broken-dependency case) — see [TESTING-figma-to-react.md](TESTING-figma-to-react.md). Step 6 (external review) open before merge to `main`.

## [figma-grounding 1.1.0] - 2026-07-14

### Added
- Layout-sizing grounding: hug/fill/fixed mode and axis alignment are now explicitly grounded values, pulled from get_metadata — two different sizing modes can render identically in a screenshot.
- Code Connect as a separate grounding concern (component *identity* vs. component *values*): get_code_connect_map / get_code_connect_suggestions added to the procedure.
- "Why this exists" section, remote-vs-desktop server selection caveat, call-scoping guidance (rate limits, per-screen grounding).
- failure-modes.md v2: merged with Figma's published Q&A troubleshooting docs — 500s, tools-not-loading, remote-server selection behavior, tool auto-routing mis-picks, Code Connect framework-scoping gaps, updated debugging order.

### Changed
- get_code references replaced with get_design_context (its successor), with a note on its React+Tailwind default output.

## [figma-grounding 1.0.0] - 2026-07-02

### Added
- Initial release. Forces get_metadata/get_variable_defs extraction before implementation, flags unresolved tokens instead of guessing.
