---
name: smart-zone
description: Keep work inside the context window where the model still reasons sharply, and when it runs out, finish the current piece cleanly and emit a paste-ready prompt that starts a fresh session with everything it needs. Use when a session is getting long, when reasoning has started to drift or repeat, before starting a large new phase of work, after a compaction, or when asked for a handoff, a fresh start, or a prompt to continue elsewhere.
---

# Smart zone

The **smart zone** is the span of context within which the model still reasons sharply.
Past it, the window technically still accepts tokens and the quality of the thinking in it
quietly falls off: earlier decisions get forgotten, instructions from the top of the
session stop being followed, and the same ground gets re-covered.

Working past the edge of it is the most common reason a long session produces worse work
than a short one, and it is almost always invisible from the inside. The fix is not to
push on. It is to finish the piece in hand and start again with a clean window and a good
prompt.

## Knowing where you are

The token count cannot be read directly, so work from what is observable.

**Hard signals.** A compaction or summarisation has already happened. The harness has
warned about context. The user says so. Any of these means the edge is already behind, not
ahead.

**Proxies, in rough order of weight.**

- Large files read in full, especially several, and especially anything over a few hundred
  lines.
- Long tool outputs: a wide `grep`, a full test run, a verbose build log, a big diff.
- Output that got persisted to a file rather than returned inline, which means it was big.
- Many turns, with many rounds of edit-and-check.
- The same file read more than once in a session, which usually means the first read has
  fallen out of working attention.

**Behavioural signals, which matter more than any count.** A decision made earlier being
re-litigated. A constraint from the top of the session being broken. Re-reading something
already read. Summarising instead of acting. A plan that keeps being restated rather than
advanced. Any of these means the zone has been left already.

Say so out loud when it happens. A session that silently degrades wastes more work than
one that stops.

## At a phase boundary, pick one

A **phase** is a chunk of work inside a session: the research, the implementation, the
review. Decide **at** a boundary. Mid-phase, either continue or split the remainder into
subagents, because a handoff written mid-edit loses the part that was in flight.

1. **Continue.** Costs nothing, loses nothing. Rule it out first, not last: continuing in
   the same window keeps every primary source that was read, and that is worth a lot.
2. **Subagent.** Send a tightly scoped piece to its own window and get back only the
   conclusion. The right move for anything that reads widely to produce a small answer:
   searching, auditing, checking. The parent's window stays clean.
3. **Clear.** Empty the window when nothing in it matters to what comes next.
4. **Compact.** Compress and carry on. The default, at the bottom of the list rather than
   the top, because compaction keeps a lossy paraphrase of primary sources rather than the
   sources.
5. **Hand off.** Write the prompt below. Narrow but decisive: a different harness, a
   different directory, another person, or a genuinely fresh start on the next phase.

## Before handing off

Finish the piece in hand. A handoff in the middle of an edit is worse than no handoff,
because the next session inherits a half-applied change and no way to tell what was
intended.

Specifically:

- Leave the tree in a state that builds, or say plainly in the prompt that it does not and
  what is broken.
- Commit what is coherent, so the next session can read the diff instead of guessing.
  Partial work that cannot be committed gets described explicitly.
- Write anything unresolved into `NOTES.local.md` rather than only into the prompt, since
  the prompt gets consumed once and the file persists.

## The handoff prompt

Emit it as one fenced block, so it can be copied in a single action. Everything the next
session needs, nothing it does not.

````markdown
```
## Goal

<the actual objective, in one or two sentences, as the user would state it. Not the
history of how it was approached.>

## Where things stand

<what is true of the code right now. Branch, what builds, what is committed, what is
in flight.>

## Done

- <each completed piece, with the commit or the file that proves it>

## Next

1. <the immediate next action, concrete enough to start on>
2. <then>

## Decisions already made

- <decision>: <the reason, briefly. This is the part that is most expensive to
  rediscover and most likely to be silently reversed.>

## Known traps

- <anything that was tried and did not work, so it is not tried again>
- <anything surprising about the codebase that cost time to learn>

## Read these first

- `path/to/file` <and why>
- <issue or PR URL>

## Skills to invoke

- <skill name>, for <what>
```
````

### What goes in it

**References, not copies.** Point at the spec, the plan, the ADR, the issue, the commit,
the diff. Re-stating their contents makes the prompt long and makes it a second source of
truth that will go stale against the first.

**The reasons behind decisions.** Everything else in the prompt can be rediscovered from
the repository. Why a particular approach was chosen, and what was rejected, cannot.
Without this section the next session re-opens settled questions, which is the single
biggest cost of a handoff done badly.

**The dead ends.** What was tried and failed is as valuable as what worked, and it is
nowhere in the code.

### What stays out

The conversation. How the thinking evolved. Tool output. Anything already captured in a
spec, a commit message or an issue. Exploration that led nowhere and taught nothing.

**Redact secrets.** The prompt becomes the next session's input and may get pasted
somewhere else entirely. No tokens, no keys, no credentials, no personal data.

## Working so the handoff stays cheap

The cost of a handoff is set long before it happens.

- **Write decisions down as they are made**, into the spec, an ADR, the issue, or
  `NOTES.local.md`. A decision in the conversation only is a decision that will be lost.
- **Commit at every coherent point.** The diff is the most reliable record of state there
  is, and it needs no maintenance.
- **Send wide reading to subagents.** Anything that reads a lot to produce a little
  belongs in another window.
- **Read the part of the file that is needed.** Reading 2000 lines to change one function
  spends the budget on 1990 lines of nothing.
- **Keep one phase per window where it is practical.** The exploration, the spec and the
  task breakdown want to be in one window because each builds on the last. The
  implementation of each task does not, and starts better from a clean one.
