---
description: Check tests and update stale ones; audit maintained docs (incl. CHANGELOG) and update if needed
---

Verify and update tests and maintained documentation.

1. Read the root `AGENTS.md` — it defines the test/check commands, the changelog rules, and which docs are maintained for this repo.

2. Run the test suite per `AGENTS.md` (typically `npm test`; also run `npm run check` and `npm run lint` if the repo defines them). If a test fails:
   - Determine whether the code behavior intentionally changed. If so, the test expectation is stale — update the test to match the intended behavior.
   - If it is a real regression, fix the bug instead of weakening the test. Never delete or disable tests just to make them pass.

3. Audit the maintained docs: `README.md`, `AGENTS.md`, `CLAUDE.md`, `CHANGELOG.md`, and the `docs/` directory. Update only what is actually stale or missing:
   - New/changed environment variables, new endpoints or routes, changed architecture or behavior, new commands.
   - `CHANGELOG.md`: add notable user-facing/API/infra changes to the `[Unreleased]` section using the repo's categories (Added/Changed/Fixed/...). Entries must be in English per the repo's changelog rules.

4. Do NOT commit — the user decides the next step.
