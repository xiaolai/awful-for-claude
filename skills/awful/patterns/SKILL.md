---
name: patterns
description: "Use when designing event routing, preventing cascades, composing workflows, or choosing between label-based vs dispatch-based triggers for GitHub agent workflows."
version: 0.1.0
---

# Event Bus Patterns

GitHub's event system is the backbone of awful workflows. Understanding the key patterns enables reliable, non-looping agent automation.

## Trigger Patterns

### Labels as Routing Keys

The most powerful routing primitive in GitHub: `issues.labeled` fires with the label name available for filtering.

```yaml
on:
  issues:
    types: [labeled]
jobs:
  triage:
    if: github.event.label.name == 'needs-triage'
```

Labels let you build a pure event-driven routing table: each label maps to exactly one agent response.

### repository_dispatch for Cross-Workflow Signaling

When an agent in workflow A needs to trigger workflow B without using a label:

```yaml
# Workflow A sends signal
- name: trigger downstream
  run: |
    gh api repos/${{ github.repository }}/dispatches \
      --field event_type=agent-completed \
      --field client_payload='{"issue":${{ github.event.issue.number }}}'

# Workflow B listens
on:
  repository_dispatch:
    types: [agent-completed]
```

Use sparingly — prefer labels when the routing is user-visible.

### workflow_run Chaining

For sequential agent pipelines where B must always follow A:

```yaml
on:
  workflow_run:
    workflows: ["Triage Agent"]
    types: [completed]
```

Limitation: `workflow_run` only fires for workflows on the default branch.

### Comment Slash Commands

`issue_comment` events let users direct agents from issue/PR threads:

```yaml
on:
  issue_comment:
    types: [created]
jobs:
  handle:
    if: |
      github.event.issue.pull_request != null &&
      startsWith(github.event.comment.body, '/review')
```

Always validate command format strictly before routing — comment bodies are untrusted.

### Schedule-Based Triggers

Periodic tasks (stale cleanup, weekly reports) use cron expressions:

```yaml
on:
  schedule:
    - cron: '0 9 * * 1'  # Monday 9am UTC
```

Cron jobs run on the default branch. They cannot be tied to specific issues/PRs; agents must query state themselves.

---

## Loop Prevention

Loops are the most common failure mode in agent workflows. Three layers of defense.

### Layer 1: Bot Actor Guard

Every write-capable workflow must check:

```yaml
if: github.actor != 'github-actions[bot]'
```

This prevents: agent adds label → label event fires → agent adds label → ...

For workflows that may be triggered by other bots (dependabot, renovate), expand the guard:

```yaml
if: |
  github.actor != 'github-actions[bot]' &&
  github.actor != 'dependabot[bot]'
```

### Layer 2: Label Prefix Convention

Agents apply labels with the `awful:` prefix. Routing conditions match on plain labels only:

- Human applies: `bug`, `priority-high`, `needs-triage` → triggers agent
- Agent applies: `awful:triaged`, `awful:needs-attention` → does NOT trigger agent

This creates a clean separation between signal labels (human intent) and state labels (agent output).

### Layer 3: Concurrency Groups

Prevents duplicate runs for the same issue/PR:

```yaml
concurrency:
  group: awful-${{ github.event.issue.number || github.event.pull_request.number }}
  cancel-in-progress: false
```

`cancel-in-progress: false` queues rather than cancels — ensures the second run still happens after the first completes, but never runs two simultaneously.

---

## Orchestration Patterns

### Fan-Out

One event triggers multiple independent agents. Each agent has its own workflow file:

```
issues.opened
  → triage-agent.yml      (classify, label)
  → auto-assign.yml       (find owner, assign)
  → welcome-agent.yml     (if first-timer)
```

Fan-out is implicit in GitHub Actions — multiple workflows can listen to the same event. No explicit coordination needed.

### Pipeline (Sequential)

Agent A output triggers Agent B via label:

```
issues.opened → triage-agent adds label:needs-specialist
  → issues.labeled:needs-specialist → specialist-agent
    → specialist-agent adds label:awful:specialist-reviewed
```

Design pipelines to terminate — every path must reach a state with no further triggers.

### Gate (Human-in-the-Loop)

Agent proposes, human approves:

```
issues.labeled:needs-fix → fix-proposer-agent
  → fix-proposer creates PR
    → human reviews and merges (or closes)
```

Use comment slash commands (`/approve`, `/reject`) to give humans structured control points.

### Periodic Sweep

Scheduled agent checks state and acts:

```
cron:daily → stale-agent
  → scans all open issues
  → issues warning comments
  → closes after 7-day grace period
```

Sweeps must be idempotent — they may find the same issues multiple times.

---

## Anti-Patterns

### Label Ping-Pong

```
Agent A adds label:X → Agent B removes label:X → Agent A adds label:X...
```

Prevention: never remove a label as a trigger signal. Use separate labels for input vs output.

### Event Storm

A single action creates many events, each triggering a workflow:

```
Agent creates 10 labels → 10 label events → 10 workflow runs
```

Prevention: batch label operations when possible. Add bot actor guard to all triggered workflows.

### Unbounded Cascade

No depth limit on chain reactions — agent A → B → C → D → ... → out of resources.

Prevention: design pipelines with explicit terminal states. Document the max depth in the blueprint.

### Direct Writes Without Safe-Outputs

Agent modifies repo files directly without a review buffer:

```
agent edits src/main.py directly → pushes to main
```

Prevention: agents should create PRs, not push directly. In gh-aw format, use `safe-outputs` to restrict operations.
