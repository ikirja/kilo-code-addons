---
description: "Explore docx files, extract ecosystem findings, categorize into bugs/updates/feature suggestions, write app plan + TEMP.md. Usage: /analyze-docs [path]"
---

Analyze docx documents (meeting notes, briefs, specs) and turn them into actionable work items for the current app and the ecosystem.

1. Find the documents:
   - If `$1` is provided, glob that path for `*.docx`, `*.doc`, `*.pdf`.
   - Otherwise glob `temp/**/*.{docx,pdf,doc}` and `temp-docs/**/*.{docx,pdf,doc}` in the current repo.

2. Extract the text of each document:
   - `.docx` is a zip archive; extract `word/document.xml` and strip XML tags, e.g. via python3 (`zipfile` + `re`), or use `pandoc -t plain <file>` if installed.
   - `.pdf`: use `pdftotext` if available, otherwise read via the Read tool.
   - `.doc` (legacy binary): convert first with `libreoffice --headless --convert-to docx`, then extract as `.docx`; if conversion is unavailable, read via the Read tool.

3. Load the `ecosystem-analysis` skill via the `skill` tool and follow it for: identifying the current app vs. sibling ecosystem repos, categorizing findings into bugs/updates/feature suggestions, and writing `.kilo/plans/<epoch-ms>-<slug>.md` (current app) plus `.kilo/plans/TEMP.md` (other apps/websites).

4. Summarize in the reply: per category, what was found for the current app, the other-app repos, and the paths of the two files written.
