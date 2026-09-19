---
name: review-changelog
description: >
  Audit CHANGELOG.md's Unreleased section against the commits since the last
  tag: flags user-facing commits with no matching entry, entries that don't
  correspond to any real commit, and Keep-a-Changelog format violations. Read
  -only — reports findings, does not edit the file. Use before cutting a
  release, or whenever the user says "review the changelog", "check
  changelog", "audit changelog", or invokes `/review-changelog`.
---

## Overview

A CHANGELOG entry is a claim about what shipped. This skill checks that claim
against reality in both directions: real changes missing an entry, and
entries with no real change behind them.

## Step 1 — Find the range

```bash
git describe --tags --abbrev=0   # last release tag
git log <lastTag>..HEAD --oneline
```

## Step 2 — Read the Unreleased section

Extract `CHANGELOG.md` from `## [Unreleased]` down to the next `## [`
heading (whichever comes first — some repos keep it as a bare placeholder
with the pending entries in a sub-section, others list them directly under
it; match what this file already does, don't impose a shape on it).

## Step 3 — Check format

Only recognized Keep-a-Changelog subheadings: `### Added`, `### Changed`,
`### Deprecated`, `### Removed`, `### Fixed`, `### Security`. Flag:

- Bullets floating outside any subheading.
- Unknown/misspelled subheadings.
- A subheading with zero bullets under it (dead weight, remove it).

## Step 4 — Missing entries (commits with no CHANGELOG bullet)

For each commit in the range, judge whether it's user-facing (new API,
behavior change, bug fix, perf change, removed/deprecated API) vs internal
(refactor, test-only, docs, build/CI, chore). Flag user-facing commits that
have no corresponding Unreleased bullet. Use the commit's diff, not just its
subject line, when the subject is vague (`fix: bug`, `update`).

## Step 5 — Orphaned entries (bullets with no matching commit)

For each Unreleased bullet, try to find the commit(s) that introduced it
(match by subject, by touched files/symbols named in the bullet, or by an
issue/PR/commit-SHA reference the bullet already carries). Flag any bullet
you can't tie to a real commit in range — likely stale (left over from a
reverted change) or drafted ahead of the actual commit.

## Step 6 — Reference convention

Check whether already-released sections in this file link entries to
issues/PRs/commits (`(#123)`, `([#123](url))`, `(a1b2c3d)` — conventions
differ per repo, sniff it from history rather than assuming one). If the
repo has such a convention, flag Unreleased bullets missing it.

## Step 7 — Report

```
CHANGELOG review: <N> commits since <lastTag>

Missing entries:
  - <commit sha> <subject> — looks user-facing, no Unreleased bullet found

Orphaned entries:
  - "<bullet text>" — no matching commit found in range

Format issues:
  - <line>: <what's wrong>

Reference convention: <e.g. "(#NNN) on every bullet in prior sections">
  - <bullet text> — missing reference
```

If everything checks out, say so plainly — don't pad a clean result with
manufactured nitpicks.

## Rules

- Read-only. Report findings; only edit `CHANGELOG.md` if the user then asks
  you to fix something.
- Don't flag internal/chore commits for missing entries — Keep-a-Changelog
  entries are for users of the library, not contributors.
- Don't invent bullet text when reporting a gap — name the commit and let the
  user (or a follow-up request) write the entry.

## When NOT to use

- Mid-development, between arbitrary commits on a feature branch — too noisy
  before the work has settled. Best run right before cutting a release or
  opening the release PR.
