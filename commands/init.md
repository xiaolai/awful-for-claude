---
description: "Initialize awful for this repo — detect repo type, check gh-aw, select templates, create workflow directory"
allowed-tools: Read, Write, Bash, Glob, AskUserQuestion
---

# /awful:init

Initialize awful for the current repository.

## Steps

### Step 1: Run Detection

Execute the checks from `commands/shared/detect.md`. Display the results:

```
Detecting environment...

- gh-aw: {available vX.Y.Z | not available — will generate GitHub Actions YAML}
- Repo type: {type}
- Existing workflows: {n files | none}
- Blueprint: {found | not found}
```

If a blueprint already exists, ask: "A blueprint already exists at `workflows/BLUEPRINT.md`. Re-run init? This will not overwrite the blueprint or workflow files — only the awful config. [y/N]"

### Step 2: Template Selection

Based on repo type, recommend templates:

| Repo Type | Recommended Templates |
|-----------|----------------------|
| OSS library | triage, stale, first-timer |
| OSS app | triage, pr-review, stale, first-timer |
| Internal service | triage, pr-review, incident |
| Production service | all 7 |
| Any (minimal) | triage |

Ask the user: "Which templates would you like to install? I recommend {list} for a {type} project."

Show all 7 options with one-line descriptions:
- **triage** — classify new issues automatically
- **pr-review** — automated first-pass code review
- **docs** — generate docs when issues are labeled needs-docs
- **release** — generate changelogs and create GitHub releases on tags
- **stale** — close inactive issues and PRs after 37 days
- **first-timer** — welcome first-time contributors
- **incident** — structure incident response when issues are labeled incident

Let the user confirm or modify the selection.

### Step 3: Create Workflow Directory

```bash
mkdir -p workflows
```

If `workflows/` already exists, skip silently.

### Step 4: Check Secrets

Check for `ANTHROPIC_API_KEY` repo secret using `gh secret list 2>/dev/null`:
- If present → confirm
- If not found (or command fails) → warn:

```
Warning: ANTHROPIC_API_KEY is not set as a repo secret.
Agents will fail at runtime without it.

To add it:
  gh secret set ANTHROPIC_API_KEY --body "sk-ant-..."

Or for gh-aw format, CLAUDE_CODE_OAUTH_TOKEN can be used instead.
```

### Step 5: Write Config

Create `.claude/awful.local.md`:

```markdown
---
output_format: auto
templates:
  - {selected template 1}
  - {selected template 2}
initialized_at: {ISO timestamp}
repo_type: {detected type}
---
```

If `.claude/` doesn't exist, create it.

### Step 6: Show Summary

```
awful initialized successfully.

Templates selected: {list}
Config written: .claude/awful.local.md
Output format: {gh-aw markdown | GitHub Actions YAML}
{If ANTHROPIC_API_KEY missing: Warning: Set ANTHROPIC_API_KEY repo secret before running workflows}

Next steps:
  /awful:design    — interactively design your agent system → blueprint
  /awful:gen       — generate workflow files from an existing blueprint
```
