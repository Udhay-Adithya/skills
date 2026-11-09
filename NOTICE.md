# Attribution

Some skills here started as someone else's work. This file records where each came
from, what its licence is, and what changed. The originals are worth reading.

## Theo Browne, [t3dotgg/tstack](https://github.com/t3dotgg/tstack)

MIT, Copyright (c) 2026 Theo Browne. A copy of the licence is in
`licences/tstack-LICENSE`.

Nine skills derive from that repository, renamed and adapted:

| Here | Upstream |
| --- | --- |
| `autopilot` | `full-send` |
| `brink` | `full-stop` |
| `crucible` | `sol-loop` |
| `dispatch` | `file-pr` |
| `inherit` | `takeover` |
| `quorum` | `council` |
| `remark` | `leaving-pr-comments` |
| `revive` | `modernize` |
| `vigil` | `babysit-pr` |

`skills/vigil/scripts/pr-wait` is Theo's, carried over unmodified.

### What changed

**Harness.** Upstream targets T3 Code and calls its tools directly
(`watch_pull_request`, `unwatch_pull_request`, `orchestrator_capabilities`,
`delegate_task`). Those are gone, replaced with the Claude Code equivalents: the
desktop app's own PR monitor and Auto-fix for watching, the `Agent` tool for
delegation, `AskUserQuestion` for questions. `pr-wait` stays as the fallback watcher
everywhere else, run under the Monitor tool.

**Attribution.** Upstream ends a PR description with the model and harness that wrote
the change, and prefixes every agent-authored comment with a note naming the model.
Both are removed here, because my own instructions say nothing committed or posted may
indicate how it was written. This is a real disagreement with upstream rather than an
oversight: Theo wants the disclosure, I do not. `remark` says how to put it back.

**`crucible`.** Upstream pins the reviewer to OpenAI's GPT Sol and will not substitute.
Here the reviewer is a tier list, the requirement being that it is not the model doing
the work, with the tier named in the report so a weak round is visible. The loop
structure (scope, budget, brief, ledger, stop condition) is Theo's and is the part that
matters.

**Naming.** Renamed to single words that say what the skill does, so nine PR skills do
not all begin with `pr-` or `full-`. Cross-references between the skills follow the new
names.

**Style.** Reflowed to my own writing rules.
