---
description: "Bump version (CHANGELOG + package.json), run pre-commit, commit. Usage: /release [patch|minor|major|X.Y.Z] (default patch). Never modifies package-lock.json"
---

Create a release: version bump in `package.json` + `CHANGELOG.md`, validate, commit. Do NOT touch `package-lock.json` — the user's constraint, containers will not build otherwise.

1. Read the root `AGENTS.md` — the "Changelog / When Releasing" section defines this repo's exact steps and commit message style.

2. Determine the new version:
   - `$1` may be `patch`, `minor`, `major`, or a literal `X.Y.Z`.
   - If `$1` is empty, default to `patch`.
   - Read the current version from `package.json` and compute the target (semver bump or the literal).

3. `CHANGELOG.md` (Keep a Changelog):
   - Move the `[Unreleased]` content into a new section `[X.Y.Z] - YYYY-MM-DD` (today's date). Leave `[Unreleased]` as an empty section for the next cycle.
   - Update the link references at the bottom: change `[unreleased]: <repo>/compare/vX.Y.Z...HEAD` and insert `[X.Y.Z]: <repo>/compare/v<PREV>...vX.Y.Z` above the previous version link. Derive `<repo>` from the existing links.

4. Bump the `package.json` version WITHOUT changing `package-lock.json`:
   - Preferred: edit `package.json` directly, e.g.
     `node -e "const fs=require('fs');const p=JSON.parse(fs.readFileSync('package.json','utf8'));p.version=process.argv[1];fs.writeFileSync('package.json',JSON.stringify(p,null,2)+'\n');" X.Y.Z`
   - Alternatively run `npm version X.Y.Z --no-git-tag-version` and then IMMEDIATELY restore the lockfile (`git restore package-lock.json` or `git checkout -- package-lock.json`) if it is tracked.
   - Before committing, verify with `git status` / `git diff` that `package-lock.json` is NOT modified.

5. Run `npm run pre-commit` to validate everything is green.

6. Commit with the repo's version-bump style derived from `git log --oneline -10`: `release: bump version to X.Y.Z` (rankup-app, landings) or `chore: bump version to X.Y.Z` (widget-pro, orbitron-hub) — match what this repo actually uses.

7. Do NOT create or push a tag — that is the `/tag-push` step.
