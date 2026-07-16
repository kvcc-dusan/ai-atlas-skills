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

Status: **IN TESTING**

Tester: ______________  Dates: ______________
Claude Code version (`claude --version`): ______  Figma MCP: desktop / remote

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

**Pick a frame:** something small and simple — a card, a button row, one section of a
screen. Not a whole page.

**Prompt to paste** (replace the link):

```
/figmaswiftui https://www.figma.com/design/...?node-id=...
```

**Answer key — fill this BEFORE running** (what should the result contain for YOUR
chosen frame? e.g. "a VStack with the image on top, title, subtitle; one View file +
one ViewModel; @MainActor"):

______________  (all runs of this case reuse the same key)

**Pass checklist (YES/NO per run):**

| Check | Run 1 | Run 2 | Run 3 |
|---|---|---|---|
| It read `CLAUDE.md` and did NOT re-ask things the file already answers (architecture, concurrency, tokens) | | | |
| It pulled real Figma data (you saw get_metadata / get_variable_defs tool calls) BEFORE writing values | | | |
| It showed a plan/mapping table and waited for your OK BEFORE writing Swift | | | |
| Generated ViewModel is `@MainActor` (the CLAUDE.md demands it) | | | |
| Spacing/sizes in code came from the pulled data, not round guesses (spot-check 2–3 values against Figma's Dev Mode inspect) | | | |
| No component names invented (this project has no library — everything should be plain SwiftUI) | | | |
| It never stated a value/fact confidently that it didn't pull (golden rule 4) | | | |

Verdict: PASS / FAIL — notes: ______________

---

## Case 2 — Complex screen

**What this checks:** on a realistic screen, the structure survives — repeated elements
become a loop, different gap sizes stay different, and mixed text styling isn't
flattened.

**Setup:** same `CLAUDE.md` as Case 1 (fresh folder, paste it again).

**Pick a frame:** a real screen with (a) at least one visibly repeated element (list
rows, cards), (b) visibly different spacing in different areas (tight inside a group,
looser between groups), and (c) if you can find one, a text block where different words
have different colors/weights.

**Prompt:** same as Case 1, with your link.

**Answer key BEFORE running** (for your frame: which element must become a ForEach?
roughly which areas have which spacing? which text is multi-styled?): ______________

**Pass checklist (YES/NO per run):**

| Check | Run 1 | Run 2 | Run 3 |
|---|---|---|---|
| Repeated element → its own subview + ForEach/List (NOT copy-pasted N times) | | | |
| Different gaps preserved as different values (NOT one averaged spacing everywhere) | | | |
| Multi-style text kept as differently-styled parts (NOT one flat style) — skip if your frame has none | | | |
| Plan table shown and confirmed before code | | | |
| Elements that fill the width use flexible sizing, not a hardcoded number like 390 | | | |
| No confident un-pulled facts | | | |

Verdict: PASS / FAIL — notes: ______________

---

## Case 3 — Messy input (the RIGHT answer is flagging, not code that hides problems)

**What this checks:** when information is missing, the skill says "I don't know / this
didn't resolve" instead of inventing something plausible. **A run where it produces
clean-looking code with no flags is a FAIL here.**

**Setup:** same `CLAUDE.md`, but CHANGE the Design system section to:

```markdown
## Design system
- Component library location: Sources/DesignSystem
- Is Code Connect set up for this library? no
- Component catalogue: DSButton — DSButton(title:style:action:). Nothing else exists.
- Token namespace: DS.Spacing (only .small=4, .medium=8, .large=16 exist)
```

(Note: `Sources/DesignSystem` doesn't actually exist in your folder — that's part of
the test.)

**Pick a frame:** anything with a component that is clearly NOT just a button (a
custom chart, a fancy card, a segmented control...) and at least one color.

**Prompt:** same as Case 1, with your link.

**Answer key BEFORE running:** list what MUST get flagged for your frame (e.g. "the
chart has no DS match → it must stop and ask me; colors have no tokens in DS.Spacing →
raw values must be flagged"): ______________

**Pass checklist (YES/NO per run):**

| Check | Run 1 | Run 2 | Run 3 |
|---|---|---|---|
| The no-match component was surfaced — it asked you what to do (compose / TODO / new component), did NOT invent `DSChart` or similar | | | |
| Values with no matching token were flagged (a one-line "no token for X, using raw value") — not silently hardcoded | | | |
| It did NOT claim to have found `Sources/DesignSystem` (it doesn't exist) | | | |
| Nearest-match cheating absent: nothing "close enough" was silently promoted to a token | | | |

Verdict: PASS / FAIL — notes: ______________

---

## Case 4 — Known trap (layer named "Body")

**What this checks:** a Figma layer literally named "Body" or "Content" must not
become a Swift property that collides with SwiftUI's `body` (that's a compile error
the skill specifically promises to avoid).

**Setup:** same `CLAUDE.md` as Case 1. In Figma, take any small frame you can edit and
rename one of its layers to exactly `Body` (ask Dušan to prep one if you can't edit).

**Prompt:** same as Case 1, with your link.

**Answer key:** generated View must compile-safely name that element something other
than `body` (and say so, or just do it) — no redeclaration of `body`.

**Pass checklist (YES/NO per run):**

| Check | Run 1 | Run 2 | Run 3 |
|---|---|---|---|
| No property/subview literally named `body` colliding with `View.body` | | | |
| ViewModel still `@MainActor` (CLAUDE.md rule carried through) | | | |
| Bonus if you have Xcode: paste the file into a scratch iOS project — does it compile? (skip if no Xcode; note "skipped") | | | |

Verdict: PASS / FAIL — notes: ______________

---

## Case 5 — Broken dependency (Figma unreachable)

**What this checks:** with no Figma data available, the skill must say so and stop —
NOT generate SwiftUI from memory or from the link text alone. **Code output = FAIL.**

**Setup:** same `CLAUDE.md`. Then break the Figma connection — easiest way:
**fully quit the Figma desktop app** (Cmd+Q, check it's not in the dock) before
starting the `claude` session. (If you're on the remote MCP instead, ask Dušan to
temporarily point you at a file your account can't access — same effect.)

**Prompt:** same as Case 1, with a valid-looking link.

**Answer key:** an explicit failure message (connection/access error surfaced to you)
and no generated screen code.

**Pass checklist (YES/NO per run — 2 runs are enough here):**

| Check | Run 1 | Run 2 |
|---|---|---|
| It told you clearly the Figma data couldn't be fetched | | |
| It did NOT produce SwiftUI code anyway | | |
| It did NOT pretend a screenshot/memory was good enough | | |

Verdict: PASS / FAIL — notes: ______________

---

## Wrap-up (after all cases)

- Any moment across ALL runs where the skill stated something confidently that it
  couldn't have known? (This is the highest-severity check — one instance = the run
  fails, and it must be listed here even if you already failed the run for it.)
  ______________
- Anything inconsistent between runs of the same case: ______________

### Final verdict

- [ ] Every case matched its answer key on repeated runs
- [ ] No run presented a guess as verified
- [ ] Case 5 surfaced the failure visibly, produced no code
- [ ] (Maintainer fills) external review logged

**MERGE / DO NOT MERGE:** ______________
