---
id: DEC-0004
title: Review before history
status: decided
date: 2026-09-22
fronts: [working-system]
owner: Royce Nobles
communicated: []
---
# DEC-0004 Review before history

**Decision:** No change enters the vault's git history without Royce reading it first. In interactive sessions Claude stages changes and Royce commits. Unattended roles commit to their own branch and open a pull request; `main` holds only what Royce has approved.

## Context
Royce reviews diffs in Warp before committing, a habit from software development that fits this system well. Claude sessions and, later, standing roles will write to the vault. The vault is canonical memory ([[Claude Working System]]), so an unreviewed write is an unreviewed change to what every future session believes.

## Options considered
1. **Claude commits directly.** Fastest; history fills with changes Royce has not read. Acceptable only for plumbing.
2. **Claude stages, Royce commits** (chosen for interactive work). Every diff is read; Royce's commit message records his judgment; `git restore --staged` is a clean veto.
3. **Roles commit to branches and open PRs** (chosen for unattended work). A scheduled run has no reviewer present, so it cannot wait; a branch plus a PR defers the review instead of skipping it, and reuses the discipline already used for human contributors.

## Consequences
- Claude, when working with Royce, ends a change with `git add` and a one-line note on what to look for in the diff. It does not commit unless asked.
- Each standing role has a branch naming convention and a commit identity, so the PR list reads as a log of what the team of roles did.
- A role's PR is merged by Royce, or by a rule he sets later (for example, auto-merge for open-thread updates only). No rule exists yet.
- Depends on [[DEC-0003 GitHub credential for cloud roles]] for roles to push at all.
