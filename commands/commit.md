---
description: Run the pre-commit command, fix failures, commit with a conventional message
---

Run the repo's pre-commit validation, fix everything it finds, and commit the current changes.

1. Read the root `AGENTS.md` for this repo's conventions (pre-commit definition, commit message style, what must never be committed).

2. Run `git status` and `git diff`. If there is nothing to commit, say so and stop.

3. Run `npm run pre-commit` (or the repo's pre-commit script per `AGENTS.md`). Fix every failure — lint errors, formatting, failing tests, build errors — by editing the code, then re-run until it passes. Note: some repos' pre-commit scripts auto-format or auto-fix files; re-check `git status` after running.

4. Derive a conventional commit message from the diff using the style seen in `git log --oneline -10` (e.g. `fix:`, `feat:`, `refactor:`, `test:`, `docs:`, `chore:`). If the message is ambiguous, propose one and ask the user to confirm before committing.

5. Stage only the files that belong to the change — never secrets, `node_modules/`, `dist/`, or lockfiles unless intentionally changed — and commit.

6. Report the commit hash and a one-line summary. Do not push.
