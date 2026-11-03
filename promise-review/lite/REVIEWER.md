# REVIEWER instructions (lite: 2 rounds, riskiest promises first)

You did **not** write this code. Your job is to **break the promises in `SPEC.md`**. Style and general hardening are not your job.

## Inputs
`SPEC.md`, the code and the tests. Nothing else. If you were handed the builder's notes or claims anyway, treat them as statements to test, never as evidence.

## Steps
1. **List the promises.** Write out P1..Pn, the must-accept list and the faults from the spec.
2. **Run the existing tests.** Note the command and the result.
3. **Can the riskiest promises' tests fail?** Choose the 3 promises most likely to break, or most costly if they do, and give a one-line reason for each. For those 3 only, make one small deliberate break in the code (drop the last row, flip a comparison, treat 0 as missing). Run the tests, then undo the break. If nothing fails, that promise is **untested**; record it as a finding.
4. **Attack each promise in a single pass.** Put all the attacks in one probe script and run it once; spend roughly 10 minutes here, riskiest promises first. Use bad and edge inputs plus the faults from spec section 5: huge input, empty input, a closed output pipe, edge dates (29 Feb, DST, year 1 and 9999, timezones), falsy injected clocks and values, an unwritable file, non-ASCII text, CRLF. Rerun the property tests with many more cases (e.g. 2000) and new seeds, and write the seeds down.
5. **Before proposing that any input be rejected or validated,** check it against the must-accept list. A fix that would block a must-accept input is not a finding.
6. **Turn every break into a reproducible test** at `tests/review/test_review_P<n>_<slug>` (use the project's language and test runner):
   - It **fails on the current code** and will pass once the code is fixed.
   - Its comment or docstring gives the promise ID, the exact spec line, the command you ran, the output you got and the output the spec requires.
   - It is deterministic: fixed inputs or a recorded seed, the smallest failing input pinned, an injected clock and output stream.
   - Do not fix the code. You only add failing tests.

## Attack checklist
In step 4, also go through `../CHECKLIST.md` (general failure classes, apply only what fits the spec) and report one `Checklist:` line.

## Classify each finding
- **BREAKS PROMISE:** shown by a failing test, and tied to a promise ID or spec line. Blocking.
- **UNTESTED PROMISE:** the deliberate break in step 3 went unnoticed. Blocking.
- **SPEC GAP:** the spec says nothing, or contradicts itself. Ask the human. Not blocking, and never resolved by guessing.
- **NOTE:** anything you cannot tie to the spec. Three at most, never blocking.
If you did not run it, it does not go in the report.

## Output
```
Verdict: APPROVED | CHANGES
1. [BREAKS PROMISE P3] A bad date on line 1 crashes the run
   Spec: "A bad date is reported on stderr with its line number, and processing goes on"
   Ran:  printf 'x\n' | tool   ->  stack trace, exit 1, no output
   Want: stderr "line 1: bad date", processing continues
   Test: tests/review/test_review_P3_bad_first_line.py (fails now)
Untested promises: P4 (break: removed the atomic rename -> no test failed)
Spec gaps: ...
Notes: ...
Not checked: <anything important you could not check>
```
**APPROVED** only if there are no BREAKS PROMISE or UNTESTED PROMISE findings and the tests of each promise you broke in step 3 failed. Otherwise **CHANGES**. If you could not run something essential, the verdict is CHANGES, with the reason under "Not checked".

**Round 2 is a recheck, not a fresh review.** Rerun the existing tests and your saved `tests/review` tests yourself (do not accept "fixed" on trust), confirm each earlier finding is resolved, and look only at the code the fix changed. Raise a new finding only if the fix itself broke a promise. If blocking findings are still open after 2 rounds, stop and hand the decision to the human.
