---
id: DEC-0002
title: Identity documents live outside the vault
status: decided
date: 2026-09-22
fronts: [working-system, security]
owner: Royce Nobles
communicated: []
---
# DEC-0002 Identity documents live outside the vault

**Decision:** Government ID, employment verification and agreement, and the articles of incorporation are stored as encrypted attachments in Bitwarden, not in the vault or the repository. The vault holds only a pointer: "verification documents live in Bitwarden."

## Context
These files were placed in the vault for the Apple Business and Google Play registration workflow, which is complete. They are the documents used to prove identity and authority to vendors, which makes them the highest-value target in the vault. No agent will ever need them.

## Options considered
1. **Keep in the vault, gitignore a private folder.** Protects the repo but not the connected-folder path, and relies on convention.
2. **Move to a sibling folder outside the vault.** Hard boundary, but Obsidian links break and the files remain on a general-purpose filesystem.
3. **Move to the password manager** (chosen). Encrypted at rest, already the place credentials live, nothing left in the vault to protect.

## Consequences
- No private-folder architecture in the vault. Write rules for agents are relaxed to one line: do not write anything an agent found that Royce would not post in the leadership channel himself.
- Company-owned documents (articles of incorporation) should move to a shared Bitwarden organization collection when one exists, so they outlive any individual account.
- Any future identity or contract document follows the same rule: password manager, never the vault.
