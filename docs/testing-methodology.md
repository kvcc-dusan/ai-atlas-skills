# Testing methodology

The standard process for validating a skill before it merges into this repo. Every skill goes through all six steps — this is not optional and not per-skill discretion.

The reasoning behind the strictness: skills run inside non-deterministic model behavior. A skill that "worked when I tried it" may fire inconsistently, degrade silently when a dependency breaks, or — worst of all — present a guess with the confidence of a verified answer. These failure modes are invisible in a single casual test, which is exactly why the process below exists.

## 1. Build a 5-case eval set

The set must cover, at minimum:

1. **A simple case** — the happy path, the scenario the skill was written for.
2. **A nested/complex case** — realistic depth, multiple layers, the kind of input real work produces.
3. **A deliberately "unclean" case** — something without a clean answer. The correct behavior here is flagging ambiguity, not producing one.
4. **An edge case likely to trigger a known failure mode** — pick from the skill's `references/` failure catalog if it has one.
5. **A broken-dependency case** — a context where a tool the skill relies on is missing or misconfigured.

## 2. Record the answer key first

Manually write down the correct/expected answer for each case **before** running the skill. This is the answer key. Skip this step and you are evaluating vibes, not accuracy — output that "looks right" after the fact will always look right.

## 3. Run each case 2–3 times

Not once. Non-determinism means a single clean run is not evidence. A skill that passes a case 1 of 3 times fails that case.

## 4. Check for false confidence

Explicitly check every run: does the skill ever present a guess with the same confidence as a verified answer? This is the highest-severity failure category — worse than being visibly wrong. A visibly wrong answer gets caught by the user; a confidently wrong one does not. A skill that fails this check does not merge, regardless of how well it scores otherwise.

## 5. Break a dependency on purpose

Misconfigure a tool, revoke a permission, disconnect a server — whatever applies to the skill under test. Confirm the skill surfaces the failure explicitly instead of silently degrading to a plausible-looking fallback.

## 6. External review — only after self-testing passes

Circulate to 2–3 real users, each with a **fixed task** (not open-ended exploration — fixed tasks make results comparable). Collect feedback through a structured template and log it in one shared place, not scattered messages.

## Pass criteria

A skill passes when:

- Every eval case matches its answer key on repeated runs (steps 1–3)
- No run presented a guess as a verified answer (step 4)
- The broken-dependency run surfaced the failure visibly (step 5)
- External testers completed their tasks without the skill misleading them (step 6)

Log the results (cases, runs, outcomes) and reference them in the PR description — see [CONTRIBUTING.md](../CONTRIBUTING.md).
