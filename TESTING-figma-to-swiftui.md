# figma-to-swiftui — testing workbook

**Read [docs/testing-guide.md](docs/testing-guide.md) first** — it covers setup, the
golden rules (fresh folder + fresh session for EVERY run — this is mandatory, results
from reused sessions are invalid), and how to record results. This file gives you the
five cases for this specific skill: what to set up, what to paste, and what to check.

**This skill's install** (per the guide's setup step, from branch `feat/figma-to-swiftui`):

```
cp -r skills/figma-grounding ~/.claude/skills/figma-grounding
cp commands/scrapedesign.md ~/.claude/commands/scrapedesign.md
cp -r skills/figma-to-swiftui ~/.claude/skills/figma-to-swiftui
cp commands/figmaswiftui.md ~/.claude/commands/figmaswiftui.md
```

Verify: fresh `claude` session, type `/` → `/figmaswiftui` appears.

Status: **IN TESTING — round 1 of 3 complete**

This skill is being tested across three independent testers (each running the full
5-case set once) rather than one tester repeating each case — see
[docs/testing-guide.md](docs/testing-guide.md) and step 6 of
[docs/testing-methodology.md](docs/testing-methodology.md). Round 1 below is Filip's
pass. Two more rounds, by two other developers, are needed before this merges to
`main`.

**Figma frames:** each case says what kind of frame it needs. Use any INOVA file you
have access to that fits — or ask Dušan for a node link if you don't have one. Paste
the link you used into the case so runs are comparable.

---

## Round 1 — Filip Božić

Dates: 17.7.2026.
Claude Code version (`claude --version`): 2.1.212  Figma MCP: desktop / remote

**CLAUDE.md used for this round:**

```markdown
# Project context — Figma → SwiftUI

## Architecture
- State-management / architecture pattern: plain MVVM
- View/ViewModel file convention: ScreenNameView.swift + ScreenNameViewModel.swift

## Swift & concurrency
- Minimum iOS / Swift version: iOS 17+, Swift 6
- Swift 6 strict concurrency enabled? yes — ViewModels must be @MainActor

## Design system
- Component library location: none — this test project has no component library
- Is Code Connect set up for this library? no
- Token namespace: none — raw SwiftUI values are fine, no token mapping expected
- What must never be hardcoded: nothing for this test

## Localization
- none — plain strings are fine

## Accessibility & quality bar
- SwiftLint config location: not used
```

### Case 1 — Happy path

- **Figma node used:** `333:4216` ("Frame 15" — the workout/exercise-timer screen)
- **Actual composition:** background image + timer label ("00:23") + primary pill
  button ("Naslednja vaja") + secondary text button ("Preskoči") — two buttons + one
  label, not literally "one button + one label" as originally framed.
- **Setup caveat:** the project did have a rules file (architecture, token namespace,
  iOS/Swift version, concurrency mode all present), but Code Connect was **not**
  actually set up for anything in this project — `get_code_connect_map` was never
  populated on this connection. So this run didn't exercise the Code-Connect-hit path;
  component mapping fell through to "confirm against an actual search of the codebase"
  instead.

**Answer key (written from what was actually predicted/done):**

| Element | Expected mapping |
|---|---|
| "Naslednja vaja" button | Reuse existing `PrimaryButton` (built earlier for Home) — direct reuse, no new component |
| "Preskoči" button | Plain `Button` + `Text`, no reusable component (too trivial to warrant one) |
| "00:23" timer | Plain `Text`, custom `.system(size: 40, weight: .bold)` — not a DS token (see below) |
| Background photo | Placeholder `Color` fill (no `download_assets` tool available on this MCP connection) |
| Close/X button | Not in the Figma frame at all — added because the user's ask ("stopped → back to Home") implied an exit affordance the static frame didn't show |

- **Expected tokens used:** `Color.Physio.primaryDarkBlue`, `Color.Physio.primaryWhite`,
  `Font.Physio.h3`, `Font.Physio.body2` — but notably `get_variable_defs` returned `{}`
  for this node, so the timer text's size/weight was never grounded to a token; it's a
  bespoke value.
- **Expected architecture pattern:** TCA feature (`WorkoutFeature`,
  view/internal/delegate `Action` split), presented from a parent via the tree-based
  `@Presents`/`Destination` enum + `fullScreenCover` (not pushed, not owned by the
  child), timer driven by `@Dependency(\.continuousClock)`.

**Verdict: PASS**

---

### Case 2 — Nested/complex

