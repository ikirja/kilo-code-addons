---
description: "Create a vX.Y.Z git tag on HEAD and push it. Usage: /tag-push [version] (default: package.json version)"
---

Tag the current HEAD and push the tag to origin.

1. Determine the version:
   - `$1` if provided, otherwise read `version` from `package.json`.

2. Verify the tag does not exist yet: run `git tag -l "vX.Y.Z"`. If it exists, abort and tell the user.

3. Create the tag on HEAD: `git tag vX.Y.Z`.

4. Push the tag: `git push origin vX.Y.Z`.

5. In the reply:
   - Note that per the repo's `AGENTS.md`, pushed tags trigger CI/CD deployment (no separate production branch).
   - Check whether the branch is ahead of its remote (e.g. `git rev-list --count origin/main..HEAD`). If there are unpushed commits, tell the user the code commits were NOT pushed and ask whether they also want to push the branch.
