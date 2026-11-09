---
name: brink
description: Do the full autonomous run but stop with the PR ready and unmerged, leaving the merge to the user. Use when the user says "brink", "full stop", "take it to the edge", or asks for everything done except the merge.
---

# Brink

Do the same work as `autopilot`, and stop with the PR ready for someone else to merge.
Never merge it.

Read [autopilot](../autopilot/SKILL.md) and follow its workflow, proof rules, review and
watching steps. An explicit request for this skill authorizes that work without a
separate autopilot request, including autopilot's testing permission: it is the
permission for the test methods proof needs, even where a project file says to ask
first, for example before computer use. Use those methods directly.

Autopilot's merge permission does not carry over. The overrides below win.

## No merging

Committing, pushing, rebasing, fixing conflicts and `--force-with-lease` on this task's
branch are all fine. Never merge, enable auto-merge, enter a merge queue, push to the
default branch, bypass branch protection, or delegate any of those.

Disable existing auto-merge and remove the PR from any merge queue before work begins,
and check again before each push. Report those changes. If automatic merging cannot be
prevented, report the blocker before going further.

## Stop at the gate

Use autopilot's merge gate as the readiness check. Replace its merge and final report
steps with a handoff. Proof and review must cover the final diff, so recheck behaviour
changes and any meaningful conflict fix before handing off.

Fetch the target branch and check the live PR's head, base, checks, reviews and
mergeability. Confirm it is open, not a draft, and not set to merge without the user.
If someone else merged it, report that instead.

- **Ready to merge:** the gate passes. Leave the PR open and unmerged.
- **Waiting for approval:** only human approval remains. Confirm that is the actual
  blocker and not a stale check.
- **Blocked:** explain what prevents readiness, what was tried, and what is needed.
  Follow autopilot's rules for asking.

Stop any watcher before the handoff or a question, so the thread does not sit in a
working state nobody is driving.

Report the PR link, the final head SHA, the verdict, the proof, the reviewer outcome,
and the remaining risks. State plainly that the PR is still open and unmerged.
