# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

An Obsidian vault (not a code project — there is no build, lint, or test) holding Royce Nobles' working notes as an Ohanafy leader. Ohanafy builds a Salesforce-managed-package ERP for beverage distributors. The current dominant focus is the **Gulf Go Live** (Gulf Distributing), so most new material relates to it.

The vault lives in iCloud Drive and is also a git repo. iCloud syncs the files; git keeps the history.

## Layout

- `Journal/YYYY/MM-MMMM/YYYY-MM-DD.md` is the daily note, created from `Templates/Daily.md`. Its sections are `# Objectives`, `# Meetings` (one `##` per meeting), and `# Notes`. The path format comes from `.obsidian/daily-notes.json`, so match it exactly (e.g. `Journal/2026/10-October/2026-10-15.md`).
- `Projects/<Project>/` holds projects. Sub-areas are nested folders (e.g. `Projects/Gulf Go Live/Integrations/`).
- `Architecture/<Domain>/` holds standing technical reference that isn't tied to a project (e.g. Salesforce packages).
- `Templates/` holds Obsidian templates. Don't put content notes here.

## Conventions

- **Folder notes:** the `folder-notes` plugin treats `Folder/Folder.md` as the folder's index page and hides it in the file tree. Every folder that represents a topic gets one. When you rename a folder, rename its folder note to match.
- **Links:** use `[[wikilinks]]` by note name with no path or `.md` (e.g. `[[Mobile Offline]]`, `[[Stress Testing]]`). Note names must stay unique across the vault for this to work. Obsidian's `alwaysUpdateLinks` fixes links when a rename happens *inside Obsidian*. A rename done from the shell does not, so grep for and update the `[[Old Name]]` references yourself.
- Notes are plain Markdown with heading-structured sections and terse bullets. Frontmatter/properties aren't used yet, so don't add them unasked.
- A stub note (a heading only, or empty) is a deliberate placeholder for an area that will be filled in later. Don't delete it.

## Workflows seen in the history

- **Journal processing:** read a day's journal and move the durable facts (decisions, dates, concerns, action items) into the relevant project or architecture notes. Leave the journal entry as the dated record.
- **Daily planning:** create or fill today's journal note from the template, and update project notes (e.g. key dates, areas of concern).

## Git

- `.gitignore` excludes per-device Obsidian UI state (`workspace*.json`), `.trash/`, `.DS_Store`, and iCloud conflict and placeholder files. Never commit those.
- Commit messages are short and lowercase and name the activity ("journal processing", "daily planning", "updating wiki"). Commit only when asked.
- iCloud can produce `* (conflicted copy)*` files. If you see one, point it out instead of silently merging or deleting it.
