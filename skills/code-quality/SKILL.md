---
name: code-quality
description: Review a change across correctness, readability, architecture, security and performance, with severity-labelled findings and structural remedies rather than complaints. Use before merging anything, after finishing a feature, when evaluating code written by another agent or model, after a bug fix, when refactoring, or when asked to review a diff, a branch or a PR.
---

# Code quality

Review every change before it merges. Five axes, findings labelled by what they require,
and a named remedy attached to anything structural.

**The standard for approval:** approve a change that definitely improves overall code
health, even where it is not perfect. Perfect code does not exist and the goal is
continuous improvement. Do not block something because it is not how you would have
written it. If it improves the codebase and follows the project's conventions, approve it.

## The five axes

### 1. Correctness

- Does it match the spec or the task?
- Are the edge cases handled: null, empty, zero, the boundary, the maximum?
- Are the error paths handled, not only the happy one?
- Do the tests pass, and are they testing the right thing?
- Any off-by-one, any race, any state that can go inconsistent?

### 2. Readability

- Are the names descriptive and consistent with the project? No bare `data`, `temp`,
  `result`.
- Is the control flow followable, without nested ternaries or deep callbacks?
- **Could this be shorter?** A thousand lines where a hundred would do is a failure, not
  thoroughness.
- **Is each abstraction earning its complexity?** Do not generalise before the third case.
- Is there dead code: unused variables, compatibility shims, commented-out blocks?
- **Is a new conditional bolted onto an unrelated flow?** That is a design smell, not a
  nit. Push the logic into its own helper or state instead of tangling a path that already
  works.
- **Do repeated conditionals on the same shape appear?** They mean a model is missing. A
  temporary branch is usually permanent.

### 3. Architecture

- Does it follow an existing pattern, or introduce a new one? If new, is that justified?
- Do dependencies flow one way, with no cycles?
- **Does this refactor reduce complexity or just move it?** Count the concepts a reader
  has to hold. If a cleaner version leaves that count unchanged, it is not cleaner. Prefer
  the restructuring that makes a whole branch or layer disappear over the one that
  re-centralises the same logic. Prefer deleting an abstraction to polishing it.
- **Is feature-specific logic leaking into a shared module?** Keep logic in the layer that
  owns it, reuse the canonical helper rather than a near-duplicate, and do not normalise
  drift because it is already there.
- **Are the type boundaries explicit?** Question every gratuitous `any`, `dynamic`, cast
  or silent fallback. Making the boundary explicit usually simplifies the control flow
  around it.

### 4. Security

- Is input validated at the boundary, before it reaches logic or a template?
- Are secrets out of the code, the logs and the history?
- Is authorisation checked where it is needed, not only authentication?
- Are queries parameterised?
- Is output encoded for where it is going?
- Is data from any external source, including the project's own API, treated as untrusted?

For anything that looks like a real vulnerability, stop reviewing and use
`security-review`. A finding needs a trust boundary and a result, not a bad smell.

### 5. Performance

- Any N+1 query?
- Any unbounded loop or unconstrained fetch?
- Any synchronous work that should be async, or sequential work that should be concurrent?
- Any missing pagination on a list endpoint?
- Any unnecessary rebuild or re-render?
- Any large object allocated in a hot path?

## Propose the move, not just the problem

A review that says "this is complex" leaves the author guessing. Name the restructuring:

- Replace a chain of conditionals with a typed model or an explicit dispatcher.
- Collapse duplicate branches into one flow.
- Separate orchestration from the logic it orchestrates.
- Move feature-specific logic out of a shared module into the package that owns it.
- Reuse the canonical helper instead of a bespoke near-duplicate.
- Make a type boundary explicit so the downstream branching disappears.
- Delete a pass-through wrapper that adds indirection and clarifies nothing.
- Extract a helper, or split a large file into focused modules.

Prefer the remedy that removes moving pieces over the one that spreads the same complexity
around.

## Process

### 1. Understand the intent

Before reading any code: what is this trying to accomplish, what spec or task does it
implement, what behaviour is expected to change.

### 2. Read the tests first

Tests show both intent and coverage. Do they exist? Do they test behaviour rather than
implementation? Are the edge cases there? Are the names descriptive?

**Then answer the last question by experiment, not by reading.** Invert one condition the
change introduces: drop a negation, swap `&&` for `||`. Run the suite. Restore the file.

A mutation that stays green is a finding. Name the missing case.

### 3. Walk the implementation

File by file, the five axes in mind.

### 4. Check the findings before reporting them

Check each candidate against the callers, the guards and the tests that actually reach the
changed code, and drop it if they rule it out. Label a defect that already existed in the
base separately from a regression this change introduces.

**Zero findings is a valid result.** Do not invent one to look thorough.

### 5. Label severity

| Prefix | Means | Author does |
|---|---|---|
| *(none)* | Required | Address before merge |
| **Critical:** | Blocks merge | Security, data loss, broken behaviour |
| **Nit:** | Optional | May ignore. Style, formatting |
| **Consider:** | Suggestion | Worth thinking about, not required |
| **FYI** | Information | Nothing to do |

Without this, authors treat every comment as mandatory and spend time on the nits.

**Lead with what matters.** Order by leverage: correctness and security, then structural
regressions and missed simplifications, then everything else. A few high-conviction
comments beat a long list. **With one structural problem and ten nits, the structural
problem is the review.**

### 6. Check the verification story

What was run. Did the build pass. Was it exercised by hand. Are there screenshots for a UI
change. Is there a before and after where it claims an improvement.

A change whose author cannot say how they know it works is not ready, whatever the diff
looks like.

## Change sizing

```
~100 lines   Good. Reviewable in one sitting.
~300 lines   Fine for a single logical change.
~1000 lines  Too large. Split it.
```

**Watch file size too, not only diff size.** A small diff can push a file past a healthy
boundary. Around 1000 total lines in one file is a signal worth acting on, not a hard cap.
When a change materially grows an already-large file, ask whether to extract first and add
second.

**One change** is one self-contained modification addressing one thing, with its tests,
leaving the system working. One part of a feature, not the whole feature.

Splitting, when it is too large:

| Strategy | How | When |
|---|---|---|
| Stack | Land a small change, start the next on top | Sequential dependencies |
| By file group | Separate changes for groups needing different reviewers | Cross-cutting |
| Horizontal | Shared code and stubs first, then the consumers | Layered architecture |
| Vertical | Smaller full-stack slices | Feature work |

**Refactoring and new behaviour in one change is two changes.** Submit them separately. A
variable rename riding along is fine.

Large is acceptable for a whole-file deletion or a mechanical refactor, where the reviewer
verifies intent rather than every line.

## Change descriptions

The first line stands alone in the history: short, imperative, and informative enough that
someone searching later understands it without the diff. "Delete the FizzBuzz RPC", not
"Deleting the FizzBuzz RPC".

The body carries what the code cannot: context, the decisions, the reasoning, the
shortcomings of the approach taken. Link the issue, the benchmark, the spec.

Never: "fix bug", "fix build", "address feedback", "phase 1", "moving code from A to B".

## Where a second opinion is worth it

One reviewer catches most of it. For anything touching auth, permissions, money, data
loss, or concurrent state, a reviewer that is not the model that wrote the code catches a
different class of problem, because it shares none of the assumptions that produced the
bug. Use `crucible` for the loop, or `quorum` for a panel.
