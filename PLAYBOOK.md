# Work setup playbook for Claude Code

You are setting up how I organize my engineering work on this Mac. Read this ENTIRE file before doing anything.
Talk to me in Portuguese. Write every file, script and comment in English.

## What I do
Software engineer: troubleshooting prod and nonprod, architecture documentation, investigating unfamiliar systems for integrations and migrations.

## Target organization
- SRC_DIR (default ~/src): code clones only. No notes inside.
- WORK_DIR (default ~/work): one folder ("dossier") per piece of work, named <yyyy-mm>-<type>-<slug>. I start `claude` from inside it. Types: incident, investigation, migration, arch.
- KB_DIR (default ~/kb): durable knowledge that grows over time: systems/, runbooks/, adr/, glossary.md. A git repo.
- Global Claude config in ~/.claude: a short rules block, skills, agents, a production guard hook, permissions.

IMPORTANT: I ALREADY have my own setup in ~/.claude (an assistant I built with CLAUDE.md, skills, agents, hooks, maybe memory/notes folders). It must be preserved and integrated. Nothing of mine gets lost or replaced.

## Hard rules (follow all of them, always)
1. Phase 0 backup happens before any change. Never delete a file. Never rewrite text I wrote; you may only append, or replace text between your own `work-setup` markers.
2. Never move my work files or notes. I move them myself. You only create empty structure.
3. Single source of truth: if something I already have does the job of a component below, REUSE or EXTEND it. Never create a parallel duplicate.
4. Stop at every line marked STOP and wait for my answer.
5. Keep ~/.claude/work-setup/PROGRESS.md as a checklist of every step below. Mark a step done right after finishing it. If the session is compacted or restarted: re-read this file and PROGRESS.md, then continue from the first unchecked step.
6. Local files only. Do not run anything against any cloud account, cluster or database.
7. Use your todo list for the phases. Do one step at a time and check the result before the next.
8. When this file gives file contents, write them EXACTLY as given. Change only what a step explicitly tells you to change.
9. Shell variables do not survive between your Bash calls. Whenever a command uses `$BK`, start it with `BK=<backup path from PROGRESS.md>`.

---

## Phase 0 — Backup

```bash
mkdir -p ~/.claude/work-setup
TS=$(date +%Y%m%d-%H%M%S)
BK=~/.claude-backup-$TS
rsync -a --exclude projects --exclude shell-snapshots --exclude todos --exclude statsig --exclude ide --exclude work-setup ~/.claude/ "$BK/"
cp ~/.claude.json "$BK/claude.json" 2>/dev/null || true
echo "$BK"
```

Verify: number of files in $BK is > 0 and matches `find ~/.claude` minus the excluded folders. Write the backup path at the top of PROGRESS.md.

## Phase 1 — Inventory (read-only)

