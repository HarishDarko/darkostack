# SPEC: <task name>

> Fill in every section and keep it to roughly one page. Both the builder and the reviewer work from this file and nothing else.
> Copy it into your project as `SPEC.md`. The example rows below are invented; replace them with your own.

## 1. Goal
Two to four plain sentences: what is being built, and why.

## 2. Interface
How the thing is called. Example: `tool [--from DATE] FILE` reads FILE (or stdin) and writes CSV to stdout. Exit codes: 0 = ok, 1 = bad input, 2 = I/O error.
List every dependency the tests must be able to swap out (**injectable**: passed in, not hard-coded), such as the clock, the filesystem or the output stream.

## 3. Promises (must never break)
One row per promise. Each one must be checkable by a test.

| ID | Promise (plain words) | Kind | Check with | Fails when (the wrong result it rejects) |
|----|----------------------|------|------------|-------------------------------|
| P1 | Every valid input row produces exactly one output row, in the same order | always | property test (count + order) | rows lost, duplicated or reordered |
| P2 | Formatting a date and parsing it again returns the original date | always | property test (there and back) | 29 Feb, year 1, year 9999 or a timezone shift alters the date |
| P3 | A bad date is reported on stderr with its line number, and processing goes on | always | example + probe | crash, silent skip, wrong line number |
| P4 | A half-written output file is never left behind | never | fault probe (disk full, killed mid-write) | a truncated file remains |
| P5 | An injected clock that returns `0` (the epoch) is a real time, not "no clock" | must-accept | example | the code treats 0 as missing (a truthiness check) |

Kinds:
- **always**: holds for every input.
- **never**: must not happen under any input.
- **must-accept**: valid input that has to keep working. A reviewer may not "harden" it away.
- **must-reject**: input that has to be refused; say how it is refused.

## 4. Must-accept examples
Concrete valid inputs that must always work. They stop over-correction, where a reviewer adds checks that reject valid input.
- `2024-02-29`, `0001-01-01`, an empty file (empty output, exit 0), a 2 GB file, non-ASCII names, CRLF line endings, ...

## 5. Faults to survive
Keep the ones that apply and write down the expected behaviour for each.

| Fault | Expected behaviour |
|---|---|
| Output pipe closed early (`tool big.csv \| head -1`) | exits quietly, no stack trace |
| Huge input (e.g. 10^7 rows) | streams; memory stays flat |
| Edge dates (leap day, DST change, year boundary, timezone) | as P2 |
| Falsy injected values (clock = 0, empty string, null) | as P5 |
| Missing, unreadable or unwritable file | exit 2 with a clear message |

## 6. Not doing (out of scope)
Required; never leave it empty. A reviewer may **not** ask for anything on this list.
- No locale-specific date formats. No network access. No config file.

## 7. Done means
The exact commands that must pass, e.g. `pytest -q` and `pytest -q tests/review`, plus any manual checks.

## 8. Open questions
Write them as `[NEEDS DECISION: ...]`. The builder asks rather than guessing. A reviewer who finds the spec silent on something files a SPEC GAP, not a defect.

## 9. Change log
Any edit after the build starts gets one dated line, e.g. `2026-11-05: P3 now also covers CRLF input (answer to the builder's SPEC GAP)`. Never edit a promise silently. The builder and the reviewer always work from the latest version.
