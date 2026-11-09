---
name: spec-first
description: Write a gated specification before any code, surfacing assumptions and turning vague requirements into testable success criteria, then stop for approval before planning. Use when starting a project, a feature or any change touching several files, when requirements are vague or only exist as an idea, when a request bundles several independently shippable capabilities, or when asked for a spec, a PRD or requirements.
---

# Spec first

Write the specification before the code. The spec is the shared source of truth: what is
being built, why, and how anyone will know it is done. Code without one is a guess that
happens to compile.

## When to skip this

A one-line fix. A typo. A change where the requirement is unambiguous and self-contained.
A spec for those is ceremony.

Everything else: a new feature, a change across several files, an architectural decision,
anything that would take more than half an hour. A simple task does not need a *long*
spec, but it still needs acceptance criteria, and a two-line spec is a real spec.

## The gate

```
SCOPE CHECK -> SPECIFY -> PLAN -> TASKS -> IMPLEMENT
                  |         |       |
               approval  approval approval
```

**Do not advance a phase without approval.** The whole value is in the stop.

## Phase 0: scope check

Most requests describe one capability. If this one does, skip straight to Specify. This
phase is for the exception.

Decompose first when one requirement bundles several independently testable capabilities:

- It names distinct capabilities with their own consumers or their own data.
- The acceptance criteria cluster into groups that could ship and be verified separately.
- One capability could be cut or swapped without rewriting the others' requirements.

Then propose a map before writing any spec. A table and a build order, not a project plan:

```markdown
# Capability map: <initiative>

| Module | Responsibility | Depends on |
|---|---|---|
| identity | Accounts, sessions, SSO | none |
| billing | Plans, invoices, payments | identity |
| reporting | Usage dashboards | billing |

Build order: identity, then billing, then reporting
```

- **Stable ids.** Kebab-case, chosen once, never renamed mid-effort. Specs and tasks select
  work by id rather than by guessing which spec is current.
- **Dependencies point one way.** If two modules each need the other, they are one module.
- **Interfaces belong to the provider.** The map records that billing depends on identity;
  the contract between them lives in identity's spec.

The map is gated like every other phase. Getting module boundaries wrong is expensive and
reviewing ten lines is not.

Then run Specify through Implement per module, in dependency order.

## Phase 1: specify

### Surface the assumptions first

Before writing any spec content, list what is being assumed:

```
Assuming:
1. Flutter web is the only target that matters now
2. The server is the source of truth, nothing is cached locally
3. Money is parsed as an integer number of paise, never a double
4. Auth is the existing short-lived access token plus refresh cookie
Correct any of these now, or I proceed on them.
```

Do not silently fill an ambiguous requirement. The entire purpose of a spec is to surface
misunderstandings before they become code, and an unstated assumption is the most
dangerous kind of misunderstanding there is.

### Reframe vague requirements as criteria

```
Asked for: "make the dashboard faster"

Criteria:
- First panel renders under 500ms on a warm cache
- No layout shift when data lands: the skeleton matches the content's height
- Nine panels load concurrently, not sequentially
Are those the right targets?
```

This turns a judgement call into something that can be looped on and verified. "Faster"
cannot be finished; a number can.

### The spec

```markdown
# Spec: <name>

## Objective
What is being built and why. Who uses it. What success looks like.

## Tech stack
Framework, language, key dependencies with versions.

## Commands
Build, test, lint, dev. Full commands with flags, not tool names.

## Structure
Where source, tests and docs go.

## Code style
One real snippet from this codebase beats three paragraphs of description.

## Testing strategy
Framework, where tests live, which levels cover which concerns, what always gets a test.

## Boundaries
- Always: <run the checks before committing, follow the existing conventions>
- Ask first: <schema changes, new dependencies, CI changes>
- Never: <commit secrets, edit generated files, delete a failing test>

## Success criteria
Specific, testable conditions.

## Open questions
Anything still needing an answer.
```

Where the project already uses a specification system, keep its format and its storage
conventions instead of creating a parallel `SPEC.md`. This skill owns the clarification
and the gates; the existing tool owns how the spec is represented.

Where the effort is already mapped on the issue tracker, the spec is the collapse of those
decisions. See `atlas`, and link the issues rather than restating them.

### Then stop

Once the spec is saved:

1. Summarise it and list the open questions.
2. Ask for approval or changes.
3. **End the turn.** Do not start planning, do not write code, not even a little. Planning
   begins in a later turn, after approval.

This is the step most often skipped and the one that makes the rest work.

## Phase 2: plan

With an approved spec: the major components and their dependencies, the build order, the
risks and what to do about them, what can run in parallel against what must be sequential,
and the verification checkpoints between phases.

The plan has to be reviewable. Someone should be able to read it and say "yes, that is the
right approach" or "no, change this".

## Phase 3: tasks

- Each task finishes in one focused session.
- Each has explicit acceptance criteria.
- Each has a verification step: a command, a build, a check.
- Ordered by dependency, not by how important each feels.
- None touches more than about five files.

```markdown
- [ ] <task>
  - Acceptance: <what is true when it is done>
  - Verify: <the command or check that proves it>
  - Files: <what gets touched>
```

Where the tracker holds the work, each task is an issue with the effort's milestone.

## Phase 4: implement

One task at a time. Load the spec sections and source files each task needs rather than
the entire spec, for the reasons in `smart-zone`. Drive the tests first where the project
works that way. Review the diff before it merges with `code-quality`.

## Keeping the spec alive

- **A decision changes: update the spec first, then the code.** The other order produces
  documentation, not specification.
- **Scope changes: reflect it.** Features added or cut belong in the spec.
- **Commit it.** The spec lives in version control beside the code.
- **Link it from the PR**, section by section where the change is large.

An outdated spec still beats no spec, but a spec updated after the fact is just a
description of what happened.

## The rationalisations

| Said | Actually |
|---|---|
| "This is simple, it does not need a spec" | Simple things need short specs, not no specs. Two lines is fine. |
| "I will write the spec after the code" | That is documentation. The value was in forcing clarity first. |
| "A spec will slow this down" | Fifteen minutes of spec against hours of rework. |
| "The requirements will change anyway" | Which is why it is a living document. |
| "I know what I want" | Every clear request still carries implicit assumptions. The spec is how they surface. |
| "It is one feature, splitting it is overhead" | If the criteria cluster into separately shippable groups, every downstream task has to reason over the whole contract. A ten-line map is cheaper. |
| "I will decompose during planning" | Planning slices tasks inside a spec. By then the oversized spec already exists. |

## Red flags

- Code being written with no written requirement anywhere.
- Asking whether to start building before defining what done means.
- Implementing something no spec or task mentions.
- An architectural decision made and not recorded.
- The spec and the plan written in the same turn.
- One spec whose requirements span several independently shippable capabilities.

## Before implementing, confirm

- [ ] The spec covers objective, stack, commands, structure, style, testing, boundaries
- [ ] Success criteria are specific and testable
- [ ] Boundaries are written as always, ask-first, never
- [ ] The spec is saved in the repository
- [ ] The turn ended after saving it, and approval arrived in a later turn
- [ ] Where the request bundled capabilities, the map was approved before any module spec
- [ ] Every module spec traces to a module id in that map
