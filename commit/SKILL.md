# Commit Skill

Stage and commit all changes, then push.

## Steps

1. Run `git status` and `git diff` to understand what changed
2. Stage changed files by name (never `git add -A` or `git add .`)
3. Write a concise commit message (conventional commits format, subject ≤50 chars)
4. Commit and push to current branch
5. Report the commit SHA

## Rules

- Do NOT modify build commands, CLAUDE.md, or README unless the user explicitly asked
- Do NOT add files that look like secrets (.env, credentials, tokens)
- Do NOT amend existing commits — always create a new one
- Do NOT push if on main/master without confirming with the user first
