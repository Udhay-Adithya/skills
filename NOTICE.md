# Attribution

Some skills here started as someone else's work. This file records where each came from,
what its licence is, and what changed. The originals are worth reading.

Every source below is MIT, and a copy of each licence is in `licences/`.

## Theo Browne, [t3dotgg/tstack](https://github.com/t3dotgg/tstack)

Copyright (c) 2026 Theo Browne. Licence: `licences/tstack-LICENSE`.

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
`delegate_task`). Those are gone, replaced with Claude Code equivalents: the desktop app's
own PR monitor and Auto-fix for watching, the `Agent` tool for delegation,
`AskUserQuestion` for questions. `pr-wait` stays as the fallback watcher everywhere else.

**Attribution.** Upstream ends a PR description with the model and harness that wrote the
change, and prefixes every agent-authored comment with a note naming the model. Both are
removed here, because my own instructions say nothing committed or posted may indicate how
it was written. This is a real disagreement with upstream rather than an oversight: Theo
wants the disclosure, I do not. `remark` says how to put it back.

**`crucible`.** Upstream pins the reviewer to OpenAI's GPT Sol and will not substitute.
Here the reviewer is a tier list whose only hard requirement is that it is not the model
doing the work, with the tier named in the report so a weak round is visible. The loop
structure (scope, budget, brief, ledger, stop condition) is Theo's and is the part that
matters.

**Naming.** Renamed to single words that say what each does, so nine PR skills do not all
begin with `pr-` or `full-`.

## Matt Pocock, [mattpocock/skills](https://github.com/mattpocock/skills)

Licence: `licences/mattpocock-LICENSE`.

| Here | Upstream |
| --- | --- |
| `prune` | `productivity/writing-for-agents` |
| `which-skill` | `engineering/ask-matt` |
| `atlas` | `engineering/wayfinder` |
| `smart-zone` | `engineering/ask-matt` (context hygiene), `productivity/handoff` |

**`prune`** keeps the part that made the original worth having: the four cuts (duplicated
meaning, stale lines, no-ops, sprawl), the information hierarchy, pointers as the thing
that decides whether material is ever reached, and prompting the positive instead of the
prohibition. Dropped: the leading-words section, which is upstream's best idea and needs
its own treatment rather than a paragraph. Added: a section on `NOTES.local.md`, which is
where my own caveats and open questions go, and which prunes by resolution rather than by
length.

**`atlas`** is upstream's wayfinder made GitHub-native. Upstream is deliberately
tracker-agnostic and defaults to local markdown; this one commits to GitHub issues, with
real sub-issue and blocking relationships, a milestone for progress and a project board
for state, since that is how I already work. The concepts (the map as an index not a
store, fog of war, the issue-or-fog test, out of scope never graduating, refer by name not
number, one ticket per session) are upstream's and are the reason it works.

**`smart-zone`** takes the [smart zone](https://www.aihero.dev/ai-coding-dictionary/smart-zone)
term and the five phase-boundary options from upstream, and adds what was missing for me:
how to tell where you are in the window when the token count cannot be read, and a
paste-ready handoff prompt with the sections that are expensive to rediscover, which is
the decisions and the dead ends rather than the state.

**`which-skill`** is the same idea as ask-matt, routing over my skills rather than Matt's.
The useful structure, a main flow with on-ramps and a vocabulary layer underneath, is his.

## Addy Osmani, [addyosmani/agent-skills](https://github.com/addyosmani/agent-skills)

Licence: `licences/addyosmani-LICENSE`.

| Here | Upstream |
| --- | --- |
| `spec-first` | `spec-driven-development` |
| `code-quality` | `code-review-and-quality` |

**`spec-first`** keeps the gated workflow, the phase 0 capability map, surfacing
assumptions before writing anything, reframing vague requirements as testable criteria,
the rationalisations table and the hard stop after the spec. Trimmed the cross-references
to a dozen sibling skills I do not have, and pointed the tracker parts at `atlas`.

**`code-quality`** keeps the five axes, the approval standard, the structural remedies,
change sizing, the severity prefixes and the mutation check on the tests, which is the
best single idea in it: invert a condition the change introduces, run the suite, and a
green result names a missing test. Trimmed the references to sibling skills and pointed
the security axis at `security-review`.

## Cloudflare, [cloudflare/security-audit-skill](https://github.com/cloudflare/security-audit-skill)

Licence: `licences/cloudflare-LICENSE`.

`security-review` is a distillation, not a port. Upstream is roughly 2000 lines across a
SKILL.md and twelve companion files, built for multi-agent audits of large targets: a
deterministic coverage ledger, isolated hunter waves, a findings schema with validators,
and a race-safe procedure for promoting evidence files out of a sandbox.

Kept, because it is the conceptual core and it travels: requiring a named boundary and a
concrete result before anything counts as a finding, bounded local evidence, respecting
what source can and cannot show, needs-validation as a first-class state with no severity,
the severity anchors and the high-against-medium discriminator, the smallest effective fix
at the last trusted decision point, and the anti-patterns.

Dropped, because a solo reviewer on a small target cannot use it and would only skip it:
the ledger, the agent write-isolation machinery, the budget arithmetic and the run
profiles. The skill points at upstream for those rather than pretending to replace them.

Added: the places worth looking in my own stack, ordered by how often each actually
yields something.

## Not vendored

The design engineering skills I use are MIT and could be redistributed, but they are not
here, because `npx skills update` keeps them current and a fork would not. Install them
directly:

```bash
npx skills add emilkowalski/skill -g --skill '*'
npx skills add jakubkrehel/skills -g --skill '*'
```

`design-engineering` is my own distillation of what I actually reach for across those two
sets, plus Addy Osmani's `frontend-ui-engineering`, written so it stands alone and covers
both of my stacks. It owes them its content and copies none of their text.
