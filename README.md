# awful

**Agent Workflow Framework Using Labels** — design, generate, and wire agent workflows driven by GitHub events.

awful turns your GitHub repo into an agent-powered automation system. GitHub is the infrastructure: events are the message bus, labels are routing keys, and issue/PR comments are agent I/O. No external orchestrator needed.

Available from the [xiaolai marketplace](https://github.com/xiaolai/claude-plugin-marketplace).

---

## What It Does

| Capability | Status | Description |
|-----------|--------|-------------|
| **Design** | v0.1 | Interactive agent system design — architect interviews you, proposes architecture, outputs a machine-readable blueprint |
| **Generate** | v0.1 | Generate gh-aw markdown or GitHub Actions YAML workflow files from blueprints |
| **Wire** | v0.1 | 7 production-ready built-in templates: triage, PR review, docs, release, stale, first-timer, incident |
| **Simulate** | v0.2 | Dry-run workflows locally before pushing to GitHub |
| **Audit** | v0.2 | Check existing workflows for loop risks, permission issues, and coverage gaps |

---

## Installation

```bash
# Install for this project
claude plugin install awful@xiaolai --scope project

# Install for all projects
claude plugin install awful@xiaolai --scope user
```

> **Install fails with "Plugin not found in marketplace 'xiaolai'"?** Your local marketplace clone is stale. Run `claude plugin marketplace update xiaolai` and retry — `plugin install` does not auto-refresh.

---

## Quick Start

Three commands to go from zero to running agents:

```bash
# 1. Initialize awful for this repo (detects environment, selects templates)
/awful:init

# 2. Design your agent system interactively (produces workflows/BLUEPRINT.md)
/awful:design

# 3. Generate workflow files from the blueprint
/awful:gen
```

Then commit the generated files and push. GitHub will run your agents on the next matching event.

---

## Commands

### v0.1 (available now)

| Command | Description |
|---------|-------------|
| `/awful:init` | Initialize awful — detect repo type, select templates, create config |
| `/awful:design` | Interactive agent system design → blueprint. Use `--from-templates` to skip interview |
| `/awful:gen` | Generate workflow files (gh-aw markdown or Actions YAML) from blueprint |
| `/awful:map` | Visualize event→workflow→agent routing as a mermaid diagram + routing table |

### Planned (v0.2+)

| Command | Description |
|---------|-------------|
| `/awful:sim` | Dry-run a workflow locally with a simulated event payload |
| `/awful:audit` | Check workflows for loop risks, permission issues, coverage gaps |
| `/awful:update` | Re-run design interview for a specific agent and regenerate its workflow |

---

## Built-In Templates

| Template | Trigger | Description |
|----------|---------|-------------|
| **triage** | `issues.opened` | Classify new issues (bug/feature/question), assess priority (high/medium/low), post acknowledgment |
| **pr-review** | `pull_request.opened/synchronize` | Automated first-pass code review — correctness, security, quality, consistency |
| **docs** | `issues.labeled:needs-docs` | Read related code, generate documentation, open a PR |
| **release** | `push:tags/v*` | Generate changelog from commits since last tag, create GitHub release |
| **stale** | `schedule:daily` | Warn issues/PRs inactive for 30 days inactive + 7 day grace period, then close |
| **first-timer** | `pull_request.opened` (FIRST_TIME) | Welcome first-time contributors with guidance and links |
| **incident** | `issues.labeled:incident` | Assess severity (SEV1-3), post structured tracking comment, notify team |

---

## How It Works

### GitHub as Infrastructure

awful doesn't introduce an external orchestrator. GitHub's built-in primitives are the entire infrastructure:

- **Events** (`issues.opened`, `pull_request.labeled`, `push:tags/*`) are the message bus
- **Labels** are routing keys — which label is applied determines which agent runs
- **Comments** are agent I/O — agents read context from comments, write results as comments
- **Workflows** are the agent runtime — each agent lives in a `.github/workflows/` file

### Blueprints as Design Docs

`workflows/BLUEPRINT.md` is the source of truth for your agent system. It has:
- YAML frontmatter (machine-readable): agents, routing rules, labels, human gates
- Markdown body (human-readable): system overview, routing rationale, known limitations

The `/awful:gen` command reads the blueprint and generates workflow files. The blueprint is the design doc; the workflow files are the artifact.

### Two Output Formats

| Format | How | When to Use |
|--------|-----|-------------|
| **gh-aw markdown** | Requires `gh aw` CLI extension | Safe-outputs buffer, cleaner agent prompts |
| **GitHub Actions YAML** | Standard Actions YAML | No extra tools required, maximum compatibility |

awful auto-detects which format to use based on your environment. Override with `output_format` in `.claude/awful.local.md`.

---

## Loop Prevention

Three mechanisms working together:

**1. Bot actor guard** — every job checks `if: github.actor != 'github-actions[bot]'` so agent actions don't re-trigger the same workflow.

**2. Label prefix convention** — agents apply labels with the `awful:` prefix (`awful:triaged`, `awful:needs-attention`). Routing conditions match only on unprefixed labels. Human-applied labels trigger agents; agent-applied labels don't.

**3. Concurrency groups** — `group: awful-{agent-name}-${{ github.event.issue.number }}` with `cancel-in-progress: false` ensures only one run per issue/PR at a time, preventing event storms.

---

## Security

| Feature | gh-aw Markdown | GitHub Actions YAML |
|---------|----------------|---------------------|
| Safe-outputs buffer (allowlist) | Yes (`safe-outputs: true`) | No |
| Network isolation | Partial | No |
| Fork PR guard | Applied automatically | Applied automatically |
| Permission scoping | Per-workflow | Per-workflow |
| Input sanitization framing | In every prompt | In every prompt |

Every generated workflow includes:
- Minimal permissions (only what the agent actually needs)
- Fork PR guard on all PR-triggered jobs with write permissions
- Data-not-instructions framing in every agent prompt
- Event context passed as environment variables, not shell-interpolated

---

## Agents

| Agent | Model | Role |
|-------|-------|------|
| **architect** | opus | Interactive system design — interviews user, proposes architecture, writes blueprint |
| **generator** | sonnet | Reads blueprint, generates workflow files in selected format |

---

## Skills

| Skill | Description |
|-------|-------------|
| `awful:patterns` | Event bus patterns, label routing, orchestration patterns, anti-patterns |
| `awful:conventions` | Blueprint format, gh-aw markdown format, Actions YAML format specs |
| `awful:security` | Permission scoping, fork PR guard, input sanitization, safe-outputs |
| `awful:templates` | Reference for all 7 built-in templates with customization points |

---

## Configuration

`.claude/awful.local.md` (created by `/awful:init`):

```yaml
---
output_format: auto       # auto | ghaw | actions
templates:                # templates selected during init
  - triage
  - stale
  - first-timer
initialized_at: 2026-03-28T00:00:00Z
repo_type: nodejs
---
```

`.github/awful-config.yml` (optional, for per-team settings):

```yaml
incident_team: "@your-org/on-call"
mentor_pool: ["maintainer-1", "maintainer-2"]
stale_days: 45            # default: 30
stale_grace_days: 14      # default: 7
```

---

## Prerequisites

- **GitHub CLI** (`gh`) — required for generating and managing workflows
  ```bash
  brew install gh && gh auth login
  ```
- **ANTHROPIC_API_KEY** — set as a repo secret for agents to call Claude
  ```bash
  gh secret set ANTHROPIC_API_KEY --body "sk-ant-..."
  ```
- **gh aw extension** (optional) — enables gh-aw markdown format with safe-outputs
  ```bash
  gh extension install github/gh-aw
  ```

---

## Roadmap

### v0.2
- `/awful:sim` — dry-run workflows locally with simulated event payloads
- `/awful:audit` — loop risk detection, permission audit, coverage gap report
- `/awful:update` — re-design and regenerate a single agent without touching others
- `safe-outputs` config in blueprint (per-agent operation allowlists)

### v0.3
- Multi-repo blueprints (agent in repo A triggers event in repo B)
- Metrics collection (agent run counts, label distribution, response times)
- `/awful:incident-update` slash command support

---

## License

ISC — Copyright 2026 xiaolai

Part of the [xiaolai Claude Code plugin collection](https://github.com/xiaolai/claude-plugin-marketplace).
