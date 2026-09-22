---
id: DEC-0001
title: Vault sync model
status: decided
date: 2026-09-22
fronts: [working-system]
owner: Royce Nobles
communicated: []
---
# DEC-0001 Vault sync model

**Decision:** The Ohanafy vault uses two sync channels for two audiences. iCloud syncs the vault between Royce's devices. A private GitHub repository (`roycenobles/ohanafy-vault`) syncs the vault between the vault and Claude. GitHub is the source of truth when the two disagree.

## Context
The vault is the canonical memory for the working system described in [[Claude Working System]]. Claude reaches it two ways: as a connected folder when a session is linked to the MacBook, and by cloning the repository when running unattended in the cloud. Only the second path works while the Mac is asleep, so standing roles depend on the repository being current.

## Options considered
1. **iCloud for devices, git for Claude** (chosen). Keeps existing device sync; adds a repo Claude can always reach. Risk: iCloud and git in one folder can produce conflicted copies or mangle `.git` during sync.
2. **Git only; retire iCloud for this vault.** Cleanest single source. Cost: mobile sync depends on Obsidian Git, which is slow on mobile and needs credentials on every device.
3. **Obsidian Sync plus git.** Replaces iCloud with Obsidian's own sync. Cost: a paid service for a problem iCloud already solves.

## Consequences
- `.gitignore` excludes `.DS_Store`, Obsidian's per-device `workspace*.json`, `.trash/`, iCloud placeholders and conflicted copies.
- Obsidian Git (when installed) commits on a slow interval (10–15 min) and is disabled on mobile, so iCloud has time to settle before a commit.
- Royce's Mac is the only machine holding a GitHub credential until a cloud role needs one; that is a separate, pending decision ([[DEC-0003 GitHub credential for cloud roles]]).
- Revisit if conflicted copies appear in history more than occasionally.
