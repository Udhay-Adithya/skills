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
npx skills add Udhay-Adithya/skills -g --skill flutter-riverpod-mvvm
```

See what is in here without installing anything:

```bash
npx skills add Udhay-Adithya/skills --list
```

Global skills land in `~/.claude/skills` and are available in every project. Drop `-g`
to install into the current project instead, at `.claude/skills`.

Update later:

```bash
npx skills update -g -y
```

## What is in here

### Writing code

| Skill | For |
| --- | --- |
| `flutter-riverpod-mvvm` | Flutter on Riverpod and MVVM: four layers per feature, `Either<Failure, T>` out of repositories, a sealed failure hierarchy, exhaustive four-state UI, no service locator |
| `release` | Deriving the next semantic version from the Conventional Commits since the last tag, then landing the bump and the tag |

### Moving a pull request

These chain into each other. `autopilot` runs the whole sequence; the rest are useful
on their own.

| Skill | For |
| --- | --- |
| `autopilot` | The whole run unwatched, from current state to a merged PR, proving every change |
| `brink` | The same run, stopping with the PR ready and unmerged |
| `crucible` | Rounds of adversarial review by a model that is not the one doing the work, on a strict budget |
| `quorum` | Three reviewers from three providers, reconciled into one verdict with scores per round |
| `dispatch` | Filing a PR whose title and description say why the change matters |
| `vigil` | Keeping watch through review and CI, handling bot findings until it is ready |
| `revive` | Judging whether a stale PR still belongs in the product before rebasing it |
| `inherit` | Taking over someone else's unfinished work while preserving their credit |
| `remark` | Writing a PR comment or dismissal that leads with the conclusion and cites evidence |

## Invoking them

Type a slash and the name:

```bash
/crucible
```

Anything after the name is passed through as arguments. Explicit invocation is rarely
necessary though: each skill's `description` carries its trigger phrases, so a plain
request reaches the right one. "Cut a release" finds `release`, "watch the PR" finds
`vigil`, a question about where a Dart file belongs finds `flutter-riverpod-mvvm`.

## Attribution

Nine of these started as Theo Browne's, from
[t3dotgg/tstack](https://github.com/t3dotgg/tstack), MIT. [NOTICE.md](NOTICE.md)
records what came from where and what changed, including the two places where I
deliberately disagree with the originals.

## Licence

[MIT](LICENSE).
