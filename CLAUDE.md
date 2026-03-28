# awful

Agent Workflow Framework Using Labels. Turns GitHub repos into agent-powered automation systems.

## Architecture

GitHub IS the infrastructure: events are the message bus, labels are routing keys, comments are agent I/O.

v0.1: design (blueprint) → gen (workflow files) → map (visualization) → init (setup)

## Commands

- commands/init.md — `/awful:init` — initialize awful for a repo
- commands/design.md — `/awful:design` — interactive agent system design → blueprint
- commands/gen.md — `/awful:gen` — generate workflow files from blueprint
- commands/map.md — `/awful:map` — visualize event→workflow→agent routing
- commands/shared/detect.md — detect repo type, gh-aw availability (not user-invocable)
- commands/shared/format.md — output format selection (not user-invocable)

## Agents

- agents/architect.md — opus, designs agent systems from requirements
- agents/generator.md — sonnet, generates workflow files (gh-aw MD or Actions YAML)

## Skills

- skills/awful/patterns/ — event bus patterns, label routing, orchestration
- skills/awful/conventions/ — gh-aw markdown + Actions YAML format specs
- skills/awful/security/ — safe-outputs, permissions, input sanitization
- skills/awful/templates/ — 7 built-in workflow template reference

## Templates

- templates/triage.md — issue classification and routing
- templates/pr-review.md — PR code review
- templates/docs.md — documentation generation on label
- templates/release.md — release management on tag push
- templates/stale.md — stale issue/PR cleanup
- templates/first-timer.md — first-time contributor welcome
- templates/incident.md — incident response on label

## Loop Prevention

- Bot actor guard: `github.actor != 'github-actions[bot]'`
- Label prefix: agent-applied labels use `awful:` prefix
- Concurrency groups per issue/PR number

## Prerequisites

- GitHub CLI (`gh`) installed
- `ANTHROPIC_API_KEY` or `CLAUDE_CODE_OAUTH_TOKEN` as repo secret (for Actions)
- Optional: `gh aw` extension for gh-aw format output
