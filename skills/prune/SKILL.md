---
name: prune
description: >-
  Write and prune documents that agents read: skills, CLAUDE.md, AGENTS.md, NOTES.local.md and
  anything reached by a pointer. Finds duplicated meaning, stale lines, no-ops and sprawl, and
  cuts them. Use when creating or editing a skill, changing CLAUDE.md or AGENTS.md, when two
  documents say different things about one subject, when a document has grown long, or when
  instructions are being ignored.
---

# Prune

A document an agent reads is not prose. It is a program whose purpose is to make the agent
take the same process every run. The levers that make it predictable are the same whether
it is a skill, a `CLAUDE.md`, or a note reached by a pointer.

Most of the work is removal. Adding feels safe and removing feels risky, so the default
fate of any instruction file is **sediment**: stale layers that settle until you have to
core down through them to find what is still live. Two statements about one subject where
one has gone stale is the sediment showing.

## The four cuts

Walk the document four times, once per cut. Each pass finds things the others do not.

### 1. Duplicated meaning

Each meaning gets **one source of truth**, one authoritative place, so changing the
behaviour is a one-place edit.

Duplication costs twice: maintenance, and a false signal of importance, because a meaning
stated three times reads as more important than it is.

**Where two statements of one meaning disagree, that is the finding.** Do not merge them
into something that splits the difference. Work out which is current, delete the other,
and say which was dropped so the decision can be reversed.

This is the opposite of repeating a **term** on purpose. Repeating one word to anchor a
behaviour is a technique. Repeating the meaning in three paragraphs is the bug.

### 2. Stale lines

A line goes stale when the behaviour or the world it describes changes. Check anything
naming a version, a file path, a command, a flag, a model or a service.

**The environment is a source of truth too.** `package.json` scripts, `pubspec.yaml`, the
directory layout, `--help` output. A document that restates any of those is a cache of a
lookup, and it earns its place only where the lookup is expensive.

Cache what cannot be found by looking: the unwritten convention, the reason behind a
choice, the trap no config file admits to. Leave the one-command lookups to the
environment, where they cannot rot.

### 3. No-ops

An instruction the model already follows by default costs tokens to say nothing.

The test is whether the line changes behaviour against the default, and it is relative to
the model, not to a reader. Two people disagreeing about whether something is a no-op are
disagreeing about the default, and the way to settle it is to run the document, not to
argue.

When a sentence fails the test, delete the sentence. Do not trim it down to a shorter
no-op.

The test also grades word choice. "Be thorough" is a no-op against a model that is already
fairly thorough. The fix is a stronger word, not a longer sentence.

### 4. Sprawl

A document can be too long while every line in it is live, unique and true. Attention
thins across the excess, and each extra line is one more thing to keep current.

The cure is not compression, it is the hierarchy below.

## Where a piece of material sits

Three rungs, ordered by how immediately the agent needs it:

1. **In-file step.** What to do, in order. The primary tier.
2. **In-file reference.** Consulted on demand. A flat set of peers, every rule of a review
   on one level, is a legitimate shape and not a smell.
3. **Disclosed reference.** In its own file, reached by a pointer, loaded only when the
   pointer fires.

Pushing too little down bloats the top. Pushing too much down hides what is needed. That
tension is the whole decision, and branching is the cleanest test for it: **inline what
every path needs, disclose what only some paths reach.**

In a document with steps, in-file reference that should have been disclosed buries those
steps, and whether the agent attends to them becomes a coin flip.

**Keep a concept's parts together.** Definition, rules and caveats under one heading, so
reading one part brings its neighbours along. Scattering one meaning across a file is a
different failure from duplicating it, and it reads worse.

## Pointers

A pointer is the line that names out-of-context material and encodes when to reach it. A
skill's `description` is one. A line in `CLAUDE.md` naming a document is the same object.

**The wording of the pointer, not the quality of its target, decides whether the material
is ever reached.** Important material behind a weak pointer is a reliability bug. Sharpen
the wording first, and only inline the material if sharpening fails.

A pointer does two jobs: say what the material is, and name the distinct cases that should
trigger it. An always-loaded pointer costs on every single turn, so it takes harder
pruning than the body:

- Put the triggering word first.
- One trigger per case. Synonyms for one case are that case written twice.
- Cut identity the body already carries.

## Say what to do, not what to avoid

Steering by prohibition pulls the forbidden thing into context and makes it **more**
available. The negation is a weak modifier against a strongly activated concept, so a ban
half-reads as an instruction.

State the target behaviour instead, so the unwanted one never gets named. Keep a
prohibition only as a hard guardrail that cannot be phrased positively, and pair it with
the positive target even then.

## End every step on something checkable

A step ends on a condition that says the work is done. Vague bounds invite stopping early,
because attention slides toward being finished.

Two properties matter. **Can the agent tell done from not-done?** "Understanding reached"
cannot be checked; "every modified file accounted for" can. And **how much does it
demand?** "Every rule applied" forces more work than "produce a list", and it binds a flat
set of rules just as well as it binds a sequence of steps.

Sharpen the wording first, since that is cheap and local. Only split the sequence to hide
the later steps if the bound is genuinely irreducible and the rush is actually happening,
and note that hiding only works across a real context boundary. An inline call leaves the
later steps sitting in context.

## NOTES.local.md

Caveats, open questions, things unverified and anything that would reveal how the work was
done belong in `NOTES.local.md` at the repository root, gitignored, never in code,
commits, docs or a README.

It is the one document here written for a human rather than an agent, which changes what
pruning means for it:

- **It is append-mostly, and that is correct.** It is a log. Do not prune it for length.
- **Prune it for resolution.** A question that has been answered, a caveat that has been
  fixed, a "check whether X" that was checked: delete the entry, or move its conclusion
  into the document that now owns it. An open-questions list where half the questions are
  closed stops being read.
- **Date entries**, and convert any relative date to an absolute one when writing it.
  "Last week" in a log is worthless three months later.
- **Where an entry contradicts something in `CLAUDE.md` or a skill, that is a pruning
  job in the other document**, and the note was the symptom. Fix the source of truth and
  delete the note.

Before writing to it, confirm it exists and is ignored:

```bash
grep -q NOTES.local.md .gitignore || echo "NOTES.local.md" >> .gitignore
```

## Reporting a prune

Say what came out and why, grouped by cut:

| Cut | Location | What | Why |
|---|---|---|---|
| Duplicated | `CLAUDE.md:40`, `skills/x/SKILL.md:12` | Commit format stated twice, the two disagree on scope casing | Kept the `CLAUDE.md` statement, which matches the actual history |

For a conflict, always say which statement survived and which was dropped. That is the one
finding here nobody can verify from the diff alone, and it is the one most likely to be
wrong.
