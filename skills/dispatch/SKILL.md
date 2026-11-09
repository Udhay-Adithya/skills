---
name: dispatch
description: File a concise pull request whose title and description explain why the change matters. Use when asked to file, open, create, raise or send up a PR.
---

# Dispatch

Before filing, check whether a PR for this branch already exists. Review the diff
locally against the base branch (usually `origin/main`) to make sure its contents match
the goal.

## Title

PR titles often become commit messages, so follow the repository's conventions. Look at
recently merged PRs and the Git history for examples. Prefer a concise title that says
why the change matters rather than what was edited:

BAD

> perf(server): negotiate permessage-deflate on the websocket

GOOD

> perf(server): cut websocket frame size by 70% with gzipping

Where the repository uses Conventional Commits, keep the subject lowercase throughout,
imperative, under 72 characters, with no trailing period.

## Description

Open with the problem, in the terms the original request used. Then say briefly how it
was solved. Do not lead with an inventory of the implementation:

BAD

> Removed implicit workspace carry-over from every "new thread" entry point (cmd+n /
> cmd+shift+o, sidebar v1/v2 buttons, command palette). New threads inherit only the
> project from context; branch, worktree, and env mode always come from the configured
> defaults. Deleted buildContextualThreadOptions,
> startNewThreadInProjectFromContext, and the v1 sidebar's seed-context machinery.

GOOD

> My "new worktree" default was ignored when starting new threads on existing
> worktrees. Super unintuitive. Now your preferences always apply.

Add one short line saying how the change was proven: the command that was run, the
reproduction that now passes, the screenshot attached.

No tool or model attribution. No "generated with", no co-author trailers, nothing that
narrates how the change was written. The description is written in the voice of whoever
owns the repository.

## Filing

Check the title, description, quotes and attachments for secrets and private details
before submitting.

Open a real PR rather than a draft, so review bots run. If `gh pr create` fails because
the GraphQL rate limit is used up, open it through REST:

```bash
gh api repos/<owner>/<repo>/pulls -f title=... -f head=... -f base=... -F body=@<file>
```

Do not post a follow-up PR comment unless it serves the request. When a comment is
needed, use the `remark` skill. If watching the PR was also asked for, continue with
the `vigil` skill.
