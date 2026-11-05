# BUILDER instructions (lite)

You write the code and its tests from `SPEC.md`. A separate reviewer will try to break your work afterwards, so build every promise in a way that can be proven.

## Before coding
1. Read `SPEC.md` and restate the goal in two lines.
2. If a promise is unclear, or a `[NEEDS DECISION]` is still open, **ask**. Do not guess.
3. Settle where the tests plug in: the public seam the tests call, e.g. the CLI or `main(argv, stdin, stdout, clock)`. Make the clock, the filesystem and the output stream injectable so every test is deterministic.

## For every promise (P1, P2, ...)
1. Write at least one test named `test_P<n>_<short>` (or your language's equivalent). Give it a three-line comment or docstring:
   `protects:` which promise; `fails when:` the wrong result it catches; `does not prove:` what it leaves unchecked.
2. Where the code has a clear shape, write a **property test**: generate many inputs and assert a rule that must hold for all of them. A loop over a seeded random generator from the standard library is enough (print the seed; a few hundred cases is a good default). A property-testing library such as Hypothesis (Python) or fast-check (JavaScript) is optional; it adds automatic shrinking.
   Choose the rule that constrains the output most. Roughly, from strongest to weakest:
   - matches a simple reference version, or survives a round trip (`parse(format(x)) == x`);
   - applying it twice gives the same result as once (`f(f(x)) == f(x)`);
   - counts, order or bounds are preserved;
   - it does not crash.
   If no such rule fits, use plain examples and say so in the report.
3. A property test can pass while checking nothing. Watch for two causes:
   - the test works out the expected answer the same way the code does, so a bug they share goes unnoticed;
   - the input filter discards most generated cases, so almost nothing actually runs. Count the cases that ran.
4. **Red first.** See each test fail, for the right reason, before you write the code. For code that already exists, do the break-it-on-purpose check only for the 3 riskiest promises.
5. Every must-accept example from spec section 4 gets a test. Every fault from section 5 gets a test (for example a fake output stream that raises a broken-pipe error, or a clock that returns `0`).
6. When a generated case fails, keep the smallest failing input as a fixed example test (Hypothesis: `@example(...)`; seeded loop: record the seed and the input), so it is replayed on every run.

## Rules
- Check injected values with an explicit "is it missing" test (`is None` in Python), never with truthiness. `0`, `""` and the epoch can all be valid inputs (see P5-style promises).
- Build what the spec asks for and nothing more: no extra features, no general hardening the spec did not promise.
- **Stop on a spec problem.** If the spec cannot be followed as written (a contradiction, a missing case, a promise no test can check), stop and write:
  `SPEC GAP: Expected <what the spec says> / Found <what you found> / Why it matters <impact> / Proposal <your suggested wording>`.
  Do not resolve it yourself; wait for the human's answer.
- **Raw proof.** Run every proving command with its full output. Never pipe it through `head`, `tail` or `grep`, and paste the real summary line.
- **Do not lower the bar.** Every skipped test, weakened assertion, changed threshold or new warning suppression gets a separate line in the done report, with the reason.

## Done report (paste real output; never write "should work")
```
Commands run:     <cmd> -> <N passed, M failed, exit code>
Promise -> tests: P1 -> test_P1_rows_roundtrip (property, 500 cases, seed 1234) ...
Break-it check:   P1 broken on purpose -> test failed? yes/no
Bar changes:      none | <one line each>
Spec gaps raised: none | <SPEC GAP lines and the answer you got>
Not verified:     <what, and why>
```
Then give the reviewer **only** `SPEC.md` and the repo or branch. Do not pass on your notes, your summary or this conversation.
