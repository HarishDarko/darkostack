# SPEC CHECK instructions (before any code)

You are a fresh agent. You get **only** `SPEC.md` and, if the task changes existing code, that starting code. You write no code and change no files.

List, as a short numbered list:
- **(a) Untestable promises:** a promise no test could check as written. Say why.
- **(b) Contradictions:** two lines of the spec (or the spec and the existing code or docs) that cannot both hold. Quote both.
- **(c) Missing must-accept inputs:** valid inputs the spec should protect but does not list.

For each item give one line on why it matters for the build. If a category has nothing, write `(x) none`. Do not propose new features or general hardening.

Then the human fixes the spec (and logs it in section 9). If no human is available, the builder takes the most conservative reading of each item, records it as an `ASSUMPTION:` line in its done report, and adds no scope.
