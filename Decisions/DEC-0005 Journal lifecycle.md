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

**Decision (proposed):** Journal notes are immutable capture. Triage is a per-day act: read the day once, route every durable item to a front, a decision, a reference note, or `Fronts/Unrouted.md`, and leave ephemeral content alone. Triage never deletes or rewrites content; it adds marks. `processed: YYYY-MM-DD` means the day has been triaged and is the only test of "done," independent of which or how many fronts it touched. An open task that moved becomes a forwarded line marked with a plain glyph (`- ↪ text → [[Front]]`), not a checkbox state. After triage, the journal holds no open tasks; fronts are the only place open threads live.

## Context
Tasks written in daily notes went unseen after the day passed, and knowledge did not compound ([[Claude Working System]]). The first front ([[Mobile Application]]) was built by extracting journal content from 2026-08-31 to 2026-09-22, which raised the question of what happens to the journal afterwards.

## Options considered
1. **Leave the journal untouched; use the front's `as_of` as a watermark.** Simplest, but Royce edits older days, and nothing in the journal shows a task has moved, so it still reads as open.
2. **Delete or trim processed content.** Loses the record of what was known on a given day, which fronts cannot hold and which git history alone does not surface in Obsidian.
3. **Mark, do not modify** (chosen). A per-day `processed` date, and glyph forwarding for tasks that moved. The journal stays a faithful daily record; task views stop counting forwarded items as open; triage skips already-processed days.
4. **A per-front `processed` list as the test of done** (tried first, rejected 2026-09-23). A day with nothing relevant to a front would never be marked for it, so "fully processed" could not be read from the note.
5. **A separate `routed` list recording which fronts a day fed** (rejected 2026-09-23). Redundant: forwarded lines already link to their front, and the front's Log headings link back to the journal day, so backlinks carry provenance in both directions without a property.

## Consequences
- Journal notes gain exactly one frontmatter field, `processed`, the first time they are triaged. Every triaged day gets it, even one with no durable content, because "nothing to route" is still a completed triage.
- Log headings in fronts are links to journal days (`## [[2026-09-08]]`), which is how a front records what fed it.
- A custom checkbox state such as `[>]` was tried and rejected: default Obsidian renders any non-space character as a checked box, which reads as "done." The `↪` glyph is deliberately not a checkbox.
- A task forwarded to a front is closed in the front, not in the journal. If Royce ticks the journal line anyway, the reconcile treats that as a signal to close the item in the front; the line is otherwise never edited.
- Journal notes are archived monthly into `Journal/YYYY/MM-Month/`. Links resolve by filename, so archiving does not affect fronts or marks; the reconcile searches `Journal/` recursively. The archive move is a candidate for a month-end skill, or can be eliminated by setting the Daily Notes date format to `YYYY/MM-MMMM/YYYY-MM-DD`.
- Triage is the only process that reads the journal. Per-front reconcile reads what triage routed to it, plus Slack and Jira; it does not rescan journal days.
- Items whose front does not yet exist go to `Fronts/Unrouted.md`. A cluster forming there is the signal to create a front.
- The ten days extracted for the Mobile front on 2026-09-22 carry forwarded lines only; they are not `processed` until their non-mobile items are triaged.
- Status moves to `decided` after a week of use if the convention holds; otherwise revisit.
