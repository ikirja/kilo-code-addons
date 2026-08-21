---
description: "Review the last N commits, fix all findings, run pre-commit, repeat until clean. Usage: /review-loop [N] (default 1)"
---

Run an iterative code review loop over the last commits in this repository until no review findings remain.

## Setup

- `N` = `$1` if provided, else `1`.
- Read the root `AGENTS.md` first and follow its constraints (commit style, changelog rules, DB/migration rules, path aliases) and its definition of the pre-commit command.
- Check `git status`. If there are uncommitted changes, ask the user whether to commit them first or exclude them. Do not review over a dirty tree unless the user says so.

## Loop (max 4 rounds)

Each round:

1. Review the last N commits: run `git log --oneline -N`, then read the full diff with `git diff HEAD~N..HEAD` plus `git show <sha>` per commit, and the surrounding code context via the Read/Grep tools. If N exceeds the total commit count (`git rev-list --count HEAD`), clamp N to that count. Skip generated/vendor files: `package-lock.json`, `node_modules/`, build output such as `dist/`.

2. Hunt for real defects, not style: correctness bugs, security issues, race conditions, null/undefined handling, edge cases, DB/schema/migration correctness, error handling that bypasses the repo's error pipeline, missing or outdated tests. Cross-check the sharp edges in the repo's `AGENTS.md` (e.g. no `drizzle-kit push`, schema single-source rules).

3. If there are findings: fix ALL of them in the working tree, run `npm run pre-commit` until it is green, then commit the fixes with a conventional message matching the repo log style (e.g. `fix: address review findings from <topic>`). Then set `N = N + 1` (the fix commits join the reviewed range) and run another round.

4. If there are no findings in the current range: stop the loop.

## Stopping

- If findings are still produced after 4 rounds, stop looping — do not churn. Present the remaining findings to the user and let them decide.
- At the end, summarize: which commits were reviewed, what was fixed (list), and anything still open. Do not push.
