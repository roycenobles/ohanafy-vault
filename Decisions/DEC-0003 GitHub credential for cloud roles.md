---
id: DEC-0003
title: GitHub credential for cloud roles
status: pending
date: 2026-09-22
fronts: [working-system, security]
owner: Royce Nobles
communicated: []
---
# DEC-0003 GitHub credential for cloud roles

**Decision:** Pending. Due when the first standing role is ready to run unattended.

## Context
A scheduled Claude task runs fresh in the cloud and must clone and push `ohanafy-vault` to read decisions and write findings. That requires a credential the cloud run can use. Until then, only Royce's Mac holds a GitHub credential ([[DEC-0001 Vault sync model]]).

## Options considered
1. **Proxy-injected credential**, the mechanism Ohanafy already uses for Salesforce, Jira and Gong in Claude sessions. The token never appears in a prompt or file. Requires an admin to add GitHub to the injected-credential list.
2. **Fine-grained personal access token in the task configuration.** Scoped to this one repository, contents read/write only, with an expiry. Works without admin action; weaker posture because the token travels with the task.

## Consequences (when decided)
- Whichever path is chosen, the token is scoped to `ohanafy-vault` alone, contents read/write only, with an expiry date.
- Each role commits under a recognizable author name so the repository history is an audit trail of what the roles did.
