---
description: "Explore video files, extract ecosystem findings, categorize into bugs/updates/feature suggestions, write app plan + TEMP.md. Usage: /analyze-video [path]"
---

Analyze video files (screen recordings, product walkthroughs) and turn them into actionable work items for the current app and the ecosystem.

1. Find the videos:
   - If `$1` is provided, glob that path for `*.mp4`, `*.webm`, `*.mov`, `*.avi`, `*.mkv`.
   - Otherwise glob `temp/**/*.{mp4,webm,mov,avi,mkv}` and `temp-docs/**/*.{mp4,webm,mov,avi,mkv}` in the current repo.

2. Read every video with the vision skill: load the `vision` skill via the `skill` tool and run its pipeline for each video (one video per API call; requires ffmpeg for video). Relay/extract the descriptions.

3. Load the `ecosystem-analysis` skill via the `skill` tool and follow it for: identifying the current app vs. sibling ecosystem repos, categorizing findings into bugs/updates/feature suggestions, and writing `.kilo/plans/<epoch-ms>-<slug>.md` (current app) plus `.kilo/plans/TEMP.md` (other apps/websites).

4. Summarize in the reply: per category, what was found for the current app, the other-app repos, and the paths of the two files written.
