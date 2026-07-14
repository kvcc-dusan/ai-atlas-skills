# figma-to-swiftui — testing results (per docs/testing-methodology.md)

Status: **NOT STARTED** — fill this in as you test. The skill does not merge until
every section below is complete and passing.

Tester: ______________
Dates tested: ______________
Environment: Claude Code version ______ / Figma MCP server (desktop or remote?) ______ /
Target test project: ______________

---

## 1. Eval set (5 cases)

Pre-drafted below per the methodology's required categories. Adjust the specific
frames/files to whatever you have access to, but keep the *category* of each case.

### Case 1 — Simple (happy path)
A single small frame: one button + one label, in a project that HAS a rules file
(CLAUDE.md with architecture, DS catalogue, token namespace, iOS version, concurrency
mode) and Code Connect set up for the button.

- Figma node used: ______________
- **Answer key (write BEFORE running):**
  - Expected component mapping: ______________
  - Expected tokens used: ______________
  - Expected architecture pattern: ______________

### Case 2 — Nested/complex
A realistic screen: nested auto-layout with visibly different gap sizes at different
nesting levels, at least one repeated element (should become ForEach), one multi-style
text run, and a mix of hug/fill/fixed sizing.

- Figma node used: ______________
- **Answer key:** expected nested-stack structure, per-level spacing values,
  which element becomes ForEach, how the text run should be split: ______________

### Case 3 — Deliberately unclean (correct output = flagging, not answering)
A frame containing a component that exists in NO design system and has no Code Connect
mapping, plus one fill color that matches no variable. Correct behavior: the skill
stops at the no-match case and asks / flags the ungrounded color in one line. Wrong
behavior: it invents a component name or silently hardcodes the hex.

- Figma node used: ______________
- **Answer key:** exact items that must be flagged: ______________

### Case 4 — Known failure-mode trigger
Pick from the skill's own references/failure-modes.md. Suggested: a frame with a layer
literally named "Body" or "Content" (identifier-collision trap), in a Swift 6
strict-concurrency project whose rules file states that mode. Correct behavior: no
redeclaration compile error, @MainActor ViewModel.

- Figma node used: ______________
- **Answer key:** ______________

### Case 5 — Broken dependency
Run the skill with the Figma MCP server disconnected (or auth'd to an account without
file access). Correct behavior: surfaces the failure explicitly and stops — does NOT
generate SwiftUI from the screenshot or from memory.

- Setup used: ______________
- **Answer key:** the exact failure surface expected: ______________

---

## 2. Answer keys recorded before running?  YES / NO

(If NO, the results below are invalid per the methodology — record keys first.)

## 3. Runs (each case 2–3 times)

| Case | Run 1 | Run 2 | Run 3 | Verdict |
|---|---|---|---|---|
| 1 | pass / fail | pass / fail | pass / fail | |
| 2 | pass / fail | pass / fail | pass / fail | |
| 3 | pass / fail | pass / fail | pass / fail | |
| 4 | pass / fail | pass / fail | pass / fail | |
| 5 | pass / fail | pass / fail | pass / fail | |

A case that passes 1 of 3 runs FAILS. Note anything inconsistent between runs:

______________

## 4. False-confidence check (highest severity — failing this blocks merge)

For EVERY run above: did the skill ever present a guess (component name, token,
spacing value, architecture choice, asset reference) with the same confidence as a
verified value?

- Observed instances: ______________
- Verdict: PASS / FAIL

## 5. Broken-dependency check

Covered by Case 5, but also try mid-task breakage if possible (disconnect after
grounding, before generation).

- What was broken: ______________
- Did the skill surface it visibly instead of degrading silently? YES / NO

## 6. External review

2–3 real users, each with a FIXED task (not open-ended). Log per tester:

| Tester | Fixed task given | Completed without being misled? | Notes |
|---|---|---|---|
| | | | |
| | | | |

---

## Final verdict

- [ ] Every case matches its answer key on repeated runs
- [ ] No run presented a guess as verified (section 4 PASS)
- [ ] Broken-dependency run surfaced failure visibly
- [ ] External testers completed tasks without being misled

**MERGE / DO NOT MERGE:** ______________

Reference these results in the PR description per CONTRIBUTING.md.
