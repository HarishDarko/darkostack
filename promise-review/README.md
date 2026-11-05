# Promise Review: promises, build, review for AI coding agents

Part of [darkostack](../README.md).

This is a small set of three Markdown files that you hand to an AI coding agent. First you write down the **promises** the code must keep: rules that must never break, plus valid inputs that must keep working (`SPEC.md`). A **builder** agent turns each promise into tests, watches them fail, makes them pass and reports real command output (`BUILDER.md`). Then a **fresh reviewer**, an agent that never saw the builder's conversation, tries to break each promise with bad inputs and faults, saves every break as a failing test, ties every finding to a line of the spec, and answers **APPROVED** or **CHANGES** (`REVIEWER.md`). Because the reviewer may only block on things it can prove against the spec, it does not pile on generic "hardening" that rejects valid input.

## Why

It came out of a small, informal comparison. Several setups built the same task with the same underlying model. Every setup passed all 79 hidden tests, so the hidden tests could not tell them apart. A blind probe round, scored out of 20 (closed output pipes, edge dates, falsy injected clocks, valid inputs that must still be accepted, and so on), did:

| Setup | Probe score (/20) |
|---|---|
| Claude Code on its own | 12.5 |
| Claude Code + a fresh Claude reviewer | 16.5 |
| Claude Code + a Gemini reviewer | 13.5 |
| A three-agent team | 18.5 |
| **This template** (`full/`) | **19.0**, but took about 2 h 12 min |
| This template, lite (`lite/`) | **17.5**, in about 66 min (half the full version's time) |

What seemed to matter: explicit promises, a reviewer with clean context, and a rule that every finding must cite the spec. Generic hardening advice lowered scores.

**Caveat:** each setup was run once, on one task. Treat these numbers as a hint, not a benchmark.

## Which version

- `full/`: the full version. The reviewer tries a deliberate break for every promise, and you allow up to 3 review rounds.
- `lite/`: written to cut the run time. It allows at most 2 review rounds, does deliberate breaks only for promises you mark `high` risk in the spec (at least 3), and makes round 2 a recheck of earlier findings rather than a new review.

Both folders hold the same four files: `SPEC-TEMPLATE.md`, `SPEC-CHECK.md`, `BUILDER.md` and `REVIEWER.md`.

## Use it with Claude Code (desktop app or CLI)

1. Copy `SPEC-TEMPLATE.md` into your project as `SPEC.md` and fill it in. Ten minutes is usually enough; the Promises and Must-accept sections matter most. Copy `BUILDER.md` and `REVIEWER.md` into the project too (or note where they are).
2. **Spec check, before any code.** In a fresh session say:
   `Follow SPEC-CHECK.md. Read only SPEC.md (and the starting code, if any).`
   Fix the spec for each item it lists, and log the change in section 9. This takes a few minutes and catches gaps before they turn into bugs.
3. **Builder session.** Open a session in the project (a new chat in the desktop app, or run `claude` in the project folder) and say:
   `Follow BUILDER.md to implement SPEC.md. Stop at the done report.`
4. **Reviewer session, with fresh eyes.** Start a **new** session so it has no memory of the build (in the CLI, a new `claude` run without `--continue` or `--resume`) and say:
   `Follow REVIEWER.md. Review this repo against SPEC.md only. Do not read any other notes.`
   You can also ask the builder session to launch a subagent and give it only the paths of `REVIEWER.md` and `SPEC.md`; a subagent starts with a clean context.
5. If the verdict is **CHANGES**, give the builder the reviewer's report and the new `tests/review/` tests, let it fix the code, then run a fresh reviewer again. Stop after 3 rounds (2 for the lite version) and decide yourself.

## Use it with any other agent

Nothing here is tied to one tool. Any agent that can read files and run commands works: give it the same two prompts. The things to keep:

- The reviewer runs in a **separate session with a clean context** and gets only `SPEC.md` and the repo, never the builder's chat or summary.
- A second reviewer on a different model is fine as an extra opinion, under the same rule: findings that do not cite the spec are notes, not blockers.
- The examples use Python, but the steps are language-neutral. For property tests, a loop over a seeded standard-library random generator is enough (the comparison runs used exactly that). A property-testing library such as Hypothesis (Python) or fast-check (JavaScript) is optional.

## Credits and licence

The ideas are borrowed from several public skill packs and docs; see [CREDITS.md](CREDITS.md). The wording in this repo is original. Licence: see [LICENSE](../LICENSE).

## Attack checklist
`CHECKLIST.md` is a general list of ten common failure classes (size, malformed input, boundaries, time, falsy values, environment failures, errors, repeats, valid-input regressions, security basics). Both reviewer versions use it. It was added after the contest and tested on a new task the template had never seen (an expense splitter, written without knowledge of the checklist). With and without it, both versions passed all 45 hidden tests and 12 fixed probes. A blind code read scored the checklist version 8.5/10 against 7.5/10: it kept reporting all errors after a malformed record and was about 4x faster on large splits, for about 2 extra minutes. One run each, so treat it as a signal.

## Spec check and per-promise status (Nov 2026)
Five additions, inspired by HumanLayer's Research-Plan-Implement skills and QualityLayer (see `research/` and CREDITS):
- `SPEC-CHECK.md`: a fresh agent reads only the spec and lists untestable promises, contradictions and missing must-accept inputs **before** any code is written.
- The builder stops on a spec problem with a fixed `SPEC GAP: Expected / Found / Why it matters / Proposal` note instead of guessing.
- Proof output is never cut short (no piping through `head`, `tail` or `grep`).
- The reviewer gives every promise one line: PASS, FAIL or NOT CHECKED. NOT CHECKED never counts as a pass.
- `SPEC.md` keeps a dated change log for edits after the build starts.

Tested on a new held-out task that starts from existing code (a small layered-config CLI: add overrides, fix two reported bugs, find four unreported ones). Same model, same reviewer, one run each:

| Setup | Hidden tests | Fixed probes | Blind score (/10) | Time | Output tokens |
|---|---|---|---|---|---|
| Lite, before these additions | 45/45 | 12/12 | 8.5 | 42 min | ~121k |
| **Lite with these additions** | 45/45 | 12/12 | **9.5** | 46 min | ~131k |

The blind judge added 57 probes and 900 random cases (both passed all) and made 13 deliberate breaks: the new version's own tests caught 13, the old one's 12. The spec check flagged a real gap (no defined exit code for a failed output write) that the final code then handled with a clear error, where the old version exited 0 silently. One task, one run each: a signal, not proof.
