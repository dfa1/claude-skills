# claude-skills

Personal [Claude Code](https://claude.com/claude-code) skills, cloned to
`~/.claude/skills` on each machine so they're available in every project
automatically — no per-project setup.

Personal use only, not meant for reuse by others as-is (paths, conventions,
and assumptions are specific to my own repos).

## Setup on a new machine

```bash
git clone https://github.com/dfa1/claude-skills.git ~/.claude/skills
```

## Skills

- `commit` — stage, commit, push with basic guardrails.
- `improve-performance` — JMH + JFR iterative performance optimization workflow.
- `review-performance` — code review checklist for hot-loop/FFM (`MemorySegment`)
  memory correctness and zero-copy I/O.
- `release` — cut a release for a Java FFM Maven project; re-derives the actual
  version-bump steps from that repo's own docs instead of hardcoding one flow.
- `review-changelog` — audits `CHANGELOG.md`'s Unreleased section against the
  commits since the last tag.
