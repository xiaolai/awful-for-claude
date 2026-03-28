---
description: "Design an agent workflow system — define agents, events, routing, data flow, and quality gates interactively"
argument-hint: "[--from-templates]"
allowed-tools: Read, Write, Glob, Task, AskUserQuestion
---

# /awful:design

Design an agent workflow system for the current repository. Produces a machine-readable blueprint at `workflows/BLUEPRINT.md`.

## Arguments

- `--from-templates` — skip interactive design, generate a blueprint automatically from the templates selected during `/awful:init`

## Steps

### Step 1: Check Prerequisites

Check if awful has been initialized:
- If `.claude/awful.local.md` doesn't exist, suggest running `/awful:init` first and offer to proceed anyway.
- If `workflows/BLUEPRINT.md` already exists, ask: "A blueprint already exists. Would you like to: (1) Start fresh, (2) Extend the existing blueprint, or (3) Cancel?"

### Step 2: Route Based on Mode

#### Mode A: `--from-templates`

1. Read `.claude/awful.local.md` to get the selected templates
2. For each template, read its YAML frontmatter from `templates/`
3. Build a blueprint automatically:
   - `agents:` — one entry per template, using template name as role
   - `routing:` — derived from template `triggers:` and `conditions:`
   - `labels:` — union of all `labels_required:` across templates, plus `awful:` state labels
   - `human_gates:` — for pr-review (merge), docs (PR merge), release (release approval)
   - `error_handling:` — from each template's `error_handling:` field
4. Write the blueprint
5. Show summary and offer to adjust

#### Mode B: Interactive Design (default)

Dispatch the **architect agent** (`agents/architect.md`) with:
- Detection results from `commands/shared/detect.md`
- Contents of `.claude/awful.local.md` if it exists (for context on selected templates)
- Instruction to run the interactive design interview

The architect will:
1. Interview the user about their needs
2. Propose an architecture
3. Iterate based on feedback
4. Write `workflows/BLUEPRINT.md`

### Step 3: Post-Design

After the blueprint is written (either mode):

1. Show a brief summary of what was designed:
   ```
   Blueprint written: workflows/BLUEPRINT.md

   Agents: {n} ({list of names})
   Events wired: {list}
   Labels: {n} ({list of routing labels})
   Human gates: {list or "none"}
   ```

2. Tell the user the next step:
   ```
   Next: /awful:gen — generate workflow files from this blueprint
   ```
