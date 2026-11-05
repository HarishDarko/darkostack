# HumanLayer RPI vs Promise Review

Researched 5 Nov 2026. Sources read: `humanlayer/humanlayer` at f9bd735 (Apache-2.0), `humanlayer/skills` at 4d69d20 (MIT), `humanlayer/advanced-context-engineering-for-coding-agents` at da16ab7, and Dex Horthy's talk "Everything We Got Wrong About Research-Plan-Implement" (YouTube YwZR6tc7qYg). `rpi-coordination-template` (a multi-repo setup) is not relevant. Section 7 covers QualityLayer. Paths are inside `humanlayer/humanlayer` unless marked.

**In one line:** RPI is about getting the *design* right before code, with a human approving each step. Promise Review is about proving the *code* keeps its promises, with a fresh agent attacking it. They fix different failures, so a few RPI ideas slot in cheaply.

## 1. What RPI is, phase by phase

The current RPI flow is: research questions, research, design discussion, structure outline, implement, PR (`apps/docs/reference/skills-workflows.md`). The old detailed plan is now optional ("legacy").

| Phase | What it does | Short quote and path |
|---|---|---|
| Research questions | One session turns the ticket into 2 to 8 questions about the code as it is today. | "NO 'HOW WOULD WE XYZ' - ONLY 'HOW DOES IT WORK'" (`apps/riptide-rpi-claude-plugin/skills/create-research-questions/SKILL.md`) |
| Research | A fresh session answers the questions with file:line facts. It never sees the ticket. | "You may NEVER read `ticket.md`/`task.md` files" (`skills/create-research/SKILL.md`). Also: "If there are no tests, say so explicitly" |
| Design discussion | Current state, desired state, options, open questions. About 200 lines. | "You may recommend an option, but you must NOT resolve or close a question yourself" (`skills/create-design-discussion/SKILL.md`) |
| Structure outline | Ordered vertical slices, each with files, automated and manual checks. About 2 pages. | "Avoid horizontal phases like 'add all types', 'add API endpoints', 'implement UI'" and "The structure outline is our 'c header files'" (`skills/create-structure-outline/SKILL.md`) |
| Implement | An implementer subagent does one phase, runs checks, ticks boxes, then stops for the human. | "STOP and think deeply about why the outline can't be followed" then `Expected / Found / Why this matters / How should I proceed?` (`apps/riptide-daemon/src/agents/claudecode/agents/outline-implementer-agent.ts`) |
| PR | Reads the whole diff against the artifacts and writes the PR body. | `skills/describe-pr/SKILL.md` |

Why they split it: the old planning prompt had "85 or more instructions", so steps got skipped. "Now they're all less than 40" (talk). Short artifacts passed between fresh sessions keep context at 40 to 60% (`ace-fca.md`).

## 2. Their plan-review and human-review flow

- **Who reviews:** the human, at each artifact. Not another agent. "The agent can propose a choice, but the user owns the decision" (`apps/docs/explanation/workflow-phases.md`).
- **What gets checked:** research for wrong facts ("a bad line of research ... could land you with thousands of bad lines of code", `ace-fca.md`); design for choices; outline for phase order and checks; each implemented slice by running it.
- **What they stopped doing:** reading full plans. "A thousand line plan tends to be about a thousand lines of code ... So, the new advice, uh don't read the plans" (talk). Review moved to the short design and outline. He still says "Please read the code."
- **Feedback loop:** feedback edits the artifact that owns the decision, in place, via `iterate-*` skills. "Feedback on a structure outline changes the outline. It does not start implementation." Later artifacts win: `plan > structure outline > TDD > PRD > design discussion > research > ticket`. Agents must check user corrections against the code: "DO NOT just accept the correction" (`create-structure-outline/SKILL.md`).
- **Automated code review:** thin. The legacy `.claude/commands/validate_plan.md` (commit 99abe673) ran in the same session ("Review the conversation history") with generic checks. On testing, Dex said: "as far as testing and verifying, sorry, I don't have a good answer for you" (talk transcript).
- **Guardrail hook:** `.claude/hooks/block-test-output-filtering.sh` blocks piping test output to `head`, `tail` or `grep`, so full results are always captured.

## 3. Side by side

| Area | HumanLayer RPI | Promise Review |
|---|---|---|
| Spec | Ticket, then design discussion or PRD/TDD. Decisions are options the human resolves. | One-page `SPEC.md` with promises, must-accept, faults, not-doing, done commands. |
| Research | Separate phase. Blind to the ticket. Facts with file:line and current tests. | None. Builder reads the spec only. |
| Plan | Structure outline of vertical slices with checks. Detailed plan optional. | None. Builder settles test seams, then works promise by promise. |
| Build | Subagent per phase, stops on mismatch, human gate per phase. | Red-first tests per promise, property tests, break-it check, bar-change log. |
| Review | Human reviews artifacts and code. Legacy agent check in same session. | Fresh agent, spec and code only, attacks promises, leaves failing tests, cites the spec. |
| Proof of done | Outline checkboxes ticked after automated checks; human confirms manual steps. | Done report with real output, promise to test map, reviewer APPROVED. |
| Context management | Fresh session per phase, artifacts as compaction, under 40 instructions per prompt. | Fresh reviewer only. Builder runs one long session. |
| Cost/time | Anecdotes only: 7 h for 35k LOC (3 h research/plan), aim for 2 to 3x. No token figures published. | Measured once: full 19.0 in 2h12 / ~207k; lite 17.5 in 66 min / ~152k. |

## 4. Worth borrowing, ranked

