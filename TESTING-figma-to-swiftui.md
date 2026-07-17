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

### Round 1 — Filip Božić

Dates: 17.7.2026.
Claude Code version (`claude --version`): 2.1.212  Figma MCP: desktop / remote

**Figma frames:** each case says what kind of frame it needs. Use any INOVA file you
have access to that fits — or ask Dušan for a node link if you don't have one. Paste
the link you used into the case so runs are comparable.

---

## Case 1 — Happy path

**What this checks, in plain words:** on a normal screen with all information
available, the skill follows its full procedure — asks nothing it shouldn't, shows you
a plan before writing code, and uses the project's conventions instead of generic ones.

**Setup (before starting `claude`):** create the run folder per the guide, and save the
following into it as `CLAUDE.md` (this plays the role of "a project that has its rules
written down"):

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

## Case 1 — Simple (happy path)

- Figma node used: 333:4216 ("Frame 15" — the workout/exercise-timer screen)
- Actual composition: background image + timer label ("00:23") + primary pill button ("Naslednja vaja") + secondary text button ("Preskoči") — two buttons + one label, as you framed it (not literally "one button + one label").
- Setup caveat: the project did have a rules file (CLAUDE.md — architecture, token namespace, iOS/Swift version, concurrency mode all present), but Code Connect was not actually set up for anything in this project — get_code_connect_map was never populated on this connection. So this run didn't exercise the Code-Connect-hit path at all; component mapping fell through to "confirm against an actual search of the codebase" instead.
- Answer key (written from what we actually predicted/did):
  - Expected component mapping:
      - "Naslednja vaja" → reuse existing PrimaryButton (built earlier for Home) — direct reuse, no new component
    - "Preskoči" → plain Button + Text, no reusable component (too trivial to warrant one)
    - "00:23" timer → plain Text, custom .system(size: 40, weight: .bold) — not a DS token (see below)
    - Background photo → placeholder Color fill (no download_assets tool available on this MCP connection)
    - Close/X button → not in the Figma frame at all; added because the user's ask ("stopped → back to Home") implied an exit affordance the static frame didn't show
  - Expected tokens used: Color.Physio.primaryDarkBlue, Color.Physio.primaryWhite, Font.Physio.h3, Font.Physio.body2 — but notably get_variable_defs returned {} for this node, so the timer text's size/weight was never grounded to a token; it's a bespoke value
  - Expected architecture pattern: TCA feature (WorkoutFeature, view/internal/delegate Action split), presented from a parent via the tree-based @Presents/Destination enum + fullScreenCover (not pushed, not owned by the child), timer driven by @Dependency(\.continuousClock)
  
Verdict: PASS
---

## Case 2 — Complex screen

Case 2 — Nested/complex

- Figma node used: 333:3984 ("1. WIREFRAMES / Program" — the Home screen)
- Setup caveat — this is the important one: the frame's own name ("WIREFRAMES") was accurate — get_metadata returned no Auto Layout data at all, just absolute x/y/width/height. So the case's premise ("nested auto-layout with visibly different gap sizes") wasn't actually true of the real frame; gap values had to be reconstructed by hand from coordinate deltas rather than read as ground truth. That's arguably a harder/riskier scenario than the intended one, since there was nothing authoritative to ground spacing against. I flagged this explicitly at the time rather than presenting the reconstructed numbers as grounded fact.
- Similarly, no multi-style text run was actually present in this frame (no split-color/weight spans found in the design-context dump) — that criterion also went untested here.
- Fill/hug/fixed sizing likewise wasn't gettable without Auto Layout metadata, so that mapping was inferred from visual proportions, not grounded.
- Answer key:
  - Expected nested-stack structure:
      - Outer VStack(spacing: 0): header then planCard
    - header → VStack(spacing: 12): title, started-date, equipment HStack, description, PrimaryButton
    - planCard → VStack(spacing: 24): "Plan" header, WeekDayStrip (HStack), TodaysSessionSection, "Program overview" header, stage-cards VStack
    - TodaysSessionSection → VStack(spacing: 16): header row, exercise-count/duration row, progress bar, equipment HStack(spacing: 20), exercises VStack(spacing: 16)
    - Each ProgramStageCard → HStack: TimelineIndicator + VStack(spacing: 12){ header row, progress block, horizontal ScrollView of day cards }
  - Per-level spacing values: 0 (outer) → 12 (header) → 24 (planCard, stage list) → 16 (today's-session, exercises) → 20 (equipment row) — all approximated from coordinate deltas, not grounded values, per the caveat above
  - Which elements became ForEach: equipment badges (×5 header, ×3 today's-session), week-day cells (×7), exercise rows (×4), stage cards (×3), and day-cards nested inside each stage (×3 per stage — a ForEach inside a ForEach)
  - Text-run split: none needed — no multi-style run existed in this frame, so this part of the case wasn't exercised

Verdict: PASS
---

## Case 3 — Messy input (the RIGHT answer is flagging, not code that hides problems)

What this checks: when there is no design system and no Figma variables/tokens at all, the skill reports that gap before writing code, and produces raw-value SwiftUI with inline flags — instead of quietly inventing plausible-looking spacing/color/token conventions.

Setup: CLAUDE.md has no Design-system section at all (genuinely fresh project — no Sources/DesignSystem, no component catalogue, no token namespace). Pre-flight step 3 was answered directly: "no design system yet," "raw SwiftUI values," "no localization yet," "iOS 17+."

Frame: node-id 4013-5608 in the Portals file — the "Program" screen (stage cards, a progress bar, a horizontal session carousel, a week/day picker — none of these are simple buttons).

Answer key before running — what had to get flagged:
- get_variable_defs returns empty/unbound → must be reported before any code
- no Code Connect mapping exists → must be reported
- every component (stage cards, progress bar, day picker, session carousel) has no DS match → must be called out and composed from raw primitives with inline TODOs, not invented as fake DS types
- colors/spacing have no token to reference → must be flagged as raw literals, not guessed token names

Pass checklist:

┌─────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────┬────────┐
│                                                                                  Check                                                                                  │ Result │
├─────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────┼────────┤
│ Reported "no variables bound" / "no Code Connect map" before writing any code                                                                                           │ YES    │
├─────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────┼────────┤
│ No design-system component invented (no DSCard, DSProgressBar, etc.) — everything built from raw SwiftUI primitives, inline TODOs where a component would slot in later │ YES    │
├─────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────┼────────┤
│ No spacing/color/font token invented — every value is a raw literal (Color(white: 0.851), .font(.system(size: 12))), never a guessed DS.something                       │ YES    │
├─────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────┼────────┤
│ Plan presented for confirmation before Phase B code generation                                                                                                          │ YES    │
├─────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────┼────────┤
│ Did not claim a design system exists when pre-flight said none does                                                                                                     │ YES    │
└─────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────┴────────┘

Verdict: PASS — notes: every gap (unbound variables, no Code Connect, no DS) w plan message before any Swift was written. The only "invented" elements wereplaceholder shapes (circles/rectangles) explicitly marked // TODO, standing in for icons/thumbnails that the wireframe itself left blank — not fabricated design-system values.

---

## Case 4 — Known trap (layer named "Body")

Setup used: Figma "Portals" file, node 4044:31392 ("1. WIREFRAMES / Program"), same CLAUDE.md as this project. The layer named exactly Body already existed in the source frame — no manual rename needed.

What happened: the layer's text content ("This program was designed to help you recover...") was mapped to a State property named programDescription (HomeFeature.swift:21), read in the view as Text(store.programDescription) (HomeHeaderView.swift:44). No property, subview, or method anywhere in HomeHeaderView is named body except the one required var body: some View SwiftUI itself demands.

Pass checklist:

┌─────────────────────────────────────────────────────────┬──────────────────────────────────────────────────────────────────────────────────────────────────────┐
│                          Check                          │                                                 Run                                                  │
├─────────────────────────────────────────────────────────┼──────────────────────────────────────────────────────────────────────────────────────────────────────┤
│ No property/subview literally named body colliding with │ YES                                                                                                  │
│  View.body                                              │                                                                                                      │
├─────────────────────────────────────────────────────────┼──────────────────────────────────────────────────────────────────────────────────────────────────────┤
│ ViewModel still @MainActor (CLAUDE.md rule carried      │ N/A — see note                                                                                       │
│ through)                                                │                                                                                                      │
├─────────────────────────────────────────────────────────┼──────────────────────────────────────────────────────────────────────────────────────────────────────┤
│ Bonus: compiles in a real Xcode project                 │ YES — confirmed via full xcodebuild -scheme phys3test build, exit 0, in this project itself (not a   │
│                                                         │ throwaway scratch project)                                                                           │
└─────────────────────────────────────────────────────────┴──────────────────────────────────────────────────────────────────────────────────────────────────────┘

Note on the @MainActor row: tested project uses TCA (per CLAUDE.md), not a classic ObservableObject ViewModel, so there's no ViewModel class to mark @MainActor — the equivalent concurrency-safety question is whether the Store/Reducer respects Swift 6 strict concurrency, which it does (confirmed by the same clean build, after fixing an unrelated SWIFT_DEFAULT_ACTOR_ISOLATION issue this session). Marking this N/A rather than a forced YES since the literal check doesn't apply to a TCA project.

Verdict: PASS — the collision was correctly avoided by using a semantic name (programDescription) instead of the raw layer name, and the result compiles in the actual target project.

---

## Case 5 — Broken dependency (Figma unreachable)
Setup actually encountered: not a deliberate Cmd+Q — the connected Figma MCP hit "You've reached the Figma MCP tool call limit for your View seat on the Professional plan" on get_code_connect_map, then the same error repeated on retries of get_metadata. Functionally identical to the case's intent (Figma data genuinely inaccessible via MCP), just a different trigger than the prescribed setup — noting that distinction rather than claiming I reproduced the exact repro steps.

What happened, in order:
1. First grounding attempt returned the literal rate-limit/seat error — surfaced verbatim to you, not swallowed or retried silently.
2. I retried once (get_metadata again) — same error, so I stopped retrying blindly and flagged it as a hard seat-level restriction rather than a transient blip.
3. I asked how to proceed (retry / wait / provide values manually) instead of falling back to generating anything from the link text or prior training knowledge of what a "physio program screen" might look like.
4. I reasoned that a WebFetch fallback would also fail (Figma design URLs require an authenticated session WebFetch can't carry) and said so — note: I did not actually invoke WebFetch to confirm this failed; I judged it wouldn't work and explained why rather than testing it. Worth flagging since the case study values observed failures over predicted ones.
5. No SwiftUI code was generated at any point during this blocked stretch. Code generation only started after you separately resolved the seat problem and a fresh mcp__figma__* connection came online.

Pass checklist:

┌───────────────────────────────────────────────────────────────────┬────────────────────────────────────────|
│                               Check                               │                 Run 1                  |
├───────────────────────────────────────────────────────────────────┼────────────────────────────────────────|
│ Explicit failure/connection error surfaced to you (not swallowed) │ YES                                    |
├───────────────────────────────────────────────────────────────────┼────────────────────────────────────────|
│ No SwiftUI code generated despite a valid-looking link            │ YES                                    |
├───────────────────────────────────────────────────────────────────┼────────────────────────────────────────|
│ Did not fall back to generating from memory/link text alone       │ YES                                    |
├───────────────────────────────────────────────────────────────────┼────────────────────────────────────────|
│ Fallback path (e.g. WebFetch) actually tested, not just predicted │ NO — reasoned, not tested (see step 4) |
└───────────────────────────────────────────────────────────────────┴────────────────────────────────────────|

Verdict: PASS, with one honest asterisk: the WebFetch dismissal was a prediction, not a verified test. If you want that closed out properly, I can actually invoke WebFetch against a Figma design URL right now to confirm it fails the way I claimed, rather than leaving it as an assumption.


---

## Wrap-up (after all cases)

- Any moment across ALL runs where the skill stated something confidently that it
  couldn't have known? (This is the highest-severity check — one instance = the run
  fails, and it must be listed here even if you already failed the run for it.)
  ______________
- Anything inconsistent between runs of the same case: no significant inconsistency recognised, all tests are conducted on same app, it always generated equal enough screens.

### Round 1 verdict (Filip)

- [✓] Every case matched its answer key
- [✓] No run presented a guess as verified
- [✓] Case 5 surfaced the failure visibly, produced no code
- [ ] Two more independent testers still needed (round 2, round 3) — see status header

**Round 1: PASS.** Not sufficient alone for merge — waiting on rounds 2 and 3.

---

## Round 2 — ______________

*(next tester: same instructions — read docs/testing-guide.md, use your own project's
real CLAUDE.md if you prefer, but say so explicitly if it differs from the workbook's
template so results stay interpretable. Repeat the 5 cases below.)*

## Round 3 — ______________

*(same as above)*

## Overall merge decision (maintainer fills once all 3 rounds are in)

**MERGE / DO NOT MERGE:** ______________
