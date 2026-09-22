---
id: DEC-0005
title: Journal lifecycle
status: proposed
date: 2026-09-22
fronts: [working-system]
owner: Royce Nobles
communicated: []
---
# DEC-0005 Journal lifecycle

**Decision (proposed):** Journal notes are immutable capture. Processing them into fronts and decisions never deletes or rewrites their content; it adds two marks. The day's frontmatter records which fronts it was processed into (`processed: ["[[Front]]"]`). An open task that moved to a front becomes a forwarded line marked with a plain glyph (`- ↪ text → [[Front]]`), not a checkbox state. After processing, the journal holds no open tasks; fronts are the only place open threads live.

## Context
Tasks written in daily notes went unseen after the day passed, and knowledge did not compound ([[Claude Working System]]). The first front ([[Mobile Application]]) was built by extracting journal content from 2026-08-31 to 2026-09-22, which raised the question of what happens to the journal afterwards.

## Options considered
1. **Leave the journal untouched; use the front's `as_of` as a watermark.** Simplest, but Royce edits older days, and nothing in the journal shows a task has moved, so it still reads as open.
2. **Delete or trim processed content.** Loses the record of what was known on a given day, which fronts cannot hold and which git history alone does not surface in Obsidian.
3. **Mark, do not modify** (chosen). Frontmatter `processed` list per day, and `[>]` forwarding for tasks that moved. The journal stays a faithful daily record; task views stop counting forwarded items as open; reconcile skips already-marked days.

## Consequences
- Journal notes gain frontmatter the first time they are processed. Days with no durable content are never marked. The property is a flat list of links so Obsidian renders it as chips; the processing date lives in git history, not the property.
- A custom checkbox state such as `[>]` was tried and rejected: default Obsidian renders any non-space character as a checked box, which reads as "done." The `↪` glyph is deliberately not a checkbox.
- A task forwarded to a front is closed in the front, not in the journal. If Royce ticks the journal line anyway, the reconcile treats that as a signal to close the item in the front; the line is otherwise never edited.
- Journal notes are archived monthly into `Journal/YYYY/MM-Month/`. Links resolve by filename, so archiving does not affect fronts or marks; the reconcile searches `Journal/` recursively. The archive move is a candidate for a month-end skill, or can be eliminated by setting the Daily Notes date format to `YYYY/MM-MMMM/YYYY-MM-DD`.
- Reconcile reads only journal days whose `processed` list lacks the current front, so a day can be processed into several fronts over time.
- Status moves to `decided` after a week of use if the convention holds; otherwise revisit.