Write ~/.claude/work-setup/INVENTORY.md. Summaries, not full file dumps:
1. ~/.claude/CLAUDE.md: line count, list of headings with a one-line summary each, any @imports.
2. ~/.claude/rules/*.md: name + one-line summary.
3. Skills: every ~/.claude/skills/*/SKILL.md: name, description, one line on what it actually does.
4. Commands: every ~/.claude/commands/*.md: name + one line.
5. Agents: every ~/.claude/agents/*.md: name, description, tools, model.
6. ~/.claude/settings.json: permissions (allow/ask/deny), every hook (event, matcher, command), env, other top-level keys.
7. Hook scripts referenced and their paths.
8. MCP servers: output of `claude mcp list`.
9. Plugins: enabledPlugins.
10. Every folder my setup uses as memory, notes, inbox, knowledge or logs. Find them by reading paths referenced inside CLAUDE.md, rules, skills, agents and hooks. For each: path, what it holds, approx file count.
11. Do ~/src, ~/work, ~/kb already exist? What is in them (top level only)?
12. Which of these are on PATH: git gh jq python3 kubectl aws gcloud az terraform helm direnv ghq fzf. Is ~/bin or ~/.local/bin on PATH? Which shell rc file is used (~/.zshrc?)?

## Phase 2 — Mapping (decide only, change nothing)

Write ~/.claude/work-setup/MAPPING.md as a table: Component | Decision | Existing item involved | Why.

Components (specs in the Appendices):
- C1 paths: SRC_DIR, WORK_DIR, KB_DIR. If my setup already has a knowledge/notes folder that fits the KB role, propose using it (or a subfolder of it) as KB_DIR instead of ~/kb. Same reasoning for WORK_DIR.
- C2 global rules block in ~/.claude/CLAUDE.md (Appendix F)
- C3 skill `triage` (D1)
- C4 skill `investigate-system` (D2)
- C5 skill `migration-plan` (D3)
- C6 skill `adr` (D4)
- C7 skill `close-out` (D5)
- C8 agent `log-analyst` (E1)
- C9 agent `code-explorer` (E2)
- C10 agent `reviewer` (E3)
- C11 hook prod-guard (Appendix C)
- C12 permissions ask/deny (Appendix G)
- C13 dossier templates + `newwork` script (Appendices A, B)
- C14 bin folder on PATH

Decision values:
- REUSE: an existing item already covers about 80% or more of the spec. Leave it untouched; other components refer to it by its name.
- EXTEND: an existing item covers part of it. Append a section `## Work-setup additions` with only the missing steps. Do not edit its existing text.
- CREATE: nothing similar exists. Create it from the Appendix.
- RENAME-CREATE: an item with the same name exists but does something different. Create the new one with the prefix `work-` (example: `work-triage`) and use that name everywhere.

Also write in MAPPING.md:
- Section "Existing items to wire in": my existing skills/agents/MCP servers that are useful for incidents, investigations, migrations or docs (example: a skill that queries logs). The new skills must mention them on their "Related existing items" line.
- Section "Conflicts": any rule of mine that contradicts Appendix F or G, with your recommendation. My existing rules win unless I say otherwise.
- Section "CLAUDE.md size": current line count + lines added by C2. If the total is over ~200 lines, propose (only propose) which of my sections could become a skill.

STOP 1: Give me a short summary of MAPPING.md in Portuguese (one line per component + conflicts) and ask for approval. Do not continue until I approve. Apply any change I ask to MAPPING.md first.

## Phase 3 — Apply (in this order; update PROGRESS.md after each)

3.1 Folders. Create: SRC_DIR; WORK_DIR/_templates/common/.claude; WORK_DIR/_archive; KB_DIR/systems, KB_DIR/runbooks, KB_DIR/adr. Create only if missing: KB_DIR/glossary.md (content: `# Glossary`), WORK_DIR/inbox.md (content: `# Inbox`). Put a `.gitkeep` in empty KB folders. If KB_DIR is not inside a git repo: `git init` there and commit "Initial KB structure".

3.2 Templates: write the files of Appendix A into WORK_DIR/_templates/.

3.3 Script: write Appendix B to <bin>/newwork (bin = ~/bin, or ~/.local/bin if that is already on PATH). If the approved WORK_DIR is not ~/work, change only the default in the line `WORK_DIR="${WORK_DIR:-$HOME/work}"`. `chmod +x`.

3.4 Hook: write Appendix C files to ~/.claude/hooks/prod-guard.sh and ~/.claude/hooks/test-prod-guard.sh. `chmod +x` both. If C11 is RENAME-CREATE, keep the filenames anyway (they are files, not skills).

3.5 settings.json merge with jq. Never replace the file wholesale:
- Add to `.hooks.PreToolUse` the entry `{"matcher":"Bash","hooks":[{"type":"command","command":"$HOME/.claude/hooks/prod-guard.sh"}]}` unless an entry with that command already exists. Keep every existing hook.
- Union `.permissions.ask` and `.permissions.deny` with Appendix G arrays, deduplicated. Skip any rule that is listed as a conflict I did not approve.
- Write to a temp file, run `jq empty` on it, then move it into place. Show me `diff` between the backup copy and the new file.

Use exactly this (first save Appendix G to ~/.claude/work-setup/perm.json, removing any rule I rejected; create `{}` as settings.json if it does not exist):
```bash
S=~/.claude/settings.json; P=~/.claude/work-setup/perm.json
[ -f "$S" ] || echo '{}' > "$S"
jq --slurpfile p "$P" '
  .permissions.ask  = ((.permissions.ask  // []) + $p[0].ask  | unique)
| .permissions.deny = ((.permissions.deny // []) + $p[0].deny | unique)
| .hooks.PreToolUse = ((.hooks.PreToolUse // []) +
    (if ([.hooks.PreToolUse[]?.hooks[]?.command] | index("$HOME/.claude/hooks/prod-guard.sh"))
     then [] else [{"matcher":"Bash","hooks":[{"type":"command","command":"$HOME/.claude/hooks/prod-guard.sh"}]}] end))
' "$S" > "$S.tmp" && jq empty "$S.tmp" && mv "$S.tmp" "$S"
diff <(jq -S . "$BK/settings.json" 2>/dev/null) <(jq -S . "$S")
```
(`$HOME` stays literal inside settings.json on purpose; the hook runner expands it.)

3.6 Skills and agents: follow the MAPPING decisions. Skills go to ~/.claude/skills/<name>/SKILL.md, agents to ~/.claude/agents/<name>.md. Replace the `Related existing items:` line in each new skill with the items from "Existing items to wire in" (or "none"). For EXTEND, append only the missing steps under `## Work-setup additions`.

3.7 Global CLAUDE.md: append Appendix F between the markers `<!-- work-setup:start -->` and `<!-- work-setup:end -->`, replacing SRC_DIR / WORK_DIR / KB_DIR with the real approved paths and skill/agent names with the final names (RENAME-CREATE prefixes). If the markers already exist, replace only what is between them. Omit any line whose component was REUSE because my own rules already say it.

3.8 PATH: if the bin folder is not on PATH, append to the shell rc file:
```bash
# work-setup
export PATH="$HOME/bin:$PATH"
```
(use $HOME/.local/bin if that was the chosen folder). Skip if already present.

## Phase 4 — Verify (show me the real output of each)

1. `bash ~/.claude/hooks/test-prod-guard.sh` → every line PASS, exit 0.
2. newwork in a sandbox:
```bash
T=$(mktemp -d); cp -R "<WORK_DIR>/_templates" "$T/"
WORK_DIR="$T" <bin>/newwork incident smoke-test
find "$T" -path "$T/_templates" -prune -o -print | sort
grep -rn '{{' "$T"/*-incident-smoke-test || echo "no placeholders left"
WORK_DIR="$T" <bin>/newwork bogus x; echo "rc=$? (expected 1)"
rm -rf "$T"
```
3. `jq empty ~/.claude/settings.json && echo valid`
4. For each SKILL.md and agent file you created or edited: first line is `---` and the frontmatter has `name:` and `description:`.
5. `grep -c 'work-setup:start' ~/.claude/CLAUDE.md` is 1, and `wc -l ~/.claude/CLAUDE.md`.
6. Nothing of mine lost: every file in the backup still exists in ~/.claude:
```bash
(cd "$BK" && find . -type f ! -name claude.json | sort) > /tmp/ws-before.txt
(cd ~/.claude && find . -type f | sort) > /tmp/ws-after.txt
comm -23 /tmp/ws-before.txt /tmp/ws-after.txt   # must print nothing
```

STOP 2 (final): write ~/.claude/work-setup/REPORT.md and tell me in Portuguese, briefly:
- what was CREATED, EXTENDED, REUSED (names and paths)
- pending conflicts
- rollback, with the real backup path filled in: `rsync -a --delete --exclude projects --exclude shell-snapshots --exclude todos --exclude statsig --exclude ide --exclude work-setup <backup>/ ~/.claude/` (this restores ~/.claude only; the new folders SRC_DIR/WORK_DIR/KB_DIR and the PATH line stay and can be removed by hand)
- first test: open a NEW terminal, `newwork incident teste`, cd into it, run `claude`, ask "use triage on: <fake alert>".

---

# Appendices

## Appendix A — Dossier templates (WORK_DIR/_templates/)

### A1 `common/NOTES.md`
```markdown
# Notes: {{SLUG}}

Started {{DATE}}. Chronological log. One entry per check:
`HH:MM | what was checked | result | source (command, query, link) | FACT or HYPOTHESIS`

## Log

## Ruled out

## Open questions
```

### A2 `common/.gitignore`
```
evidence/
.env*
*.har
```

### A3 `common/.claude/settings.local.json`
```json
{
  "permissions": {
    "additionalDirectories": []
  }
}
```

### A4 `incident.CLAUDE.md`
```markdown
# Incident: {{SLUG}} ({{DATE}})

## Context (fill in)
- Symptom / alert:
- Environment: prod | nonprod
- Services involved:
- Impact:
- Started at:

## Rules for this work
- Production is read-only for you. Propose any state-changing command with expected effect, how to verify and how to roll back. I run it.
- Log every finding in NOTES.md as it happens, marked FACT (with source) or HYPOTHESIS.
- Use the triage skill. Send large log or metric reads to the log-analyst agent.
- Repos for this work: add their paths to `.claude/settings.local.json` > permissions.additionalDirectories.
- When resolved, run the close-out skill.

## Status
- Current hypothesis:
- Next step:
```

### A5 `investigation.CLAUDE.md`
```markdown
# Investigation: {{SLUG}} ({{DATE}})

## Context (fill in)
- Goal (integration / migration / understanding):
- Systems to investigate:
- Questions to answer:
  1.

## Rules for this work
- Read-only. Ask narrow questions, one area at a time; send broad code searches to the code-explorer agent.
- Use the investigate-system skill. The output is the KB page of each system (KB systems folder from the global CLAUDE.md).
- Every claim needs a source (file:line, doc link, command output). Unknowns go to "Open questions".
- Repos for this work: add their paths to `.claude/settings.local.json` > permissions.additionalDirectories.
- When done, run the close-out skill.

## Status
- Answered:
- Next step:
```

### A6 `migration.CLAUDE.md`
```markdown
# Migration: {{SLUG}} ({{DATE}})

## Context (fill in)
- From -> To:
- Constraints (downtime, data volume, deadlines):
- Systems involved:

## Rules for this work
- PLAN.md is the source of truth. Update it before and after each phase.
- Use the migration-plan skill to build PLAN.md. Every phase needs verification criteria and a rollback.
- Code changes happen in a worktree (`claude -w <name>`), never in the main clone.
- After each phase, the reviewer agent checks the diff against PLAN.md.
- Production is read-only for you; I execute state-changing steps.
- Repos for this work: add their paths to `.claude/settings.local.json` > permissions.additionalDirectories.
- When done, run the close-out skill.

## Status
- Current phase:
- Next step:
```

### A7 `arch.CLAUDE.md`
```markdown
# Architecture doc: {{SLUG}} ({{DATE}})

## Context (fill in)
- What is being documented:
- Audience:
- Deliverable (doc, diagram, ADR, proposal):

## Rules for this work
- Diagrams as code (Mermaid by default; C4 levels: context, container, component).
- Base diagrams on what the code and config actually show; use the code-explorer agent to verify. Mark anything not verified.
- Decisions become ADRs via the adr skill.
- Deliverables go to out/. Have the reviewer agent review the final doc.
- Repos for this work: add their paths to `.claude/settings.local.json` > permissions.additionalDirectories.
- When done, run the close-out skill.

## Status
- Done:
- Next step:
```

### A8 `PLAN.md`
```markdown
# Plan: {{SLUG}}

## Goal and success criteria

## Inventory (what moves, dependencies, consumers)

## Strategy (chosen option and why; options rejected and why)

## Phases
### Phase 1: <name>
- Steps:
- Verification (counts, checksums, contract tests, metrics):
- Rollback:
- Go / no-go criteria:
- Status: pending

## Risks

## Decisions log
```

## Appendix B — `newwork` script
```bash
#!/usr/bin/env bash
# Create a work dossier from templates.
# Usage: newwork <incident|investigation|migration|arch> <slug>
set -euo pipefail
WORK_DIR="${WORK_DIR:-$HOME/work}"
types="incident investigation migration arch"
type="${1:-}"; slug="${2:-}"
if [ -z "$type" ] || [ -z "$slug" ]; then
  echo "usage: newwork <$(echo $types | tr ' ' '|')> <slug>" >&2; exit 1
fi
case " $types " in *" $type "*) ;; *) echo "unknown type: $type (use: $types)" >&2; exit 1;; esac
tpl="$WORK_DIR/_templates"
dir="$WORK_DIR/$(date +%Y-%m)-$type-$slug"
[ -e "$dir" ] && { echo "already exists: $dir" >&2; exit 1; }
mkdir -p "$dir/evidence" "$dir/out"
cp -R "$tpl/common/." "$dir/"
cp "$tpl/$type.CLAUDE.md" "$dir/CLAUDE.md"
[ "$type" = migration ] && cp "$tpl/PLAN.md" "$dir/PLAN.md"
for f in "$dir/CLAUDE.md" "$dir/NOTES.md" "$dir/PLAN.md"; do
  [ -f "$f" ] && sed -i '' -e "s/{{SLUG}}/$slug/g" -e "s/{{DATE}}/$(date +%F)/g" "$f"
done
echo "$dir"
```

## Appendix C — Production guard hook

### C1 `~/.claude/hooks/prod-guard.sh`
```bash
#!/usr/bin/env bash
# PreToolUse hook: blocks state-changing commands that target production.
# Exit 2 = block; stderr is shown to Claude.
input=$(cat)
if command -v jq >/dev/null 2>&1; then
  cmd=$(printf '%s' "$input" | jq -r '.tool_input.command // empty')
else
  cmd=$(printf '%s' "$input" | python3 -c 'import sys,json; print(json.load(sys.stdin).get("tool_input",{}).get("command",""))')
fi
[ -z "$cmd" ] && exit 0
lc=$(printf '%s' "$cmd" | tr '[:upper:]' '[:lower:]')

write_re='(kubectl[^|;&]*[[:space:]](apply|delete|edit|patch|replace|scale|drain|cordon|uncordon|taint|annotate|label|set|rollout[[:space:]]+(restart|undo))([[:space:]]|$)|helm[[:space:]]+(install|upgrade|uninstall|rollback|delete)([[:space:]]|$)|terraform[[:space:]]+(apply|destroy|import|state[[:space:]]+(rm|mv|push))([[:space:]]|$)|aws[[:space:]]+s3[[:space:]]+(rm|mv|sync)([[:space:]]|$)|aws[[:space:]].*[[:space:]](delete|terminate|put|update|create|modify|remove|stop|reboot)-|gcloud[[:space:]].*[[:space:]](delete|update|create|deploy|set)([[:space:]]|$)|(drop|truncate|alter)[[:space:]]+(table|database|schema)|delete[[:space:]]+from|update[[:space:]]+[a-z_.]+[[:space:]]+set|insert[[:space:]]+into)'
prod_re='(^|[^a-z])(prod|production|prd)([^a-z]|$)'

printf '%s' "$lc" | grep -Eq "$write_re" || exit 0

target_prod=0
printf '%s' "$lc" | grep -Eq "$prod_re" && target_prod=1
case "${AWS_PROFILE:-}${KUBECONFIG:-}" in *prod*|*prd*) target_prod=1;; esac
if printf '%s' "$lc" | grep -Eq '(kubectl|helm)'; then
  ctx=$(kubectl config current-context 2>/dev/null | tr '[:upper:]' '[:lower:]')
  printf '%s' "$ctx" | grep -Eq "$prod_re" && target_prod=1
fi

if [ "$target_prod" -eq 1 ]; then
  echo "BLOCKED by prod-guard: state-changing command against production. Show the user the exact command, expected effect, how to verify and how to roll back. The user runs it." >&2
  exit 2
fi
exit 0
```

### C2 `~/.claude/hooks/test-prod-guard.sh`
```bash
#!/usr/bin/env bash
# Regression tests for prod-guard.sh. Usage: test-prod-guard.sh [path-to-hook]
hook="${1:-$HOME/.claude/hooks/prod-guard.sh}"
fail=0
check() {
  expected=$1; c=$2
  printf '{"tool_input":{"command":"%s"}}' "$c" | AWS_PROFILE= KUBECONFIG=/dev/null "$hook" 2>/dev/null
  rc=$?
  if [ "$expected" = block ] && [ $rc -eq 2 ]; then echo "PASS block: $c"
  elif [ "$expected" = allow ] && [ $rc -eq 0 ]; then echo "PASS allow: $c"
  else echo "FAIL expected $expected (rc=$rc): $c"; fail=1; fi
}
check block "kubectl delete pod api-1 --context prod-eu"
check block "terraform apply -var-file=prod.tfvars"
check block "aws ec2 terminate-instances --instance-ids i-1 --profile production"
check block "psql -h prod-db.internal -c delete from users where id=1"
check block "helm upgrade api ./chart --kube-context prd-01"
check block "aws s3 rm s3://prod-bucket/x"
check allow "kubectl get pods --context prod-eu"
check allow "kubectl logs api-1 --context prod-eu"
check allow "terraform plan -var-file=prod.tfvars"
check allow "terraform apply -var-file=staging.tfvars"
check allow "aws s3 ls --profile production"
check allow "grep -r product src/"
exit $fail
```

## Appendix D — Skills (~/.claude/skills/<name>/SKILL.md)

### D1 `triage/SKILL.md`
```markdown
---
name: triage
description: Use when investigating an incident, alert, error spike, latency increase or outage in prod or nonprod. Hypothesis-driven triage with evidence logged in NOTES.md.
---
# Triage

Related existing items: (filled during setup)

1. Frame the problem: symptom, start time, scope (environment, share of traffic, which users), what changed recently. Ask me for missing facts in ONE batch of questions.
2. Read the KB pages (systems/ and runbooks/ in the KB folder from the global CLAUDE.md) of every service involved, if they exist.
3. List 3 to 6 hypotheses, ranked by likelihood and by how cheap they are to check. Usual suspects: recent deploy, config or feature-flag change, dependency down or slow, saturation (CPU, memory, connections, disk, quotas), traffic change, expired certificate or credential, bad data.
4. For each hypothesis, run the cheapest read-only check. Send large log or metric reads to the log-analyst agent.
5. After every check, append to NOTES.md: `HH:MM | check | result | source | FACT or HYPOTHESIS`. Move disproven hypotheses to "Ruled out".
6. Build a timeline: line up the symptom onset with deploys and config changes.
7. When a cause is confirmed with evidence, propose the mitigation: exact command, expected effect, how to verify, how to roll back. I run anything that changes state.
8. Every ~10 steps, and before any /compact, update the "Status" section of the dossier CLAUDE.md.

Never:
- Claim a root cause without evidence.
- Treat missing data as healthy: "no metric" is not "zero errors".
- Run state-changing commands in production.
```

### D2 `investigate-system/SKILL.md`
```markdown
---
name: investigate-system
description: Use when mapping an unfamiliar system, service or API for an integration, migration or troubleshooting. Produces or updates the system's KB page.
---
# Investigate a system

Related existing items: (filled during setup)

1. Check whether systems/<system>.md exists in the KB folder (see global CLAUDE.md). If yes, start from it and only fill gaps.
2. Sources, in this order: repo README and docs, API specs (OpenAPI, proto, AsyncAPI), config and IaC (terraform, helm, k8s manifests), code entry points, dashboards and alerts, then questions to me.
3. Work one section at a time. For broad code searches across repos, use the code-explorer agent.
4. Every statement needs a source (file:line, link or command). Anything not confirmed goes to "Open questions".
5. Show me the page, then write it to systems/<system>.md in the KB folder and commit it in the KB repo.

Page template:

# <System name>
- Owner / team:
- Repos:
- Environments and URLs:
- Purpose (2 lines):

## Interfaces
Inbound and outbound: protocol, endpoint or topic, auth, payload format, rate limits, timeouts.

## Data
Stores, key entities, volumes, retention, PII.

## Dependencies
Upstream and downstream systems.

## Observability
Where logs are, metrics and dashboards, alerts.

## Access
How to get read-only access and credentials.

## Gotchas

## Open questions

## Sources

Last verified: <date>
```

### D3 `migration-plan/SKILL.md`
```markdown
---
name: migration-plan
description: Use when planning a migration (database, platform, service, broker, cloud, API version). Builds PLAN.md with phases, verification and rollback.
---
# Migration plan

Related existing items: (filled during setup)

1. Inventory: what moves (data, services, configs, jobs), who consumes it, volumes, current SLAs. Use the investigate-system skill for any system without a KB page.
2. Dependencies and order: what must move first; what breaks if one side changes.
3. Strategy: compare at least two options (big-bang, phased, dual-write, strangler fig, CDC/replication, blue-green). For each: risk, downtime, effort, rollback difficulty. Pick one and justify it.
4. Phases. Each phase needs: steps, verification (row counts, checksums, contract tests, metric comparisons), rollback, go/no-go criteria.
5. Write everything to PLAN.md in the dossier (template already there).
6. Ask the reviewer agent to review PLAN.md for gaps in verification and rollback. Fix the gaps that affect correctness.
7. Show me the plan before any execution.
```

### D4 `adr/SKILL.md`
```markdown
---
name: adr
description: Use when recording an architecture or technical decision. Writes a numbered ADR in the KB adr folder.
---
# Architecture Decision Record

Related existing items: (filled during setup)

1. If the company or the repo has its own ADR template, use it instead of the one below. Ask me if unsure.
2. Next number: highest NNNN in the adr/ folder of the KB (see global CLAUDE.md) plus one, 4 digits.
3. File: adr/NNNN-short-title-in-kebab-case.md
4. Show me the draft, then write it and commit it in the KB repo.

Template:

# NNNN. <Title>
- Date:
- Status: proposed | accepted | superseded by NNNN

## Context
The forces at play, facts with sources.

## Decision
What we will do, in active voice.

## Alternatives considered
Each option with pros and cons.

## Consequences
What becomes easier, what becomes harder, follow-up work.
```

### D5 `close-out/SKILL.md`
```markdown
---
name: close-out
description: Use when a piece of work (incident, investigation, migration, architecture doc) is done. Turns the dossier into durable knowledge and deliverables.
---
# Close out a piece of work

Related existing items: (filled during setup)

1. Read the dossier CLAUDE.md, NOTES.md and PLAN.md (if present).
2. KB updates: for every system touched, propose additions to systems/<system>.md in the KB (new facts, gotchas, access info). Show me the diff, then write.
3. If a procedure worked and will be reused, write it as runbooks/<name>.md in the KB: when to use, steps with exact commands, how to verify, rollback.
4. Deliverable in out/:
   - incident: out/postmortem.md, blameless: summary, impact, timeline, root cause, contributing factors, what went well, what went badly, action items (owner, due date).
   - investigation, migration, arch: out/summary.md: goal, what was found or done, decisions (link ADRs), open items.
5. Commit the KB changes in the KB repo.
6. Ask me before moving the dossier to the _archive folder of WORK_DIR. Then `mv` it there.
```

## Appendix E — Agents (~/.claude/agents/<name>.md)
No `model:` field on purpose: agents inherit the session model.

### E1 `log-analyst.md`
```markdown
---
name: log-analyst
description: Use for reading large volumes of logs, metrics or command output during an investigation. Returns patterns and evidence, not raw dumps. Read-only.
tools: Read, Grep, Glob, Bash
---
You analyze large logs and metric outputs for an engineer during an investigation.

Rules:
- Read-only. Never run a command that changes state anywhere.
- If asked, save raw outputs to the given evidence/ path instead of returning them.

Return, in at most 40 lines:
1. Main patterns, with counts.
2. First and last occurrence of each relevant pattern (timestamps).
3. Up to 5 exact sample lines for the top errors.
4. Correlation with any timestamps or events you were given.
5. The exact commands or queries you used, so the engineer can rerun them.
6. What you could NOT check and why.
```

### E2 `code-explorer.md`
```markdown
---
name: code-explorer
description: Use for broad read-only code searches across one or more repos, e.g. "where is X handled", "who calls Y", "how does data flow from A to B".
tools: Read, Grep, Glob, Bash
---
You answer questions about code across one or more repositories.

Rules:
- Read-only. Bash only for read commands (git log, git blame, git grep, ls, cat).
- Every claim must carry a file:line reference.
- Say explicitly when something was not found, and where you looked.

Return: a direct answer first, then the evidence list (file:line + one line each), then open questions. Maximum 50 lines.
```

### E3 `reviewer.md`
```markdown
---
name: reviewer
description: Use to review a plan, document or diff in a fresh context against stated criteria before calling the work done.
tools: Read, Grep, Glob
---
You review a plan, document or diff against the criteria you are given. You did not write it.

Rules:
- Report only gaps that affect correctness, safety or the stated requirements. No style preferences.
- For plans: check every phase has verification and rollback.
- For docs and diagrams: check claims against the code or sources when paths are given.

Return a numbered list: severity (blocking / important / minor), the gap, where, suggested fix.
If there are no blocking gaps, say "No blocking gaps" on the first line.
```

## Appendix F — Global CLAUDE.md block
```markdown
<!-- work-setup:start -->
## Work organization
- Code clones: SRC_DIR. Work dossiers: WORK_DIR/<yyyy-mm>-<type>-<slug>, created with `newwork <incident|investigation|migration|arch> <slug>`. Knowledge base: KB_DIR (systems/, runbooks/, adr/, glossary.md).
- Before investigating a system, read KB_DIR/systems/<name>.md if it exists.
- Production is read-only for you. Propose state-changing commands with expected effect, verification and rollback; I run them.
- Inside a dossier, log findings in NOTES.md as they happen: FACT (with source) or HYPOTHESIS.
- Show evidence (command and relevant output), never just "verified, it's fine".
- Skills: triage, investigate-system, migration-plan, adr, close-out. Agents: log-analyst, code-explorer, reviewer.
- When a piece of work ends, run close-out so the knowledge base grows.
<!-- work-setup:end -->
```

## Appendix G — Permissions to merge
```json
{
  "ask": [
    "Bash(kubectl delete:*)",
    "Bash(kubectl apply:*)",
    "Bash(helm uninstall:*)",
    "Bash(terraform apply:*)",
    "Bash(terraform destroy:*)",
    "Bash(git push --force:*)",
    "Bash(git push -f:*)",
    "Read(**/.env)",
    "Read(**/.env.*)"
  ],
  "deny": [
    "Read(~/.aws/credentials)",
    "Read(~/.ssh/id_*)"
  ]
}
```
