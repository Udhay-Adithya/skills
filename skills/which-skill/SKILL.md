---
name: which-skill
description: Ask which of my skills fits the situation. A router over this repository, naming the flow to take and the order to take it in.
disable-model-invocation: true
---

# Which skill

Nobody remembers twenty skills, so ask.

**Before claiming what a skill does, or recommending a step be skipped, read that skill's
SKILL.md.** The summaries below are for orientation. They go stale; the skills do not.

A **flow** is a path through several skills. Most work travels the main flow, and two
on-ramps merge onto it. Everything else is standalone.

## The main flow: idea to merged

1. **Too big for one session, and foggy?** Start at **`atlas`**. It charts the effort as
   issues on the tracker and resolves them one at a time until there is nothing left to
   decide. It produces decisions, not code, and then hands off at step 2. Only reach for
   it when the way to the destination genuinely is not visible yet, never for a
   well-scoped feature.

2. **`spec-first`** turns the idea into a gated spec: assumptions surfaced, vague
   requirements reframed as testable criteria, then a hard stop for approval before
   anything is planned. Where `atlas` ran, this is where its linked decisions collapse
   into one buildable plan. Going from a map straight to code throws that collapse away.

   `spec-first` then breaks the plan into tasks. Keep steps 1 and 2 in **one window**: the
   charting, the spec and the task breakdown all build on the same thinking.

3. **Build each task in its own window.** Each task is self-contained, so the previous
   one's context is disposable. **`flutter-riverpod-mvvm`** fires by itself on anything in
   Dart.

   Anything with a surface takes two passes, in order. **`design-engineering`** decides
   what goes on the screen, what it is called, which states it has and whether it works by
   keyboard. **`attention-to-detail`** then does the finishing: radii, optical alignment,
   spacing, typography rendering, icon weight, hit areas, press feedback and the exact
   motion values. The first depends on the second and hands off to it.

   Running only the first leaves a well-structured screen that looks unfinished. Running
   only the second polishes something that may not have needed to exist. Neither needs
   invoking by hand.

4. **`code-quality`** reviews the diff on five axes before it goes anywhere, with the
   mutation check on the tests. For anything touching auth, permissions, money, data loss
   or concurrent state, add **`crucible`**, which runs an adversarial loop against a model
   that is not the one that wrote the code, or **`quorum`** for three reviewers from three
   providers with scores per round.

5. **`dispatch`** files the PR: problem first, then solution, with the proof.
   **`vigil`** then keeps watch through CI and the review bots.

6. **`autopilot`** runs steps 3 to 5 unattended and merges at the end. **`brink`** is the
   same run stopping with the PR ready and unmerged. Reach for either when nobody is
   watching the thread.

7. **`release`** when it is time to cut one: it derives the bump from the commits in the
   tag range and stops for approval before touching a version.

## On-ramps

- **Something is broken in production or in a test.** Diagnose it first, then re-enter the
  flow at step 3 with the fix. A bug fix lands with the test that would have caught it, in
  the same commit.
- **A stale PR needs a decision.** **`revive`** judges whether it still belongs in the
  product before rebasing it, and will recommend closing it.
- **Someone else's unfinished work.** **`inherit`** finishes the original goal while
  preserving their credit.

## Underneath everything

These run beneath the others rather than beside them.

- **`smart-zone`** governs how long any of this can run in one window, and writes the
  paste-ready prompt when the window is spent. Invoke it at a phase boundary, or when the
  reasoning has started to drift.
- **`prune`** is for the documents themselves: a skill, `CLAUDE.md`, `AGENTS.md`,
  `NOTES.local.md`. Reach for it when two documents disagree, when an instruction is being
  ignored, or when a file has grown long.
- **`remark`** shapes anything posted to GitHub on my behalf.
- **`security-review`** for a real vulnerability question. `code-quality` has a security
  axis for the shallow pass; this one requires a trust boundary and a result before it
  calls anything a finding.

## By situation

| Situation | Reach for |
|---|---|
| A loose idea, too big to hold in one session | `atlas` |
| A clear idea, needs building properly | `spec-first` |
| A new screen, flow or component | `design-engineering`, then `attention-to-detail` |
| It works, but it looks unfinished | `attention-to-detail` |
| It looks fine, but the structure is wrong | `design-engineering` |
| Anything in Dart or Flutter | `flutter-riverpod-mvvm` |
| A diff to review before merge | `code-quality` |
| Auth, permissions, money, concurrency | `crucible`, then `security-review` |
| A second and third opinion with scores | `quorum` |
| Open a PR | `dispatch` |
| Watch a PR through CI | `vigil` |
| Do all of it unattended, and merge | `autopilot` |
| Do all of it unattended, do not merge | `brink` |
| An old PR nobody has touched | `revive` |
| Someone else's half-finished branch | `inherit` |
| Cut a version | `release` |
| The session is getting long or drifting | `smart-zone` |
| A skill or CLAUDE.md needs editing | `prune` |
| Two documents disagree about one thing | `prune` |
| Commenting on GitHub | `remark` |
| A vulnerability question | `security-review` |

## What is deliberately not here

No skill enforces the writing style, the commit format or the attribution rules. Those
live in the global `CLAUDE.md` because they must hold on every line of every turn, and a
skill that fails to trigger is a rule that silently stopped applying. Keep them there.
