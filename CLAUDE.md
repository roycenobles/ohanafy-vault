# Ohanafy vault — map for Claude

This vault is Royce Nobles's working memory as Chief Architect at Ohanafy (with CTO and CISO duties forming). It is the **canonical state** of what he knows, has decided, and has open. Claude sessions of every kind (Claude Code here, Cowork with this folder connected, scheduled cloud runs on the `roycenobles/ohanafy-vault` repo) read this file first and follow it.

North star for every note and every job: Ohanafy goes from one customer go-live per year to twelve or more. If work does not serve that, ask why it is being done.

The full plan is `System/Claude Working System.md`. Read it once per session when doing anything structural. Decisions that govern this vault are in `Decisions/` (DEC-0001 onward); read them before proposing anything that would contradict one.

## Layout

| Folder | Holds | Rule |
| --- | --- | --- |
| `Fronts/<Name>/<Name>.md` | One note per area of responsibility (Mobile Application, and later Salesforce platform, AWS, Security & compliance, Integrations, Data & cost, AI strategy). Attachments live beside the note in the same folder. | Read the front before touching anything about that area. |
| `Decisions/DEC-nnnn <Title>.md` | One note per decision, ADR shape. | Append-only in spirit: change `status`, add consequences; never rewrite the decision itself. |
| `Journal/YYYY-MM-DD.md`, archived monthly into `Journal/YYYY/MM-Month/` | Daily capture: meeting notes, tasks, observations. Current month at the root; past months in their folder (e.g. `2026/08-August/`). Search recursively. | **Immutable.** Never delete or rewrite content. Processing only adds marks (see Information flow). |
| `System/` | The plan and other notes about the working system itself. | Vault copy of the plan is re-exported from the Claude doc it points to; edit there, not here. |
| `Projects/` | Legacy. Being dissolved into `Fronts/` and `Reference/`. | Do not add to it. |
| `Reference/` | (future) People, systems, vendors: who owns what, what talks to what. | |
| `Templates/` | (future) Obsidian templates; the same files the skills use. | |
| `.obsidian/` | Obsidian and plugin configuration. Committed so it follows the repo. | Do not edit unless asked. |
| `.claude/` | Skills, agent definitions, settings for this vault. | |

Filenames are stable identifiers: Obsidian resolves `[[links]]` by filename and Claude finds backlinks by grepping for `[[Name`. Move folders freely; never rename a note without updating every link to it.

## Note shapes

### Front (`type: front`)
Frontmatter: `type, title, status, as_of, owner, tracks, milestones, related, sources`.
Sections in this order:
1. Callout explaining which sections are rewritten vs appended.
2. Quoted requirement or goal, if one exists.
3. `# Status` — why this exists, where it stands (dated), risks being watched. **Rewritten** at each reconcile; bump `as_of`.
4. `# Open threads` — `## Decisions needed`, `## Tasks`, `## Questions`. Checkboxes with owner and capture date. **Rewritten**: resolved items are removed here and recorded in the Log.
5. `# Decisions` — links to DEC notes; candidates not yet written up.
6. `# People` — who is involved and in what role.
7. `# Log` — dated, **append-only**. Date headings link to the journal day (`## [[2026-09-08]]`); cite Slack permalinks and Jira keys inline.
8. `# Reference` — links only. Nothing authoritative is copied into a front.

The Status section should answer a hallway question in under a minute: dates, owners, what is blocked, what changed.

### Decision (`type` implied by folder)
Frontmatter: `id, title, status (proposed | decided | pending | superseded), date, fronts, owner, communicated`.
Sections: bold one-line **Decision**, `## Context`, `## Options considered` (numbered, chosen one marked), `## Consequences`. Next id = highest existing + 1.

### Journal
No required shape. Headings vary by day and that is fine.

## Information flow

Capture → triage → front/decision → status. One direction. Nothing is authoritative in two places.

- **Capture** happens wherever Royce is: journal, a chat, a Claude Code session, Slack.
- **Triage** is a per-day act, done once: read the day, route every durable item to a front, a decision, a reference note, or `Fronts/Unrouted.md` (for items whose front does not exist yet), and deliberately leave ephemeral content where it is. Then **mark the day**:
  - `processed: YYYY-MM-DD` in frontmatter means the day has been triaged. This is the only meaning of "done" for a journal day, and it holds even if the day touched no front at all.
  - An open task that moved becomes forwarded: `- [ ] text` → `- ↪ text → [[Name]]`. A plain glyph, not a checkbox state, so it reads as "moved" rather than "done." After triage, the journal holds no open tasks; **fronts are the only place open threads live**.
  - If Royce later ticks a forwarded line (`- [x] … → [[Name]]`), treat it as a signal to close the item in the front. The journal line itself is not edited.
  - Notes and observations stay verbatim. Provenance runs through links, not properties: the front's Log headings link to the journal day (`## [[2026-09-08]]`), and forwarded lines link to the front, so backlinks answer "what fed this front" and "where did this go" in both directions.
- **Reconcile** (per front, on a rhythm) does not read the journal. It reads what triage routed to the front, the front's Slack channel and Jira epic, and the front itself; rewrites Status and Open threads; appends to Log with sources; bumps `as_of`. Stage for review.
- When `Fronts/Unrouted.md` accumulates a cluster of related items, that is the signal to create a front.
- **Status** is read by Royce in Obsidian and by any session asked "where does X stand."

## Working rules

1. **Read before write.** The front before touching its area; `Decisions/` before proposing something that could contradict one; this file at session start.
2. **Stage, never commit** (DEC-0004). End a change with `git add` and one line on what to look for in the diff. Commit only when Royce asks. Unattended runs commit to their own branch and open a PR.
3. **Write rule** (DEC-0002): do not write anything you found that Royce would not post in the leadership channel himself. Identity documents, contracts, and anything used to prove authority to a vendor never enter the vault; they live in Bitwarden.
4. **Provenance.** Every Log entry and every Status claim traces to a journal date, a Slack permalink, a Jira key, or a conversation with Royce. Say when something is from the vault alone and not yet reconciled.
5. **Dates** are ISO (`2026-09-22`). Update `as_of` whenever Status is rewritten.
6. **Ask, don't guess** for facts only Royce knows (owners, dates, what a name refers to). Leave an explicit "not yet identified" rather than inventing.
7. **Do not touch `.obsidian/`** or Obsidian's `.trash/` unless asked.

## Current state of the system (2026-09-22)

- Fronts: Mobile Application (first, built from the journal; not yet reconciled against Slack/Jira).
- Decisions: DEC-0001 sync model, DEC-0002 identity documents, DEC-0003 GitHub credential (pending), DEC-0004 review before history, DEC-0005 journal lifecycle (proposed).
- Skills, agents, templates: none yet. The reconcile procedure above is the first skill to write.
- Sensors and roles: none running. Sequencing is memory first, one loop second, roles third.
