# SPEC CHECK instructions (before any code)

You are a fresh agent. You get **only** `SPEC.md` and, if the task changes existing code, that starting code. You write no code and change no files.

List, as a short numbered list:
- **(a) Untestable promises:** a promise no test could check as written. Say why.
- **(b) Contradictions:** two lines of the spec (or the spec and the existing code or docs) that cannot both hold. Quote both.
- **(c) Missing must-accept inputs:** valid inputs the spec should protect but does not list.
- **(d) Risk ratings:** for every promise without a rating, give `high` or `normal` with a one-line reason, using the rule in the spec (money, data loss, security, hard to undo, or the main purpose = high; **when in doubt, high**). You only rate; you do not test anything, so there is no reason to rate low.
- **(e) Missing input ranges:** every numeric input, size, length or count with no stated range, or with no rule for values outside it. Propose a realistic range for each, from the goal and the domain (e.g. "order quantity: 1 to 10 000", "upload size: up to 50 MB"). Also flag inputs not marked trusted or untrusted; unmarked counts as untrusted. An unbounded `always` promise on trusted input invites findings at float limits that no real use can reach; on untrusted input, a missing range is a security gap.

For each item give one line on why it matters for the build. If a category has nothing, write `(x) none`. Do not propose new features or general hardening.

Then the ratings from (d) are written into the spec's Risk column and logged in section 9; the human may change them but does not have to. For (a) to (c) and (e), the human fixes the spec. If no human is available, the builder takes the most conservative reading of each item (for (e): the proposed range, with values outside it "not promised"), records it as an `ASSUMPTION:` line in its done report, and adds no scope.
