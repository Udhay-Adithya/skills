---
name: remark
description: Write a pull request comment, review reply or dismissal that leads with the conclusion and cites evidence. Use whenever leaving a comment on GitHub, replying to a review bot or a reviewer, or writing the reason for resolving or dismissing feedback.
---

# Remark

Covers top-level PR comments, inline review comments, replies to bots and reviewers,
and the written reason given before resolving or dismissing feedback.

## Voice

The comment is posted in the repository owner's voice. No attribution callout, no model
or harness name, no robot emoji, no signature footer. Nothing in a comment indicates
how it was written.

This is a deliberate departure from upstream, which prefixes every agent-authored
comment with a note naming the model. Flip it back by adding that callout here if the
project you are in wants the disclosure, and check whether the repository's own
contributing guide requires it.

## Writing it

- Lead with the conclusion.
- Cite the relevant code, behaviour, check or reproduction. A claim with a
  `file:line` behind it ends an argument; a claim without one starts another.
- Keep it concise. No canned thanks, no status narration, no filler.
- Separate verified facts from judgment. Say which is which.
- Disagreeing is fine, and a bot being wrong is common. Say why, with evidence.

## Before posting

Check the entire comment, including quoted text and attachments, for secrets and
private details.

Preview the final Markdown so quoting and blank lines render as intended. Do not add a
horizontal rule or a Setext underline that will swallow the line above it.

## After posting

Verify that GitHub returned the comment or review URL. Resolve a review thread only
after its reply actually posted.
