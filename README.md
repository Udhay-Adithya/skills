# skills

Agent skills I use, in [SKILL.md](https://code.claude.com/docs/en/skills) format. They
work in Claude Code, and in any other agent that reads the same format.

## Install

All of them, globally:

```bash
npx skills add Udhay-Adithya/skills -g --skill '*'
```

One of them:

```bash
npx skills add Udhay-Adithya/skills -g --skill design-engineering
```

See what is in here without installing anything:

```bash
npx skills add Udhay-Adithya/skills --list
```

Global skills land in `~/.claude/skills` and are available in every project. Drop `-g` to
install into the current project instead, at `.claude/skills`.

Update later:

```bash
npx skills update -g -y
```

## What is in here

Nineteen skills. `which-skill` is the router: ask it and it names the flow to take.

### Building

| Skill | For |
| --- | --- |
| `design-engineering` | Any interface, in Flutter or on the web. Motion that earns its place, concentric radii, optical alignment, shadows for elevation, theme-driven colour, real hit areas, keyboard paths. Reads the project's own stack and tokens first. Six references, one per concern and one per stack |
| `flutter-riverpod-mvvm` | Flutter architecture: four layers per feature, `Either<Failure, T>` out of repositories, a sealed failure hierarchy, exhaustive four-state UI, no service locator |
| `spec-first` | A gated spec before any code: assumptions surfaced, vague requirements reframed as testable criteria, a hard stop before planning |
| `release` | Deriving a semver bump from the Conventional Commits in a tag range, then landing it as its own commit |

### Reviewing

| Skill | For |
| --- | --- |
| `code-quality` | Five axes over a diff, with a mutation check on the tests and a named remedy for anything structural |
| `security-review` | Vulnerabilities that cross a real boundary and produce a concrete result. No finding without both |
| `crucible` | Rounds of adversarial review by a model that is not the one that wrote the code, on a strict budget |
| `quorum` | Three reviewers from three providers, reconciled into one verdict with scores per round |

### Moving a pull request

`autopilot` runs the whole sequence. The rest work on their own.

| Skill | For |
| --- | --- |
| `autopilot` | The whole run unwatched, from current state to a merged PR, proving every change |
| `brink` | The same run, stopping with the PR ready and unmerged |
| `dispatch` | Filing a PR whose title and description say why the change matters |
| `vigil` | Keeping watch through review and CI, handling bot findings until it is ready |
| `revive` | Judging whether a stale PR still belongs in the product before rebasing it |
| `inherit` | Taking over someone else's unfinished work while preserving their credit |
| `remark` | A PR comment or dismissal that leads with its conclusion and cites evidence |

### Working

| Skill | For |
| --- | --- |
| `which-skill` | Which of these fits the situation, and in what order |
| `atlas` | Work too big for one session, as GitHub issues: a tracking issue that indexes everything, real blocking relationships, a milestone and a board |
| `smart-zone` | Staying inside the window where the model still reasons sharply, and a paste-ready prompt when it runs out |
| `prune` | The documents agents read: skills, `CLAUDE.md`, `AGENTS.md`, `NOTES.local.md`. Finds duplicated meaning, stale lines, no-ops and sprawl, and cuts them |

## Invoking them

Type a slash and the name:

```bash
/which-skill
```

Anything after the name is passed through as arguments. Explicit invocation is rarely
necessary: each skill's `description` carries its trigger phrases, so a plain request
reaches the right one. "Cut a release" finds `release`, "this screen feels off" finds
`design-engineering`, "watch the PR" finds `vigil`.

Two are invocation-only, because they route rather than act: `which-skill` and anything
that would otherwise fire on every turn.

## What is not here

Nothing enforces the writing style, the commit format or the attribution rules. Those live
in a global `CLAUDE.md`, because they have to hold on every line of every turn, and a
skill that fails to trigger is a rule that silently stopped applying.

## Attribution

Sixteen of these started as someone else's, from Theo Browne, Matt Pocock, Addy Osmani and
Cloudflare, all MIT. [NOTICE.md](NOTICE.md) records what came from where, what changed,
and the places where I deliberately disagree with the original.

## Licence

[MIT](LICENSE).
