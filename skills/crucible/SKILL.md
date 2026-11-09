---
name: crucible
description: Put a change through rounds of adversarial review by a reviewer that is not the model doing the work, fixing real findings and declining the rest on a strict budget. Use when the user says "crucible", asks to harden, stress or adversarially review a change, or wants a review loop before filing a PR.
---

# Crucible

A loop that makes a change more correct before, or sometimes after, it becomes a pull
request. One reviewer, several rounds, a written ledger, and a hard budget.

Make the smallest changes that answer the findings. Watch for scope creep, and push
back on findings that do not deserve a fix.

## The reviewer

The reviewer must not be the model doing the work. A model reviewing its own diff
shares every blind spot that produced it, which is the whole reason this loop exists.

In rough order of preference:

1. A subagent on a different provider's flagship model, where this harness can reach
   one.
2. A subagent on a different model from the same provider, started with the `Agent`
   tool and a `model` override, at high effort.
3. The harness's own review command at its highest effort.

Pick the latest model unless a version was named. Never hardcode a model ID that will
go stale. Record provider, model and effort if the harness exposes them, and say "not
exposed" rather than guessing.

One reviewer only. The reviewer must not start subagents of its own, and must not edit,
commit, push or comment.

State which tier was used in the report. A round 2 reviewer on the same model as the
worker is worth saying out loud, because it is a weaker signal than it looks.

## Budget

- Default 3 rounds. Round 1 is full. Later rounds look only at changed code and open
  findings.
- Extend to 5 only for security, permissions, auth, data loss, or state and
  concurrency, and only while a blocker or a should stays open with evidence. Or when
  asked.
- Stop early if a round returns only nits. Declined nits, and comment, doc or
  format-only fixes, need no new round.
- Keep a ledger: id, severity, outcome, token usage if returned. Never guess usage.

## 1. Scope

Run `git status --short`. Do not stash, discard or include unrelated changes. Limit the
review to the change paths where they separate cleanly. Otherwise stop and ask.

Find the base after `git fetch origin`:

- **On a PR:** `gh pr view <n> --json baseRefName,headRefOid`. Check out the branch only
  if the tree is clean. Confirm `HEAD` is the PR head, or ahead only by this change.
  Base is `git merge-base HEAD origin/<baseRefName>`.
- **Local:** `git merge-base HEAD origin/<default-branch>`. With no remote, use a base
  that was named, or `HEAD` for uncommitted changes only.

The change is base to working tree in those paths, including untracked files. Use one
diff command for every round. State the intent and what is out of scope in one or two
sentences. Run the covering checks. Do not send code that this change breaks. Note
missing or already-failing checks in the brief.

## 2. The brief

The reviewer gets no conversation history, so the brief carries everything.

```
Review only. Do not edit, commit, push, comment on PRs, or start subagents.
Repo: <abs path> Base: <sha> Intent: <intent> Out of scope: <scope> Checks: <results>
See: git diff <base> [-- <paths>]; git status --short (read untracked).
Read surrounding code. Find bugs, broken callers, mismatch with intent.
Skip style unless it hides a bug. Hold to the smallest change. Flag code,
tests, or guards the intent does not need. Other paths, rare cases, small
perf, extra compat: nits.
blocker = intended case wrong. should = real bug callers will hit, in intent. nit = rest.
Reply:
VERDICT: APPROVE or CHANGES
S1 [blocker|should|nit] <file>:<line> <claim>. Fix: <suggestion> (stable ids)
```

Do not edit scoped files while a review runs. Wait for the reply. Do not poll, and do
not start a second reviewer. A timeout, an error or no reply is not an approval. On
failure, cancel, confirm it stopped, and restart with the ledger.

## 3. Answer the findings

Check each finding and each severity independently. Another code path, a rare case or a
small cost is a nit whatever label arrived with it.

- **Agree:** smallest fix. Prefer deleting or narrowing over a new lock, cache, helper
  or option. Fix the case that happened. Add one focused test that fails without the
  fix, where the intent needs it.
- **Disagree:** say why, with evidence: `file:line`, a test, command output.
- **Out of scope or a nit:** decline in one line and list it as a follow-up.

Update the ledger. Re-run the checks and `git diff --shortstat <base>`. If the review
grew the diff, cut before continuing. Never agree just to end the loop.

## 4. Next round

Start one only if budget remains and a blocker or a should is open, behaviour code
changed, or the approval does not cover the current behaviour diff.

Send the round 1 brief, the ledger (`fixed:` / `disagree:` / `declined:` with
evidence), the same base diff, and what changed since the last round. Tell the reviewer
to look at open ids and changed code only. Mark each old id `resolved` or
`still open: <reason>`.

## 5. Stop and ship

- **Agreed:** APPROVE on the current behaviour diff, no blocker or should open, and no
  remaining concern of your own.
- **Stuck:** an item stays disputed after two exchanges, or the budget ran out. Show
  both sides in two or three lines each and let the user decide. Running out of budget
  is not an approval.

Do not push during the loop. Push after agreement only if pushing, shipping or updating
the PR was asked for.

Report: rounds, reviewer tier and model, what the review changed, diff size before and
after, findings fixed and findings declined with reasons, usage if exposed, and whether
anything was pushed.
