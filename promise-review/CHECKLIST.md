# Attack checklist (general)

Common ways software breaks. The reviewer goes through it once per review. It is not a list of tests to pass: it is a reminder of where bugs usually hide.

**Rules**
- Check only the classes that apply to this spec. For each one you skip, write `N/A: <one-line reason>`.
- About 2 minutes per class. Try the riskiest input first.
- Every break still has to cite a promise or a must-accept example. If the spec says nothing, record a **SPEC GAP**, not a defect. Never "harden" past the spec.

| # | Class | Try |
|---|---|---|
| 1 | Size and limits | empty, one item, huge numbers (thousands of digits), very long lines, big files, deep nesting. No crash, no hang, sensible memory. |
| 2 | Malformed input | wrong type, bad or mixed encoding, non-ASCII, CRLF line endings, truncated or half-written data. |
| 3 | Boundaries | 0, -1, min and max values, exactly at a limit, one past it (off-by-one). |
| 4 | Time | 29 Feb, DST changes, time zones and offsets, year end, far past or future dates, a clock injected as 0 or epoch. |
| 5 | Falsy but valid | 0, "", [], False passed on purpose are used as real values, not swapped for defaults (`is None`, not truthiness). |
| 6 | Environment failures | missing or unreadable file, a directory where a file is expected, disk full or unwritable, output closed early (broken pipe; on Windows it can surface as EINVAL), network down or slow. |
| 7 | Errors | a clear message, no stack trace shown to users, the right exit code, nothing silently swallowed. |
| 8 | Repeats and timing | retries don't apply twice, two runs at once, events out of order, interrupted half-way. |
| 9 | Valid input still works | rerun every must-accept example after each fix. |
| 10 | Security basics | untrusted input reaching a shell, SQL or file paths; secrets or personal data in logs or errors. |

In the review report, add one line: `Checklist: 1 ok, 2 ok, 3 N/A (no numeric input), ...`.
