---
name: revive
description: Bring a stale pull request up to date by judging whether it still belongs in the product before rebasing it, and recommending modernize, rework or close. Use when asked to revive, refresh, rebase or modernize an old PR, or what to do with a PR that has gone stale.
---

# Revive

Revive the idea, not just the branch. A clean rebase does not prove the PR still
belongs in the product.

## Guardrails

- Pull the latest origin for this branch, the base branch, and any other branch related
  to the work before starting.
- Treat existing worktree changes as owned by the user. Use a disposable worktree when
  the active one is dirty or in use.
- Reviving authorizes rebasing and updating the PR branch. It does not authorize
  merging, closing, changing the base branch, or widening scope.
- Never use an unqualified force push. Update a rebased branch with `--force-with-lease`
  against the head SHA observed before the rebase.
- Do not post a PR comment unless one was asked for. Use the `remark` skill for every
  comment posted on someone's behalf.

## Establish the PR's intent

Resolve the PR and record its URL, state, base, head repository and branch, current head
SHA, author, checks and review state. Read the title, body, linked issues, commits,
review threads and the complete diff. State the smallest coherent goal the PR was
trying to accomplish, before judging how it went about it.

Fetch the latest remote base and PR head. Use the PR's actual base branch rather than
assuming the local branch is current. Record the merge base, so the original PR and
later base-branch changes can be compared separately.

## Check what changed around it

Inspect the commits and diffs added to the base branch since the PR diverged. Search
merged and open PRs when titles, code paths or behaviour suggest a replacement or a
competing implementation.

Look past overlapping lines:

- Has the same user-facing problem already been solved?
- Did APIs, types, schemas, dependencies or architectural boundaries change?
- Do both sides now claim ownership of the same behaviour or invariant?
- Was the code being changed deleted, moved, or deliberately simplified?
- Would landing this restore an old behaviour, or create two ways to do one thing?
- Is the original benefit still large enough to justify the maintenance cost?

A PR is not current merely because Git reports no conflicts.

## Choose a disposition before pushing

- **MODERNIZE:** the goal is still useful and the implementation is still the right
  shape. Rebase, resolve conflicts, test, update the branch.
- **REWORK:** the goal is still useful, but a mechanical rebase would preserve the wrong
  implementation. Explain the smallest credible rewrite and stop before pushing, unless
  that rework was also asked for.
- **CLOSE:** superseded, obsolete, directionally wrong, or no longer worth its cost.
  Cite concrete evidence and stop. Do not spend time rebasing an obsolete branch, and
  never close it without explicit permission.

Be willing to recommend CLOSE. Age alone is not evidence, and sunk effort is not a
reason to keep a PR alive.

## Rebase without losing intent

For MODERNIZE:

1. Create a safe local branch or a disposable worktree at the recorded PR head.
2. Rebase its commits onto the freshly fetched base branch.
3. Resolve each conflict according to the PR's intent and the base branch's newer
   invariants. If a conflict reveals a product or architecture disagreement, stop and
   reconsider the disposition instead of guessing.
4. Compare the pre-rebase and post-rebase commit series with `git range-diff` and
   inspect the final diff. Confirm no intended behaviour disappeared and no
   already-landed behaviour came back.
5. Run the repository's focused checks for the touched area. Tell real failures apart
   from pre-existing or infrastructure ones.
6. Push with `--force-with-lease` tied to the previously observed remote head. If the
   lease fails, fetch and reassess rather than overwriting new work.

Do not merge the PR.

## Report

Lead with MODERNIZE, REWORK or CLOSE and the reason. Then: the meaningful changes on the
base branch, the conceptual conflicts or confirmation there were none, the rebase and
push result if there was one, the checks run and their results, and the remaining
decision with a link to the PR.

Separate observed facts from judgment. Say what could not be verified rather than
manufacturing confidence.
