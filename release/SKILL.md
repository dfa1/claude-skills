---
name: release
description: >
  Cut a release for a Java FFM Maven project (rocksdb-ffm, zstd-ffm, lmdb-ffm,
  and similar repos following the same shape). Runs generic preflight checks,
  then follows THIS repo's own documented release process for the actual
  version-bump/tag mechanics (they differ per repo — never assume one flow),
  confirms before any push, and monitors the publish workflow. Triggers when
  the user says "release", "cut a release", "tag a release", "release X.Y",
  or invokes `/release`.
---

## Overview

The generic parts (preflight, confirm-before-push gates, monitoring) are the
same across every repo in this family. The version-bump mechanics are **not**
— they diverge per repo:

- `rocksdb-ffm` / `lmdb-ffm`: `mvn release:prepare` (auto tag + auto snapshot bump).
- `zstd-ffm`: deliberately manual (`versions:set`, hand-edit CHANGELOG, hand tag)
  — its pom.xml explains why `release:prepare` doesn't fit there.
- Whether `publish.yml` also creates the GitHub release varies (it does for
  `zstd-ffm`/`lmdb-ffm`, not for `rocksdb-ffm` as of 2026-09).

**Never hardcode a flow.** Step 1 below always re-derives it from the repo
itself, because it can change without this skill being updated.

## Step 1 — Find this repo's documented process

Look for a `<!-- Cut a release ... -->` comment block near the `release`
profile in the root `pom.xml`:

```bash
grep -n "Cut a release" -A 30 pom.xml
```

If it's missing or ambiguous, check `CLAUDE.md` for a "Releasing" section.
If neither exists, **stop and ask the user** — do not guess between
`release:prepare` and a manual `versions:set` flow; they are not
interchangeable and picking wrong leaves the repo in a half-tagged state.

Also check whether `.github/workflows/publish.yml` has a "Create GitHub
release" step (`gh release create` / awk-extracting `CHANGELOG.md`) — this
tells you whether a GH release happens automatically after the tag push.

## Step 2 — Preflight

Refuse to proceed unless **all** hold. Report any failure and stop:

- Current branch is `main`.
- Working tree is clean (`git status --short` empty).
- `main` is up to date with `origin/main` (`git fetch origin`, zero commits
  each direction).
- The full build is green — use whatever command CLAUDE.md mandates
  (`./mvnw verify`, `./mvnw package`, etc.). **Never `mvn install`** — every
  repo in this family forbids it.
- `gh auth status` succeeds.
- The target tag doesn't already exist: `git tag -l "v<version>"` empty.

## Step 3 — CHANGELOG

Find `## [Unreleased]` in `CHANGELOG.md`. These repos use the permanent-empty-
heading shape: `## [Unreleased]` stays as a bare placeholder, and a new
`## [X] - <date>` (or `— <date>`, match the file's existing dash) section is
inserted directly below it, listing what actually shipped. Pull the entries
from what's already accumulated there — don't invent from `git log`; if
`review-changelog` hasn't been run recently, run it first to catch gaps.

## Step 4 — Execute the version-bump mechanics

Follow exactly what Step 1 found in *this* repo's pom.xml/CLAUDE.md. Show the
user the commands you're about to run before running them. Two known shapes
so far — use whichever the repo actually documents:

**`release:prepare` repos:**
```bash
./mvnw --batch-mode release:clean release:prepare \
    -DreleaseVersion=<version> \
    -DdevelopmentVersion=<next>-SNAPSHOT
```
If it fails partway, `./mvnw release:rollback` before retrying.

**Manual repos (e.g. `zstd-ffm`):**
```bash
./mvnw versions:set -DnewVersion=<version>
# commit, edit CHANGELOG.md, commit "release: <version>"
git tag v<version>
```
Followed by a *separate* post-release commit bumping back to
`<next>-SNAPSHOT` — don't skip this; a repo left on a released (non-SNAPSHOT)
version number on `main` is indistinguishable from a release commit itself.

## Step 5 — Push (confirm first)

Show the new commit(s) and tag, then get explicit user OK before:

```bash
git push && git push --tags
```

## Step 6 — Monitor

```bash
gh run list --workflow=publish.yml --limit 1
gh run watch <run-id>
```

## Step 7 — Verify

- Maven Central: `curl -fsSI https://repo1.maven.org/maven2/<groupId path>/<artifactId>/<version>/...pom`
  (may take 10–30 min; not a hard gate).
- GitHub release: only check/expect one if Step 1 found the workflow creates
  it automatically. If this repo doesn't automate it (e.g. `rocksdb-ffm`
  today), say so and ask the user whether they want one created by hand —
  don't silently add automation that wasn't there.
- If the repo's flow requires a manual post-tag snapshot bump (Step 4,
  manual shape), do it now and push.

## Rules

- Never force-push or delete tags. A broken release gets a new patch version.
- Never skip Maven hooks (`--no-verify`, `--no-gpg-sign`) — signing is
  required for Maven Central.
- Never invent CHANGELOG entries.
- Confirm with the user before: any `git push`, any `gh release create` run
  by hand, and any retry of a failed release step.
- Stop on first preflight failure.

## When NOT to use

- Hot-patching an already-published version — cut a new patch instead.
- Releasing from a branch other than `main`.
- A repo whose pom.xml/CLAUDE.md documents no release process at all — ask
  the user first rather than inventing one.
