---
name: vigil
description: Keep watch over a pull request through review and CI, handling bot findings and failures until it is ready. Use when asked to watch, monitor, babysit or keep an eye on a PR, or to see a PR through review.
---

# Vigil

Keep watch over a pull request until it is ready. Many repositories have AI review
bots. They are helpful, even when they are not right.

## Waiting

Never write a watcher, a polling loop or a `sleep` loop. Use one watcher per PR, and
never two on the same PR. Every watcher on a GitHub account shares one rate limit.

### In the Claude Code desktop app

The app watches PRs itself. Handle the existing checks and comments first, then bind
the PR with the PR tools and turn on Auto-fix. The app wakes the session when a check
fails, the branch conflicts, or a review comment arrives, so do not also start
`pr-wait`, and never schedule or poll checks by hand.

Auto-fix covers CI failures, conflicts and review comments. It does not decide whether
to merge. Check the gate yourself.

### Everywhere else: `pr-wait`

Wait with `scripts/pr-wait` from this skill's directory, by its absolute path.

```bash
pr-wait <pr> [--repo owner/name] [--until ready|green|any-change]
```

It prints one line per real event. It exits 0 when the `--until` condition is met or
the PR merges, 1 when it needs attention (a required check failed, conflicts,
unresolved threads, PR closed), and 2 on bad input.

- `ready` (default): required checks green, a review bot has reviewed the head, and no
  unresolved threads. Use this to keep watch.
- `green`: required checks only. Use it where the repository has no review bot.
- `any-change`: exit on the first event.

In Claude Code, run it with the Monitor tool (`timeout_ms: 3600000`) so each line wakes
the session. Without a monitor, run it as one long foreground command. It ends on its
own.

Start it again after a push, after handling an exit 1, and after a resume. It remembers
what it already reported, so nothing that arrived in between is lost. If it says
another pid already watches the PR, do not start a second one. Kill the old one only if
its events are not arriving.

With either watcher: when an event needs no action, end the turn without a message.
Never post or say "still waiting".

## GitHub rate limits

`gh pr create`, `gh pr view` and `gh pr checks` use GraphQL. When GraphQL is used up,
the REST limit is separate and usually still has room:

```bash
gh api repos/<owner>/<repo>/pulls/<n>
gh api repos/<owner>/<repo>/commits/<sha>/check-runs
```

If a watcher stops because it could not read the PR, the rate limit is the usual
reason. Check `gh api rate_limit --jq .resources.graphql`. If `remaining` is near 0,
wait for `reset` with one background command, then watch again. Never retry in a tight
loop.

## Reviews

Act only on checks and comments newer than the latest push. Verify every bot finding
against the source before changing code. Fix real findings and CI failures, tell a
repository failure apart from an infrastructure flake, and reply with a written reason
when dismissing a false positive.

If the repository has rules a bot cites (for example `.macroscope/check-run-agents/*.md`),
read them before replying to that bot's finding.

Advisory bots are advisory. Treat their findings like any other finding, but never wait
for one and never hold a merge for one.

When a bot leaves feedback not worth addressing, reply with a written reason and resolve
the thread. Use the `remark` skill for every comment posted on someone's behalf.

Keep the PR at the size the goal needs. Review loops grow a PR one cheap-looking fix at
a time. Fix real bugs in the intended case. Decline the rest with a written reason and
list them as follow-ups: unneeded tests, guards for edge cases that do not matter,
extra integration or compatibility work, small performance costs. Prefer the fix that
deletes or narrows code.

Screenshots and videos help too, where there is a way to upload them.

## Base branch and merging

Watch for changes to the default branch and rebase when needed. `pr-wait` reports when
new base commits touch this PR's files, and when the branch becomes BEHIND or DIRTY.
A harness watcher usually reports conflicts only, so check for BEHIND independently
before calling the PR ready.

If an overlapping PR makes this one obsolete, stop watching, report it, and ask before
closing the PR unless closure was already authorized.

Stay quiet rather than posting filler comments. Stop when `pr-wait` exits 0, or when
the gate passes on the latest head. Merge only when merging was explicitly requested;
otherwise report that the PR is ready.

Right before a merge, run `git fetch` and check the PR again. Never merge when it is
BEHIND or DIRTY, or when base commits that landed after the last green run touch this
PR's files. Rebase, push, and wait for green again.