| # | Idea | Source | Why it fits | Exact change | Re-test? |
|---|---|---|---|---|---|
| 1 | Review the spec before the build. A fresh agent reads only `SPEC.md` and lists untestable promises, contradictions and missing must-accepts. | `ace-fca.md` "On Human Leverage"; `workflow-phases.md` checkpoints | The spec is the highest-leverage artifact. SPEC GAPs now surface only after the build. | README step 1, add: "Before the builder starts, give a fresh agent only `SPEC.md` and ask: which promises can't be tested, which contradict, which valid inputs are missing? Fix the spec. No code." | Yes |
| 2 | Builder mismatch stop in a fixed format. | `outline-implementer-agent.ts` | "Ask, do not guess" now covers only the start. This covers surprises mid-build. | BUILDER Rules, add: "If the spec can't be followed as written, stop and write `SPEC GAP: Expected <spec line> / Found <what> / Why it matters / Proposal`. Do not resolve it yourself." | No |
| 3 | Unfiltered test output. | `.claude/hooks/block-test-output-filtering.sh` | Backs your "paste real output" rule. A piped `tail` can hide a failure. | BUILDER done report, add: "Run the proving commands unfiltered (no `\| head`, `\| tail`, `\| grep`). Quote the final summary line." Same line in REVIEWER step 2. | No |
| 4 | Blind current-state research for existing code. | `create-research-questions/SKILL.md`, `create-research/SKILL.md` | Same idea as your fresh reviewer: hide the goal to stop opinions. Brownfield only. | BUILDER "Before coding", add step 0: "If code already exists, write `RESEARCH.md` first: how it works today and how it is tested, with file:line. Facts only, no proposals. Best done by a subagent given questions, not the spec." | Yes |
| 5 | Thin end-to-end slice first. | `create-structure-outline/SKILL.md` | Proves the seam and entry point before many promise tests depend on them. | BUILDER "Before coding" step 3, add: "Slice 1 runs the real entry point on one must-accept example end to end. Add promises in later slices. Run the full suite after each." | Yes |
| 6 | Only the human resolves open questions; decisions get written back. | `create-design-discussion/SKILL.md`; skill conventions in `skills-workflows.md` | Keeps SPEC section 8 honest, and keeps `SPEC.md` the single contract the reviewer cites. | SPEC-TEMPLATE section 8, add: "The builder may add a recommendation. Only the human marks it resolved, and the answer is written into the spec before review." | No |

## 5. What not to borrow

- **The full chain** (questions, research, design, PRD, TDD, outline, plan, worktree, PR). Built for large features in big codebases. Full already takes 2h12.
- **Detailed plans.** HumanLayer itself dropped them as a review target: they are as long as the code.
- **Human gates per phase.** They break unattended contest runs. Keep the single hand-off after the round cap.
- **Same-session validation and its generic checks** ("missing validations", "Error handling is robust"). This is the generic hardening that lowered scores, and it reads the builder's history.
- **"Later artifact wins" precedence.** If a plan can override the spec, reviewer findings stop having one source to cite. Write decisions back into `SPEC.md` instead (idea 6).
- **Research swarms, visuals, PR walkthroughs, worktrees, cloud comments.** Product plumbing, heavy on RAM and tokens.

## 6. Suggested next experiment

T4 is already used. RPI research pays off only on existing code, so use an unseen brownfield task.

- **Task T5:** a small existing repo plus a feature request and a bug report (T3 shape, new domain).
- **H (control):** lite template as is.
- **J:** lite plus ideas 1 to 4 (spec pre-review, mismatch format, unfiltered output, blind research). Skip 5 to isolate the planning effect.
- **Measure:** probe score, hidden tests, minutes, tokens, SPEC GAPs at review, rounds, must-accept regressions.

| Added step | Expected time | Expected tokens |
|---|---|---|
| Spec pre-review | +5 to 8 min | +15k to 25k |
| Research questions + blind research | +10 to 20 min | +30k to 60k |
| Fewer review rounds (if it works) | -10 to 20 min | -20k to 40k |
| **Net vs H (66 min, ~152k)** | **about +5 to +25 min** | **about +25k to +60k** |

Estimates, not measurements. J wins if it scores 18.5+ in under 90 minutes with no must-accept regressions. Repeat before changing the template.

## 7. QualityLayer (github.com/maxritter/qualitylayer, formerly Pilot Shell)

Proprietary, source-available and licensed. Only the README and public pages at qualitylayer.dev/docs were read. Ideas only; no text or files taken. Flow: Discuss, Research, Plan, Outline, Implement, Verify, Review, with human approval at Plan and Review.

- **Discuss vs SPEC.md:** Discuss settles what done means as numbered points, and later changes are kept as visible amendments. That matches your promises. It records scope, and bug tasks note what must keep working, but the docs show no list of valid inputs that must be accepted, which is your guard against over-correction.
- **Verify vs REVIEWER:** an agent that did not write the code checks the running change against the request, with a verdict per point and two runs at most. Very close to your reviewer. Differences: every point needs recorded evidence, and what it cannot verify reaches Review as open, never as passed.
- **Required lines:** users add steps the agent must mark done or skipped with a reason. Your `Checklist:` line already does this.

Worth adopting:
1. **One status line per promise** in the REVIEWER output: `P1 pass (evidence) / P3 fail (test) / P4 not checked (why)`. "Not checked" can never count as a pass. Low risk; try it in J.
2. **A change log in SPEC.md:** any edit after the build starts gets a dated line and a reason, so the reviewer and the human see what moved. No re-test needed.
