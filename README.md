# claude-work-setup

Playbook that makes Claude Code set up an engineering work organization on macOS:
work dossiers per incident/investigation/migration/architecture doc, a growing knowledge base,
skills (triage, investigate-system, migration-plan, adr, close-out), agents (log-analyst,
code-explorer, reviewer), a production write guard hook and permissions.

It integrates with an existing `~/.claude` setup: backs it up first, inventories it, and reuses
or extends existing skills/agents instead of duplicating them. It stops for approval before
changing anything.

## Usage

```bash
git clone https://github.com/machado-vitor/claude-work-setup.git ~/claude-work-setup
cd ~ && claude
```

Then paste:

```
Read ~/claude-work-setup/PLAYBOOK.md completely and execute it step by step. Start with Phase 0.
```

Requirements: `jq` (bundled with macOS 15+, otherwise `brew install jq`), `git`.

## Rollback

Phase 0 creates `~/.claude-backup-<timestamp>`. The final report prints the exact restore command.
