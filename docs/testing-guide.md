# Skill testing guide — for testers

This is the step-by-step manual for testing a skill from this repo. It assumes
**nothing**: you don't need to know Claude Code well, you don't need to be a developer.
If you can follow a recipe, you can do this.

(The formal standard behind this guide is [testing-methodology.md](testing-methodology.md)
— you don't need to read it to test, but it's the source of truth if anything here seems
to conflict. Each skill also has its own workbook, `TESTING-<skill-name>.md` at the repo
root, with the exact cases and prompts for that skill. This guide is the *how*, the
workbook is the *what*.)

## What you're actually doing (30 seconds)

A **skill** is a set of instructions that forces Claude Code to follow our procedure for
a task instead of improvising. Testing a skill means checking it *actually* follows that
procedure — especially checking it **admits when it doesn't know something** instead of
confidently making things up. That's why "the output looks fine" is not a pass: a
beautiful result that quietly guessed a color or invented a component name is exactly
the failure we're hunting.

## One-time setup (do once, ~15 min)

1. **Install Claude Code** if you don't have it: https://docs.anthropic.com/en/docs/claude-code
   (desktop app or terminal — either works; this guide shows terminal commands, in the
   desktop app the same things happen through the UI).
2. **Connect the Figma MCP server.** Open the Figma **desktop app** → Figma menu →
   Preferences → *Enable local MCP Server*. In a terminal run:
   ```
   claude mcp add --transport http figma-desktop http://127.0.0.1:3845/mcp
   ```
   To verify: start `claude` anywhere, type `/mcp` — you should see the Figma server
   listed as connected. If not, stop and ask Dušan; don't continue without it.
3. **Install the skill you're testing.** From a local copy of this repo, on the branch
   named in the skill's workbook (the workbook tells you which):
   ```
   cp -r skills/<skill-name> ~/.claude/skills/<skill-name>
   cp commands/<command-name>.md ~/.claude/commands/<command-name>.md
   ```
   The exact folder and command names are at the top of the skill's workbook. Some
   skills need a second skill installed too (e.g. both codegen skills need
   `figma-grounding`) — the workbook says so.
4. **Verify the install:** start a fresh `claude` session, type `/` — the skill's
   command (e.g. `/figmaswiftui`) must appear in the list. If it doesn't, the copy went
   to the wrong place — ask, don't improvise.

## The golden rules (this is the part people miss)

1. **Every run = a brand-new folder AND a brand-new Claude Code session.**
   Never run two cases (or two runs of the same case) in the same session or the same
   folder. Why: Claude remembers everything from earlier in a session, and files left in
   the folder (like a `CLAUDE.md` from a previous case) change its behavior. Results
   from a reused session/folder are **invalid** — not "a bit worse", invalid, because
   you can't tell whether the skill worked or the leftover memory did.

2. **Write down the expected result BEFORE you run anything.**
   The workbook calls this the *answer key*. In plain words: for each case, first write
   what a correct result must contain (the workbook gives you a checklist to base it
   on). Then run, then compare. If you skip this and judge afterwards, everything
   "looks right" — that's human, and it's why the order matters.

3. **Run each case 2–3 times** (fresh folder + fresh session each time, per rule 1).
   Claude isn't deterministic — a case that passes once and fails twice is a **fail**.

4. **Confident nonsense is the worst failure — log it even if everything else passed.**
   If the skill states something as fact that it couldn't have known (a component name,
   a color value, a project convention), that single moment fails the run, no matter
   how good the code looks. This is the #1 thing we're testing for.

## The per-run ritual

For skill `figma-to-swiftui`, case 1, run 1 — adjust names accordingly:

```
mkdir -p ~/skill-tests/figma-to-swiftui/case1-run1
cd ~/skill-tests/figma-to-swiftui/case1-run1
claude
```

Then, inside the session:

1. Do the case's **setup** step from the workbook (usually: paste a provided file into
   the folder, or nothing at all — the workbook is explicit).
2. Paste the case's **prompt** from the workbook, exactly as written (plus your Figma
   link where it says so).
3. Let it finish. Don't help it, don't answer questions it *shouldn't* need to ask —
   if it asks something the workbook says it should already know, that's a finding;
   note it and answer minimally.
4. Fill in the case's **YES/NO checklist** in the workbook while it's fresh.
5. Exit the session (`/exit` or close the window). Next run → new folder, new session,
   from the top.

## Recording results

- Results go **directly into the skill's workbook file** (`TESTING-<skill>.md`) — edit
  it, tick the boxes, write notes in the blanks. Screenshots of weird behavior are
  welcome: drop them in a folder and mention the path, or paste them in Slack with the
  case number.
- Write down *what happened*, not what you think should have happened. "It printed a
  component named DSButton, I have no idea if that exists" is a perfect note.

## When you're stuck

Ask Dušan. Do **not** improvise a workaround to keep a case moving — a case you
couldn't run *is a valid result* (it usually means the instructions have a gap, which
is exactly the kind of thing testing exists to find). "I couldn't do case 4 because I
don't have X" is useful; a quietly improvised case 4 is not.