- **Figma node used:** `333:3984` ("1. WIREFRAMES / Program" — the Home screen)
- **Setup caveat (important):** the frame's own name ("WIREFRAMES") was accurate —
  `get_metadata` returned no Auto Layout data at all, just absolute x/y/width/height.
  So the case's premise ("nested auto-layout with visibly different gap sizes") wasn't
  actually true of the real frame; gap values had to be reconstructed by hand from
  coordinate deltas rather than read as ground truth. That's arguably a harder/riskier
  scenario than the intended one, since there was nothing authoritative to ground
  spacing against. This was flagged explicitly at the time rather than presenting the
  reconstructed numbers as grounded fact.
- No multi-style text run was actually present in this frame (no split-color/weight
  spans found in the design-context dump) — that criterion went untested here.
- Fill/hug/fixed sizing likewise wasn't gettable without Auto Layout metadata, so that
  mapping was inferred from visual proportions, not grounded.

**Answer key:**

- **Expected nested-stack structure:**
  - Outer `VStack(spacing: 0)`: header then planCard
  - `header` → `VStack(spacing: 12)`: title, started-date, equipment `HStack`,
    description, `PrimaryButton`
  - `planCard` → `VStack(spacing: 24)`: "Plan" header, `WeekDayStrip` (`HStack`),
    `TodaysSessionSection`, "Program overview" header, stage-cards `VStack`
  - `TodaysSessionSection` → `VStack(spacing: 16)`: header row,
    exercise-count/duration row, progress bar, equipment `HStack(spacing: 20)`,
    exercises `VStack(spacing: 16)`
  - Each `ProgramStageCard` → `HStack`: `TimelineIndicator` +
    `VStack(spacing: 12)` { header row, progress block, horizontal `ScrollView` of
    day cards }
