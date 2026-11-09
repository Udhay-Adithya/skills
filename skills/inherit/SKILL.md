---
name: inherit
description: Take over unfinished work from another branch, PR or agent thread, finishing the original goal while preserving the original contributors' credit. Use when asked to inherit, take over, pick up or finish someone else's PR, branch or thread.
---

# Inherit

Take responsibility for finishing the original goal while preserving the original
contributors' credit. Do not defend an implementation just because another agent
already built it.

## Read the handoff

Read the source PR, branch or thread, including review findings and any later
corrections. Resolve the handoff to the actual code and the current head, and account
for uncommitted work.

Treat the handoff as context, not as proof. What a previous thread claimed it verified
is a claim, not a verification.

## Credit

Preserve Git authorship and existing co-author trailers. Credit the original
contributors in the PR description and link the original PR where there is one. Where
the work will be squashed, preserve human author and co-author credit in the merge
commit too, using verified identities rather than guessed emails.

Credit goes to the people who did the work. No tool or model attribution appears
anywhere, and it would not be a substitute for contributor credit if it did.

## Shape of the continuation

For a replacement PR, import the original commits onto a new branch and add the fixes
separately, so the inherited work and the new work stay legible. Honour an explicitly
requested in-place or stacked continuation instead. Never overwrite unrelated work or
change the original PR without authorization.

Use the `revive` skill to bring the continuation up against its intended base and the
current architecture. Rework what finishing the same goal requires, not unrelated
features. Simplify where it can be simplified without dropping intended behaviour or
surfaces.

Recheck the source PR while working: it may have moved.

## Verify independently

Reproduce the original failure and the realistic workflows, following the repository's
guidance and permissions. Green mocked tests do not prove the experience works. State
plainly what remains unverified.

Where the requested finish includes a PR, use `dispatch` to file or update it, then
`vigil` rather than stopping at creation. Use `remark` for replies. Merge or close the
original PR only when that was authorized.

## Report

What was inherited, what changed, how it was verified, how credit was preserved, and
the remaining decision.
