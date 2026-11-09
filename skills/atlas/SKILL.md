---
name: atlas
description: >-
  Plan and track work too big for one session as GitHub issues: a tracking issue that indexes
  everything, child issues wired with real blocking relationships, a project board and
  milestones, so any agent can pick up the next ready piece by reading the tracker. Use when
  starting a large or foggy effort, when asked to plan something that spans sessions, to chart
  or map out work, to open issues for a plan, or to find and work the next ready issue.
---

# Atlas

Work too large for one session lives on the issue tracker, not in a file in the repository
and not in a conversation. The tracker is the shared state: any agent, in any harness, on
any machine, reads it and knows what is done, what is ready, and what is blocked.

This produces **decisions, not deliverables**, by default. Each issue resolves a question.
The map is finished when nothing is left to decide and someone can go and build. The urge
to just build the thing is usually the signal that the map is complete.

## Refer to issues by name

Every issue has a title. In anything a person reads, use the title and wrap the link
around it. Never a bare number.

A wall of `#42, #43, #44` is unreadable and a paragraph of titles is not. The numbers do
not disappear, they ride inside the link.

## The tracking issue

One issue, labelled `atlas:map`, is the canonical artifact. It is an **index, not a
store**: each decision lives in exactly one place, its own issue, and the map gists it and
links to it.

```markdown
## Destination

<what reaching the end of this looks like: the spec, the decision, the change. One or
two lines. Every session reads this before picking anything up.>

## Notes

<the domain, standing preferences for this effort, which skills each session should
invoke>

## Decisions so far

- [<issue title>](<url>): <one-line gist of the answer>

## Not yet specified

<questions that are coming but cannot be phrased sharply yet>

## Out of scope

<work consciously ruled out of this effort, with why, linking the closed issue>
```

Open issues are deliberately **not** listed. They are found by query, so the map cannot
drift out of date against them.

## Setting it up

```bash
gh label create "atlas:map" --description "Tracking issue for a multi-session effort" --color 0E8A16
gh label create "atlas:research" --description "Answer a factual question" --color 1D76DB
gh label create "atlas:prototype" --description "Build something rough to react to" --color 5319E7
gh label create "atlas:grilling" --description "Resolve by conversation" --color FBCA04
gh label create "atlas:task" --description "Do a thing that unblocks a decision" --color D93F0B
```

Create the map, then a milestone for the effort, then a project board if one does not
exist:

```bash
gh issue create --title "<effort name>" --label "atlas:map" --body-file map.md
gh api repos/{owner}/{repo}/milestones -f title="<effort name>" -f description="<destination>"
gh project create --owner <owner> --title "<effort name>"
```

Every issue in the effort takes the milestone, which is what makes progress countable, and
goes on the board, which is what makes it visible:

```bash
gh issue edit <n> --milestone "<effort name>"
gh project item-add <project-number> --owner <owner> --url <issue-url>
```

The milestone carries the percentage complete and the board carries the state. Do not
hand-maintain either number in the map body.

## Child issues and blocking

Use GitHub's native relationships, not a checklist in the body. Native sub-issues and
blocking render in GitHub's own UI, so the state is visible without opening the map, and
they survive an agent that never reads the body.

```bash
# attach as a sub-issue of the map (issue_id is the numeric id, not the number)
gh api repos/{owner}/{repo}/issues/<map-number>/sub_issues -f sub_issue_id=<child-id>

# the numeric id for a given number
gh api repos/{owner}/{repo}/issues/<number> --jq .id
```

Verify these endpoints respond before relying on them, since GitHub's sub-issue and
dependency APIs have moved more than once. Where native blocking is unavailable on the
repository, fall back to a `Blocked by:` line in the issue body and say in the map that
the fallback is in use, so nobody trusts a frontier query that cannot see it.

A **milestone** groups the effort. A **project board** shows the state. **Sub-issues**
express the hierarchy. **Blocking** expresses the order. Each does one job, and
duplicating any of them into the map body creates a second source of truth that will
disagree.

## The frontier

An issue is **unblocked** when everything blocking it is closed. The **frontier** is the
open, unblocked, unassigned children: the edge of what is known and takeable right now.

