---
name: ecosystem-analysis
description: Shared rules for mapping findings to the ecosystem repos, categorizing into bugs/updates/feature suggestions, and writing the app plan + TEMP.md outputs. Used by the /analyze-video and /analyze-docs slash commands.
---

# Ecosystem Analysis Rules

Shared workflow used by the `/analyze-video` and `/analyze-docs` slash commands. Load this skill via the `skill` tool after extracting the raw findings from the media or documents.

## Current app vs. other apps

- The CURRENT app is the repo this session runs in.
- Other ecosystem repos: siblings of the current repo under the same parent directory (known projects: `rankup-app`, `widget-pro`, `orbitron-hub`, `rankup-landing`, `orbitron-landing`).
- Assign each finding to the app(s) it concerns. If a repo is not mentioned in the source material, say so explicitly.

## Categorization

- **bugs** — things that are broken or behave wrongly.
- **updates** — achievable with data/behavior already available (e.g. a new metric computed from existing DB data, a UI tweak).
- **feature suggestions** — require substantial new development.

## Outputs

- A comprehensive plan for the CURRENT app at `.kilo/plans/<epoch-ms>-<slug>.md`, covering all 3 categories with enough detail to implement (key files, decisions to resolve).
- A separate `.kilo/plans/TEMP.md` with the info for OTHER apps and websites, structured per repo (mirror the existing TEMP.md layout). Do not put current-app items in TEMP.md.
