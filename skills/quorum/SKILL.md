---
name: quorum
description: Convene three independent reviewers from different models and reconcile their findings into one verdict with scores per round. Use when the user says "quorum", asks for a panel, a council, a second and third opinion, or a multi-model review.
---

# Quorum

Three reviewers, each a different model, each at high reasoning effort. Their
disagreements are the point: one model's blind spot is rarely shared by all three.

## Before convening

Address outstanding PR comments, fix known issues, and run the relevant checks first.
The panel reviews the updated implementation; do not defer known fixes until reviews
come back and waste a round on findings already known.

The exception is when the implementation is about to be discarded and the question is
which direction to take instead. Then skip fixes to code being replaced.

## Picking the panel

Use one strong model from each of three different providers. If the request names
members, use those. Otherwise take the current flagship from each provider.

In Claude Code, start one subagent per model with the `Agent` tool, in parallel, setting
`model` per agent and `effort` to high. Where a provider is not reachable from this
harness, say so rather than substituting a second model from a provider already on the
panel. A panel of three where two are the same model is not a panel of three.

Give each reviewer the request and the context it needs. Keep their work read-only for
a review or a critique.

## Running a round

Ask each reviewer for findings and a score out of 10. One full review pass is one
round. A fresh review after fixes starts a new round.

Collect the responses and reconcile them. Report the useful findings, where the
reviewers disagreed, and the recommended next step. A finding two reviewers raise
independently is worth more than one raised loudly by one.

Check each finding against the source before acting on it. Three models agreeing does
not make a claim true.

Where the findings are agreed with, make the changes and start a fresh round. Continue
until the panel is satisfied.

Do not let repeated reviews turn into scope creep. Fighting to keep the scope small is
part of the job, not an obstacle to it.

## Reporting

If the work ends in a pull request, put the results in the description:

```markdown
## Panel review

Rounds: 2

| Reviewer | Round 1 | Round 2 |
| --- | --- | --- |
| Claude Opus | 7/10 | 9/10 |
| GPT Sol | 8/10 | 10/10 |
| Gemini Pro | 7/10 | 9/10 |
```

Where a reviewer gave no score, write `no score`. Never invent one.

If the PR is open, edit the description. Do not post a comment for this. If there is no
PR, the same table goes in the final reply only.