```bash
gh issue list --label "atlas:map" --state open                    # find the map
gh issue list --milestone "<effort name>" --state open --no-assignee
gh issue list --milestone "<effort name>" --state closed --json title,url,number
```

**Claim before working.** Assign the issue to yourself first, before any other action, so
a concurrent session skips it. The assignee is the claim, and an open unassigned issue is
unclaimed.

```bash
gh issue edit <n> --add-assignee @me
```

## Issue types

Every issue is either worked **with a person** or **alone**. An issue that needs a person
only resolves through that exchange, and an agent answering its own questions has broken
it rather than finished it.

| Label | Mode | For |
|---|---|---|
| `atlas:research` | Alone | A fact a decision waits on: documentation, a third-party API, a local codebase question. Delegate to a subagent with its own window. |
| `atlas:prototype` | With a person | Something rough and concrete to react to, when "how should it behave" or "how should it look" is the real question. Link the prototype, do not paste it. |
| `atlas:grilling` | With a person | Conversation. The default. |
| `atlas:task` | Either | Manual work that must happen before a decision can be made: signing up for a service so its API can be judged, provisioning access, moving data so its shape is visible. The one type that does rather than decides, and it earns that by unblocking a decision. |

## Fog of war

The map is **deliberately incomplete**. Do not chart what cannot be seen yet.

Beyond the live issues is the fog: decisions that are clearly coming but cannot be pinned
down, because they depend on questions still open. Resolving an issue clears the fog ahead
of it, and whatever has become sharp graduates into new issues.

**Issue or fog?** The test is whether the question can be stated precisely now, not
whether it can be answered now.

- **An issue** when the question is already sharp, even if it is blocked and nothing can
  be done about it yet.
- **Not yet specified** when it cannot be phrased that sharply. Do not pre-slice fog into
  issue-sized pieces: one patch of it may turn into three issues, or none.

## Out of scope

The destination fixes the scope, so work past it is not fog and does not belong in **Not
yet specified**. It gets its own section.

When an existing issue turns out to sit past the destination, **close it** and leave one
line in **Out of scope** with the gist, the reason, and a link to the closed issue. It
stays out of **Decisions so far**, which records the route actually walked.

Out-of-scope work never graduates. It comes back only if the destination is redrawn, and
then as a new effort.

## Charting

1. **Name the destination.** Settle it first, because it fixes the scope and shapes every
   issue.
2. **Map the frontier breadth-first.** Fan out across the whole space rather than going
   deep on one thread. If this turns up no fog, the effort fits in one session and needs no
   map. Stop and say so.
3. **Create the map**, the milestone and the board. Destination and Notes filled in,
   Decisions empty, the fog written into Not yet specified.
4. **Create the issues that can be specified now**, then wire the relationships in a
   **second pass**. Issues need numbers before they can reference each other, and a
   placeholder `#<n>` written in the first pass auto-links to some unrelated issue.
5. **Fire the research issues in parallel**, one subagent each with a fresh window.
6. **Stop.** Charting is one session's work and resolves nothing by hand.

## Working the map

Given a map, or nothing at all:

1. **Read the map.** The low-resolution view, not every issue body.
2. **Choose an issue.** The one named, or the first on the frontier. **Claim it** before
   anything else.
3. **Resolve it** as the type its label names. Read the label, because the body does not
   state the type. Fetch the full body of a related or closed issue on demand rather than
   loading them all. Invoke whichever skills the Notes block names.
4. **Record the resolution.** Post the answer as a comment, close the issue, and append one
   line to **Decisions so far**.
5. **Update the fog.** Create newly specifiable issues, clear the graduated patch from
   Not yet specified so it exists only as its issues, rule anything past the destination
   out of scope, and update or delete issues the answer has invalidated.

**One issue per session**, research excepted. Other sessions may be editing the tracker at
the same time, so re-read before writing.

## Handing off to the build

When the fog clears, this hands off. It does not build.

Collapse the map's linked decisions into a spec with `spec-first`, then break that into
implementation tasks. Going straight from the map to implementation skips the collapse and
throws away the linked detail, which is the whole product of the effort.
