---
name: release
description: Derive the next semantic version from the Conventional Commits since the last tag, then land the bump as its own commit and tag it. Use when asked to cut a release, bump the version, tag a release, or what the next version should be.
---

# Release

Derive the version, report it, then wait. Never bump a version unprompted.

## 1. Find the last release

```bash
git fetch --tags
git describe --tags --abbrev=0
git log "$(git describe --tags --abbrev=0)"..HEAD --oneline --no-merges
```

With no tags yet, the range is the whole history and the first release is `0.1.0`
unless told otherwise.

## 2. Derive the bump

Read the Conventional Commit type of every commit in that range:

| Found in the range | Bump |
| --- | --- |
| Any `!` before the colon, or a `BREAKING CHANGE:` footer | major |
| Otherwise, any `feat` | minor |
| Otherwise (`fix`, `perf`, `refactor`, `docs`, `test`, `build`, `ci`, `chore`, `style`, `revert`) | patch |

Below `1.0.0` the API is unstable, so a breaking change may land in a minor bump
instead of taking the project to `1.0.0`. Going to `1.0.0` is a deliberate decision
about stability, never a side effect of a commit type. From `1.0.0` on, a breaking
change needs a major bump.

Pre-release and build metadata follow the spec: `1.4.0-beta.1`, `1.4.0+20260115`.

A commit whose message does not parse as a Conventional Commit is a problem worth
naming. Do not silently treat it as a patch: list it and say the bump was derived
without it.

## 3. Report and stop

Say the derived version, the bump level, and the commits that drove it, with the
breaking ones called out. Then wait for a decision. The derivation is the deliverable;
the bump is a separate instruction.

## 4. Land the bump

Once approved, the bump is its own commit and touches nothing else:

```
chore(release): 1.4.0
```

Update every place the version appears, in that same commit:

- the manifest (`package.json`, `pubspec.yaml`, `Cargo.toml`, `pyproject.toml`)
- the lockfile, by running the package manager rather than editing it by hand
- any version constant in source
- any version in the README or the docs

Search for the old version string across the tree before committing, so nothing is
left behind:

```bash
git grep -nF "1.3.0" -- . ':!*.lock' ':!CHANGELOG.md'
```

Never fold a bump into a feature commit.

## 5. Build numbers

Where the platform has a build number separate from the version, increment it
monotonically on every release and never reset it. In `pubspec.yaml` that is the `+N`
suffix: `1.4.0+37` follows `1.3.0+36`. Android `versionCode` and iOS `CFBundleVersion`
work the same way, and a reset there is rejected by the store after the fact, which is
the expensive way to find out.

## 6. Tag

```bash
git tag v1.4.0
```

Tag the bump commit, prefix the tag with `v`, and leave the version inside the manifest
unprefixed. Do not push the tag unless pushing was asked for.

## Rules

- The bump commit goes on a branch like anything else, never straight onto `main`.
- A changelog, where the project keeps one, is generated from the same commit range and
  lands in the bump commit.
- No tool or model attribution in the commit message or the tag annotation.
