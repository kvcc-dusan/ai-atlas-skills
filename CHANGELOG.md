# Changelog

All notable changes to skills in this repo are documented here, per skill, following [Keep a Changelog](https://keepachangelog.com/en/1.1.0/) format. Each skill is versioned independently (semver).

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
