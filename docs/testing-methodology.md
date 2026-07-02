# Testing methodology

Standard process for validating a skill before it merges into this repo. This is not optional and not per-skill-discretion — every skill goes through all six steps.

## 1. Build a 5-case eval set

Cover:
1. A simple case.
2. A nested/complex case.
3. A deliberately "unclean" case — something without a clean answer.
4. An edge case likely to trigger a known failure mode.
5. A case from a context where a dependency is missing or misconfigured.

## 2. Record the answer key first

Manually write down the correct/expected answer for each case **before** running the skill. Skip this step and you're evaluating vibes, not accuracy.

## 3. Run each case 2-3 times

Not once. Non-determinism means a single clean run is not evidence the skill works.

## 4. Check for false confidence

Explicitly check: does the skill ever present a guess with the same confidence as a verified answer? This is the highest-severity failure category — worse than being visibly wrong. A visibly wrong answer gets caught; a confident wrong answer doesn't.

## 5. Break a dependency on purpose

Misconfigure a tool, revoke a permission, remove access — whatever applies to the skill. Confirm it surfaces the failure instead of silently degrading to a worse (but plausible-looking) output.

## 6. External review — only after self-testing passes

Circulate to 2-3 real users for a fixed task each (not open-ended exploration). Use a structured feedback template. Log results in one shared place, not scattered messages.
