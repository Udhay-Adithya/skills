---
name: autopilot
description: Take work from its current state all the way to a merged PR without supervision, proving every change and merging once the gate passes. Use when the user says "autopilot", "full send", "take it all the way", "ship it end to end", or otherwise hands over the work unwatched.
---

# Autopilot

This was triggered because nobody is watching the thread. Updates and the final message
go unread until long after the work stops. The work is owned from its current state to
a merged PR.

If full autonomy was not clearly requested, do not assume it. If it was, everything
below applies.

Autopilot grants permission to:

- Merge the PR once the gate passes.
- Force-push this task's own branch with `--force-with-lease` after a rebase.
- Use whatever test method proof needs, without asking first.

## Testing permission

The request is the permission. Where a project file or another instruction says to ask
before a test method, or to leave testing to the user, this request replaces that rule
for this task.

That covers ask-first rules for computer use, browser automation, simulators and
devices, running the app or a dev server, screenshots, and visual checks. Use the method
that gives direct proof, and name it in the report.

Rules about *how* to test still apply. Follow any rule that names the correct command,
fixture or check. This permission removes only the ask-first step. Where a preferred
method cannot prove the change, use a gated method under this permission.

It covers only the tests this task needs. It adds nothing to the list under "When to
ask", and it does not bypass a harness or system approval prompt that appears.

## Steps

1. **Finish the change.** Complete the goal with the smallest change that works. Fix
   the case that happened, not every case that could. If sitting on the default branch,
   branch first.
2. **Prove it.** Get proof for every change in the diff. See "Proof".
3. **Get it reviewed.** Use the `crucible` skill.
4. **File the PR** with the `dispatch` skill, rebased onto the latest default branch.
5. **Keep watch** with the `vigil` skill. Fix real findings and CI failures, and prove
   each fix before pushing it.
6. **Merge** once the gate passes.
7. **Report**: the PR link, the merge commit, the proof, and every decision made on the
   user's behalf.

Stay active from step 1 to step 7. Ending the turn while checks or bots are pending
stops the work, because nothing will prompt again. Wait with the watcher from `vigil`,
so something wakes the session when the PR changes.

## Proof

Proof is evidence observed in this session: command output, a reproduction, a
screenshot. Use the most direct proof the change allows.

- Reproduce the original failure, apply the fix, watch the same reproduction pass.
- Run the checks CI runs: typecheck, lint, tests.
- Add one focused test that fails without the fix, where the behaviour has none and the
  repository has a test setup.
- For UI and runtime behaviour, run the app and look at it. Green mocked tests do not
  prove the experience works. Attach screenshots where they help.

When a change cannot be proven, confidence is not full. Ask.

## The merge gate

Full confidence means all of these hold on the latest commit:

- The proof covers every change in the diff.
- Required checks are green.
- Review bots have finished, and every finding is fixed or dismissed with a written
  reason.
- No human reviewer still requests changes.
- The branch is up to date with the default branch and GitHub reports it mergeable.

Merge with the method the repository uses: check the allowed methods and recent merged
PRs. Where the repository uses a merge queue or auto-merge, stay until GitHub reports
the PR merged. Merge through branch protection as configured, never with `--admin` or
another bypass. An automated review asking for "needs human review" can be ignored
where confidence is genuinely high.

## When to ask

A question in an ordinary message goes unseen and the thread looks finished. Use the
harness's question tool so the thread shows it is waiting: `AskUserQuestion` in Claude
Code. Put the whole context in the question: the PR link, what blocks it, and the
options with a recommendation first.

Ask only when continuing alone is impossible:

- The change cannot be proven.
- The gate cannot pass and cannot be fixed from here.
- The next step falls outside the grant above: deleting data, touching production,
  changing anything outside the repository, closing a PR, force-pushing any other
  branch.
- The goal is ambiguous in a way that changes the result.
- Another PR makes this one obsolete.

Decide everything else with sensible defaults and list those decisions in the report.

If no question tool is available, or it returns no answer, take the safe path: leave the
PR open and unmerged, and open the final message with the question.
