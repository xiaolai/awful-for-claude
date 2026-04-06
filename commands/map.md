---
name: map
description: "Visualize event→workflow→agent routing as a mermaid diagram"
allowed-tools: Read, Glob, Bash
---

# /awful:map

Visualize the event→workflow→agent routing for the current repository's awful system. Produces a mermaid diagram and a routing table.

## Steps

### Step 1: Read Routing Data

Try sources in order:

1. **Blueprint (preferred):** Read `workflows/BLUEPRINT.md` YAML frontmatter → use `routing:` array
2. **Generated workflow files:** Scan `.github/workflows/*.md` and `.github/workflows/*.yml` for trigger events and concurrency groups
3. **Both:** If both exist, use blueprint as primary and note any drift (files that don't match the blueprint)

If no data source is available:
```
No blueprint or workflow files found.
Run /awful:init and /awful:design first.
```

### Step 2: Build Routing Table

From the routing data, construct:

| Event | Condition | Agent | Concurrency Group | Loop Guard |
|-------|-----------|-------|-------------------|------------|
| issues.opened | — | triage | awful-triage-{issue.number} | bot-actor |
| issues.labeled | label == 'incident' | incident | awful-incident-{issue.number} | bot-actor |
| pull_request.opened | same-repo | pr-review | awful-pr-review-{pr.number} | bot-actor, fork-guard |
| schedule:daily | — | stale | awful-stale-sweep | — |
| ... | | | | |

Identify and flag:
- **Cascades:** Agent A's output label matches Agent B's trigger condition — draw a cascade arrow
- **Human gates:** Routes that require human approval before an agent acts
- **Potential loops:** Routes where an agent could trigger itself

### Step 3: Generate Mermaid Diagram

Build a `flowchart LR` diagram:

```
flowchart LR
    subgraph Events["GitHub Events"]
        E1["issues.opened"]
        E2["issues.labeled\n(incident)"]
        E3["pull_request.opened\n(same-repo)"]
        E4["schedule\n(daily)"]
        ...
    end

    subgraph Agents["Agents"]
        A1["triage\n(sonnet)"]
        A2["incident\n(sonnet)"]
        A3["pr-review\n(sonnet)"]
        A4["stale\n(sonnet)"]
        ...
    end

    subgraph Outputs["Outputs"]
        O1["labels +\ncomment"]
        O2["tracking comment\n+ @mentions"]
        O3["PR review"]
        O4["close +\ncomment"]
        ...
    end

    E1 -->|"bot-actor guard"| A1
    E2 -->|"bot-actor guard"| A2
    E3 -->|"bot-actor + fork guard"| A3
    E4 --> A4

    A1 --> O1
    A2 --> O2
    A3 --> O3
    A4 --> O4
```

Color coding (applied via class definitions):
- Green (`fill:#22c55e`) — normal routes
- Red (`fill:#ef4444`) — cascade paths (agent output triggers another agent)
- Yellow (`fill:#eab308`) — human gate required before agent acts
- Blue (`fill:#3b82f6`) — event nodes

### Step 4: Display Results

Output in this order:

1. **Routing Table** (markdown table from Step 2)
2. **Mermaid Diagram** in a fenced code block
3. **Analysis** section:
   - Number of agents wired
   - Number of events handled
   - Cascades identified (with path descriptions)
   - Human gates
   - Any unhandled events (events in blueprint routing but no matching workflow file)
   - Any orphaned files (workflow files with no blueprint entry)

Example analysis:

```
Analysis:
- 4 agents wired to 5 events
- 1 cascade: triage → specialist (via label:needs-specialist)
- 1 human gate: pr-review → merge (human merges the PR)
- All blueprint entries have corresponding workflow files
```