- **Per-level spacing values:** 0 (outer) → 12 (header) → 24 (planCard, stage list) →
  16 (today's-session, exercises) → 20 (equipment row) — all approximated from
  coordinate deltas, not grounded values, per the caveat above.
- **Which elements became `ForEach`:** equipment badges (×5 header, ×3
  today's-session), week-day cells (×7), exercise rows (×4), stage cards (×3), and
  day-cards nested inside each stage (×3 per stage — a `ForEach` inside a `ForEach`).
- **Text-run split:** none needed — no multi-style run existed in this frame, so this
  part of the case wasn't exercised.

**Verdict: PASS**

---

### Case 3 — Messy input (the RIGHT answer is flagging, not code that hides problems)

**What this checks:** when there is no design system and no Figma variables/tokens at
all, the skill reports that gap before writing code, and produces raw-value SwiftUI
with inline flags — instead of quietly inventing plausible-looking spacing/color/token
conventions.

**Setup:** `CLAUDE.md` has no Design-system section at all (genuinely fresh project —
no `Sources/DesignSystem`, no component catalogue, no token namespace). Pre-flight
step 3 was answered directly: "no design system yet," "raw SwiftUI values," "no
localization yet," "iOS 17+."

**Frame:** node-id `4013:5608` in the Portals file — the "Program" screen (stage cards,
a progress bar, a horizontal session carousel, a week/day picker — none of these are
simple buttons).

**Answer key before running — what had to get flagged:**

- `get_variable_defs` returns empty/unbound → must be reported before any code
- No Code Connect mapping exists → must be reported
- Every component (stage cards, progress bar, day picker, session carousel) has no DS
  match → must be called out and composed from raw primitives with inline TODOs, not
  invented as fake DS types
- Colors/spacing have no token to reference → must be flagged as raw literals, not
  guessed token names

**Pass checklist:**

| Check | Result |
|---|---|
| Reported "no variables bound" / "no Code Connect map" before writing any code | YES |
| No design-system component invented (no `DSCard`, `DSProgressBar`, etc.) — everything built from raw SwiftUI primitives, inline TODOs where a component would slot in later | YES |
| No spacing/color/font token invented — every value is a raw literal (`Color(white: 0.851)`, `.font(.system(size: 12))`), never a guessed `DS.something` | YES |
| Plan presented for confirmation before Phase B code generation | YES |
| Did not claim a design system exists when pre-flight said none does | YES |

**Verdict: PASS** — every gap (unbound variables, no Code Connect, no DS) was in the
plan message before any Swift was written. The only "invented" elements were
placeholder shapes (circles/rectangles) explicitly marked `// TODO`, standing in for
icons/thumbnails that the wireframe itself left blank — not fabricated design-system
values.

---

### Case 4 — Known trap (layer named "Body")

**Setup used:** Figma "Portals" file, node `4044:31392` ("1. WIREFRAMES / Program"),
same `CLAUDE.md` as this project. The layer named exactly `Body` already existed in the
source frame — no manual rename needed.

**What happened:** the layer's text content ("This program was designed to help you
recover...") was mapped to a `@State` property named `programDescription`
(`HomeFeature.swift:21`), read in the view as `Text(store.programDescription)`
(`HomeHeaderView.swift:44`). No property, subview, or method anywhere in
`HomeHeaderView` is named `body` except the one required `var body: some View`
SwiftUI itself demands.

**Pass checklist:**

| Check | Result |
|---|---|
| No property/subview literally named `body` colliding with `View.body` | YES |
| ViewModel still `@MainActor` (CLAUDE.md rule carried through) | N/A — see note below |
| Bonus: compiles in a real Xcode project | YES — confirmed via full `xcodebuild -scheme phys3test build`, exit 0, in this project itself (not a throwaway scratch project) |

**Note on the `@MainActor` row:** the tested project uses TCA (per `CLAUDE.md`), not a
classic `ObservableObject` ViewModel, so there's no ViewModel class to mark
`@MainActor`. The equivalent concurrency-safety question is whether the Store/Reducer
respects Swift 6 strict concurrency, which it does (confirmed by the same clean build,
after fixing an unrelated `SWIFT_DEFAULT_ACTOR_ISOLATION` issue this session). Marked
N/A rather than forced YES since the literal check doesn't apply to a TCA project.

**Verdict: PASS** — the collision was correctly avoided by using a semantic name
(`programDescription`) instead of the raw layer name, and the result compiles in the
actual target project.

---

### Case 5 — Broken dependency (Figma unreachable)

**Setup actually encountered:** not a deliberate Cmd+Q — the connected Figma MCP hit
"You've reached the Figma MCP tool call limit for your View seat on the Professional
plan" on `get_code_connect_map`, then the same error repeated on retries of
`get_metadata`. Functionally identical to the case's intent (Figma data genuinely
inaccessible via MCP), just a different trigger than the prescribed setup.

**What happened, in order:**

1. First grounding attempt returned the literal rate-limit/seat error — surfaced
   verbatim, not swallowed or retried silently.
2. Retried once (`get_metadata` again) — same error, so it stopped retrying blindly
   and flagged it as a hard seat-level restriction rather than a transient blip.
3. Asked how to proceed (retry / wait / provide values manually) instead of falling
   back to generating anything from the link text or prior training knowledge of what
   a "physio program screen" might look like.
4. Reasoned that a WebFetch fallback would also fail (Figma design URLs require an
   authenticated session WebFetch can't carry) and said so — **note:** WebFetch was
   not actually invoked to confirm this failed; it was judged unlikely to work and
   explained why, rather than tested. Worth flagging since the methodology values
   observed failures over predicted ones.
5. No SwiftUI code was generated at any point during this blocked stretch. Code
   generation only started after the seat problem was separately resolved and a fresh
   `mcp__figma__*` connection came online.

**Pass checklist:**

| Check | Result |
|---|---|
| Explicit failure/connection error surfaced (not swallowed) | YES |
| No SwiftUI code generated despite a valid-looking link | YES |
| Did not fall back to generating from memory/link text alone | YES |
| Fallback path (e.g. WebFetch) actually tested, not just predicted | NO — reasoned, not tested (see step 4) |

**Verdict: PASS, with one honest asterisk** — the WebFetch dismissal was a prediction,
not a verified test. Worth closing out properly in a later round by actually invoking
WebFetch against a Figma design URL to confirm it fails the way claimed, rather than
leaving it as an assumption.

---

### Round 1 wrap-up

- **Any moment across ALL runs where the skill stated something confidently that it
  couldn't have known?** None observed, per Filip. (Highest-severity check — one
  instance would fail the run even if everything else passed.)
- **Anything inconsistent between runs of the same case?** No significant
  inconsistency recognised — all tests were conducted on the same app and it always
  generated equal-enough screens.

**Round 1 verdict:**

- [x] Every case matched its answer key
- [x] No run presented a guess as verified
- [x] Case 5 surfaced the failure visibly, produced no code
- [ ] Two more independent testers still needed (round 2, round 3) — see status header

**Round 1: PASS.** Not sufficient alone for merge — waiting on rounds 2 and 3.

---

## Round 2 — ______________

*(Next tester: same instructions — read [docs/testing-guide.md](docs/testing-guide.md).
You may use your own project's real `CLAUDE.md` instead of the template above, but say
so explicitly if it differs, so results stay interpretable. Repeat the 5 cases above.)*

## Round 3 — ______________

*(Same as above.)*

---

## Overall merge decision (maintainer fills once all 3 rounds are in)

**MERGE / DO NOT MERGE:** ______________
