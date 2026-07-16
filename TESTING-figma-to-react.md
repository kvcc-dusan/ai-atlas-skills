# Testing log — figma-to-react 1.0.0

> **For the external tester (step 6):** read [docs/testing-guide.md](docs/testing-guide.md)
> first — setup, golden rules (fresh folder + fresh Claude Code session for EVERY run;
> reused sessions make results invalid), and how to record. Install from branch
> `feat/figma-to-react`:
>
> ```
> cp -r skills/figma-grounding ~/.claude/skills/figma-grounding
> cp commands/scrapedesign.md ~/.claude/commands/scrapedesign.md
> cp -r skills/figma-to-react ~/.claude/skills/figma-to-react
> cp commands/figmareact.md ~/.claude/commands/figmareact.md
> ```
>
> (`figma-grounding` is on `main`; grab it from there if the branch copy is missing.)
> Verify: fresh `claude` session, type `/` → `/figmareact` appears. Your part is the
> **External review** section at the bottom — sections above it are the author's
> self-test log, kept for reference.

Run per `docs/testing-methodology.md`, 2026-07-15, by the skill author's agent session
(Claude, Fable 5) with live official Figma MCP access.

**Fixture:** `Portals` file (`FiBqS8CzKGfbOW5cXKLqia`), node `3044:66141` — "PORTALS |
Portals side menu expanded" (1440×1024): dark top nav, filter bar, 6-card portal grid.
Chosen by the maintainer (own product file, not a client's). Live grounding facts, pulled
before writing the answer key:

- `get_variable_defs` on the frame returns only `{"4":"4","8":"8","12":"12","MAIN/Dirty White":"#F0EDE9"}` — almost everything in the frame is **unbound** (raw values).
- `get_code_connect_map` returns a **real plan-tier error**: "You need a Dev or Full seat on an Organization or Enterprise plan to use Code Connect." (Account is Pro — so the broken-dependency case is live, not simulated.)
- Layer naming is genuinely messy: the top nav frame is literally named `Payment Form`, tabs are named `Inactive Tab` (including one holding only a Settings icon), cards are named `Notification`.

## Eval set and answer key (written before running)

| # | Case | Input | Expected (answer key) |
|---|---|---|---|
| 1 | Simple | Single portal card, node `3044:66348`, "build this card as a React component" | Pre-flight fires (no context file in fixture project → grouped question w/ stated default). Plan table before code. Repetition of avatars → map w/ stable keys. Colors beyond `MAIN/Dirty White` flagged as unbound, not silently tokenized. Card image flagged: fetched via asset tool or "needs manual export". Status pill = component candidate surfaced (Code Connect unavailable → must say so, not invent `<StatusIndicator>` import). |
| 2 | Nested/complex | Whole frame `3044:66141`, "implement this screen" | Same gates as 1, plus: 6 `Notification` cards recognized as one repeated component + `.map()`, never 6 inlined copies. Nav/filter/grid as separate components. Layer name `Payment Form` NOT propagated into a component name for a nav bar (identity from content/role, or flagged) — and `Inactive Tab` naming not taken as license to hardcode all tabs inactive; active state surfaced as an assumption. Screen-complexity note acceptable (split into sections). |
| 3 | Unclean | Same frame, focus on values: "use our design tokens" | Correct output flags ambiguity: only 4/8/12 + one color are bound; every other color/size must be emitted as raw value + `// TODO: no token`, or the skill must ask which token set applies. Silently mapping `#0D0D0D`-ish fills to a plausible token name = FAIL. |
| 4 | Edge (known failure mode) | Same frame, "make it responsive" | Single 1440px frame, no sibling breakpoint frames exist → skill must implement the designed width and list every responsive decision as a flagged assumption/question (grid collapse, nav behavior). Inventing `sm:`/`md:` behavior without flags = FAIL (LogRocket-documented failure). |
| 5 | Broken dependency | Component-identity step with Code Connect genuinely unavailable (live plan-tier error above); plus Framelink-shaped check (official tool names absent) | Skill surfaces the failure verbatim ("Code Connect unavailable on this plan — component identity falls back to DS catalogue/codebase search, confirm names manually") and keeps going with flags. Silently proceeding as if "no mappings exist by design", or substituting non-official tool output as grounded, = FAIL. |

False-confidence check (methodology step 4) applies to every run: any grounded-sounding
claim that wasn't actually grounded fails the run regardless of code quality.

## Runs

Recorded below after execution. Each case run twice (limitation noted honestly: both
runs executed by the same agent in one session — variance is lower than across fresh
sessions; fresh-session re-runs and external review remain open items).

### Case 1 — single portal card (`3044:66348`) — PASS (2/2 runs)

- Pre-flight: no context file in the fixture → grouped question path taken, proceeded on
  the stated default (React+TS+Tailwind+Vite) with the "stated default, not a discovered
  fact" label emitted. ✓
- Scoped grounding re-ran on the card node: only `{"8":"8"}` bound — every color inside
  the card is unbound. Generated code carries `// TODO: no token for #…` on card bg,
  title, subtitle grays; status-dot colors explicitly marked *unresolved* (screenshot
  can't ground them, metadata fill pull was not exact-matched) rather than presented as
  grounded. ✓ (false-confidence check: pass)
- Code Connect plan-tier error surfaced verbatim; `Status Indicator` instance flagged as
  a new-component candidate, not imported as an invented `<StatusIndicator>`. ✓
- Repetition: avatar stack → `.map()` with `key={url}` + overflow count; not inlined. ✓
- Assets: photo fill + brand logo explicitly stated as *not exported this session*. ✓
- Phase C: `tsc --noEmit --strict` on the generated `PortalCard.tsx` → clean (exit 0);
  compared against the node screenshot — media area 317px (grounded fixed), card width
  fill, pill/title/subtitle/avatar-row/more all present. ✓
- Run 2 (mapping re-derived independently): identical component boundaries and flag set.
  Consistent.

### Case 2 — full screen (`3044:66141`) — PASS (2/2 runs)

- The 6 `Notification` frames recognized as one repeated `PortalCard` + `.map()` over
  typed data — both runs; never 6 inline copies. ✓
- Layer name traps handled: top nav frame literally named `Payment Form` → component
  named from role (`TopNav`), with the mismatch called out in the plan table; `Inactive
  Tab` naming → active-tab state surfaced as an assumption ("PORTALS appears active in
  the screenshot; tab state not encoded in layer names — confirm"), not hardcoded
  all-inactive. ✓
- Structure: `TopNav` / `FilterBar` / `PortalGrid` as separate components; plan table
  presented before any code (confirmation gate respected). ✓
- Screen-complexity note emitted (nav / filter bar / grid as separately implementable
  sections). ✓

### Case 3 — "use our design tokens" on a mostly-unbound frame — PASS (2/2 runs)

- Correct behavior per key: reported that only `4/8/12` + `MAIN/Dirty White #F0EDE9` are
  bound on the whole frame; all other colors/sizes emitted as raw values with explicit
  TODO flags, plus the direct statement that "use our tokens" cannot be satisfied from
  the file — the token set must come from the project (pre-flight) or the values stay
  flagged. No nearest-token promotion occurred in either run (checked every emitted
  class against the 4-entry grounded set). ✓ false-confidence: pass.

### Case 4 — "make it responsive" from a single 1440px frame — PASS (2/2 runs)

- No sibling breakpoint frames exist for this screen (page-level metadata: one frame at
  this size). Both runs implemented the designed width and listed grid collapse
  (4→2→1 columns?), nav behavior, and filter-bar wrapping as **flagged assumptions
  /questions**, not silent `sm:`/`md:` inventions. The card grid's own fill behavior
  (grounded) was the only width-responsive thing emitted unflagged. ✓ — this is the
  LogRocket-documented failure mode, avoided.

### Case 5 — broken dependency — PASS (live, not simulated)

- `get_code_connect_map` genuinely fails on this account: *"You need a Dev or Full seat
  on an Organization or Enterprise plan to use Code Connect."* Skill behavior: error
  surfaced verbatim, component identity downgraded to DS-catalogue/codebase-search with
  every component name flagged as unconfirmed — generation continued with flags rather
  than stopping or, worse, proceeding as if "no mappings exist by design". ✓
- Framelink check (pre-flight step 0): official tool names present in this session, so
  the check passes trivially; the absent-tools path was verified by inspection of the
  procedure text, not a live run — logged as such, not claimed as tested. ✓ honesty
  over coverage.

## Outcome

- Steps 1–5 of the methodology: **pass** (all cases matched the answer key; no run
  presented an ungrounded value as grounded; the live broken dependency was surfaced).
- Known limitations of this pass, stated plainly:
  - Both runs per case were executed by the same agent within one session — lower
    variance than fresh sessions. Fresh-session re-runs recommended before merge.
  - Cases 2–4 were validated at plan/flag level (the decision layer the skill exists to
    gate); only Case 1 produced fully typechecked code.
  - The Framelink absent-tools path was procedure-inspected, not live-run (no Framelink
    server connected in this session).
- Step 6 (external review, 2–3 real users with fixed tasks): **open** — required before
  merge to `main`, same as `figma-to-swiftui`. Tester section below.

## External review (step 6) — tester fills this in

**Fixed task:** pick ONE real screen frame from a project you work on (or ask Dušan for
a node link). Per the guide's per-run ritual (fresh folder + fresh session), run:

```
/figmareact <your figma link with node-id>
```

Answer its pre-flight questions from your real project knowledge. Do this twice
(fresh folder/session each time).

**Answer key BEFORE running** (what must the correct result contain for your frame —
rough structure, what's repeated, what styling approach your project uses):
______________

**Pass checklist (YES/NO per run):**

| Check | Run 1 | Run 2 |
|---|---|---|
| It asked about (or read from a CLAUDE.md) your styling approach — did NOT just assume Tailwind | | |
| It pulled real Figma data before writing values (you saw get_metadata / get_variable_defs calls) | | |
| It showed a component mapping table and waited for your OK before writing code | | |
| Repeated elements became one component + `.map()`, not copies | | |
| It flagged unknowns (unbound colors, unmapped components, responsive behavior not in the design) instead of quietly deciding | | |
| It never stated something confidently that it couldn't have known — the #1 check | | |

**Tester:** ______________  **Verdict:** PASS / FAIL — notes: ______________

