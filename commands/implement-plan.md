---
description: "Implement a plan file end-to-end: implement, tests-docs, commit, review-loop, tests-docs, commit, then write TEMP_N.md decision log. Usage: /implement-plan <path-to-plan.md>"
agent: implementer
---

Fully autonomous plan-implementation pipeline. You are the implementer agent: do NOT ask the user anything; make any needed decision yourself and record every decision (with rationale) for the decision log at the end.

## Setup
- Plan path = `$1` (relative to project root or absolute). If empty, or if `$1` does not exist, glob `.kilo/plans/*.md`, pick the most recently modified, and record that decision.
- Read the plan file fully, then read the repo's root `AGENTS.md` and follow its constraints (pre-commit definition, commit style, changelog rules, never-commit rules).
- Run `git status`. If there are uncommitted changes, commit them first as a baseline (`chore:` message matching `git log --oneline -10` style) and record the decision. Do NOT push.

## STEP 1 — Implement the plan
Implement every actionable item in order, marking items done as you go. Ambiguous item → decide the most reasonable interpretation, implement it, record the decision. Impossible item (contradicts architecture/security/repo rules) → skip it, record the reason, continue.

## STEP 2 — /tests-docs
Read `~/.config/kilo/commands/tests-docs.md` (resolve the home dir; if missing, read `/home/kirill/Development/easyoneweb-projects/kilo-code-addons/commands/tests-docs.md`) and follow it exactly. Do NOT commit.

## STEP 3 — /commit
Read `~/.config/kilo/commands/commit.md` (fallback path as above) and follow it. Override: if the message is ambiguous, do NOT ask — decide the conventional message from `git log --oneline -10` style and record it. Commits implementation + test/doc updates. Do NOT push.

## STEP 4 — /review-loop
Read `~/.config/kilo/commands/review-loop.md` (fallback path as above) and follow it (review recent commits, fix findings, commit fixes, repeat until clean, max 4 rounds). Overrides: tree is already clean (from Setup) — never ask about a dirty tree; if findings remain after 4 rounds, stop and record the open findings instead of asking. Do NOT push.

## STEP 5 — /tests-docs again
Read `~/.config/kilo/commands/tests-docs.md` (fallback path as above) and follow it again — verify tests still green after review fixes and update docs if the review fixes changed anything user-facing. Do NOT commit.

## STEP 6 — final /commit
Read `~/.config/kilo/commands/commit.md` (fallback path as above) and follow it. If nothing to commit, say so and record it. Otherwise commit with a conventional message. Do NOT push.

## STEP 7 — Decision log
Create `TEMP_N.md` in the project root: use `TEMP.md` if it does not exist, else the smallest free `TEMP_1.md`, `TEMP_2.md`, … Include: plan file path, date, per-step status (1–6), every decision made (decision + rationale + step), open issues/blockers, and the commit hashes produced.

## Final reply
Concise summary: what was implemented, steps run, decisions made (point to `TEMP_N.md`), open issues. Do NOT push.
