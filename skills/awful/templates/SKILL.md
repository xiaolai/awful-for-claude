---
name: templates
description: "Use when selecting, customizing, or composing built-in workflow templates — covers all 7 templates (triage, PR review, docs, release, stale, first-timer, incident) with customization points and composition patterns."
version: 0.1.0
---

# Templates

Reference guide for awful's 7 built-in workflow templates.

## Template Overview

| Template | Event | Agent Role | Key Outputs |
|----------|-------|-----------|-------------|
| triage | `issues.opened` | Classify, label, assign | labels, assignment, acknowledgment comment |
| pr-review | `pull_request.opened/synchronize` | Review code quality | review comment, approve/request-changes |
| docs | `issues.labeled:needs-docs` | Generate documentation | docs PR |
| release | `push:tags/v*` | Generate changelog, create release | GitHub release, announcement comment |
| stale | `schedule:daily` | Find inactive issues/PRs | warning comment, close |
| first-timer | `pull_request.opened` (FIRST_TIME_CONTRIBUTOR) | Welcome, guide | welcome comment, first-timer label |
| incident | `issues.labeled:incident` | Create tracker, notify | tracking comment, @mentions, runbook link |

---

## Template Details

### triage

**Event:** `issues.opened`
**Purpose:** Every new issue gets classified immediately so the team knows what it is and how urgent it is.

**Customization points:**
- Add project-specific labels (e.g., `component-auth`, `platform-ios`)
- Adjust priority thresholds (what counts as high vs medium for your project)
- Add auto-assignment rules (e.g., `component-auth` → `@security-team`)
- Change acknowledgment comment tone (formal vs casual)

**Required labels:** `bug`, `feature`, `question`, `duplicate`, `priority-high`, `priority-medium`, `priority-low`

**Agent-applied labels:** `awful:triaged`

---

### pr-review

**Event:** `pull_request.opened`, `pull_request.synchronize`
**Purpose:** Automated first-pass code review — catches obvious issues before human reviewers spend time.

**Fork PR guard:** Required — this template has `contents: read` but the review comment requires `pull-requests: write`.

**Customization points:**
- Add project-specific review checklist items (e.g., "Check for SQL injection", "Verify API versioning")
- Set review standards (strict/standard/light)
- Configure which file types to skip (e.g., lockfiles, generated code)
- Adjust approve threshold (perfect code only vs minor issues ok)

**Agent-applied labels:** `awful:reviewed`

**Human gate:** Agent posts a review but does NOT approve — human still merges.

---

### docs

**Event:** `issues.labeled` with label `needs-docs`
**Purpose:** When a feature is merged without documentation, an issue gets labeled `needs-docs`. The agent reads the related code and generates a documentation PR.

**Customization points:**
- Specify doc format (mdx, rst, asciidoc)
- Set output location (docs/, README sections, wiki)
- Add company style guide constraints to the prompt
- Link to related issues/PRs for context

**Required labels:** `needs-docs`

**Permissions:** `contents: write`, `pull-requests: write` — this template creates a PR.

---

### release

**Event:** `push` to tags matching `v*`
**Purpose:** When a version tag is pushed, generate a changelog from commits since the previous tag and create a GitHub release.

**Customization points:**
- Changelog format (keep-a-changelog, conventional commits, custom)
- Which commit types to include/exclude (skip chores, include breaking changes prominently)
- Release announcement format
- Whether to mark as pre-release (based on tag: `v1.0.0-rc.1` → pre-release)

**Permissions:** `contents: write` — creates the GitHub release object.

**Note:** Does not push code. Creates only the GitHub Release metadata.

---

### stale

**Event:** `schedule` (daily cron, `0 9 * * *`)
**Purpose:** Keep the issue tracker clean by closing issues that have had no activity for 37 days (30 days warning + 7 days grace).

**Customization points:**
- Inactivity threshold (default: 30 days)
- Grace period after warning (default: 7 days)
- Exempt labels (default: `priority-high`, `pinned`)
- Warning comment text
- Whether to apply to PRs (default: yes)

**Required labels:** `awful:stale` (applied by the agent as warning)
**Exempt labels:** `priority-high`, `pinned`

**Idempotent:** Safe to run multiple times — already-warned issues won't get duplicate comments.

---

### first-timer

**Event:** `pull_request.opened` with `author_association == 'FIRST_TIME_CONTRIBUTOR'`
**Purpose:** Welcome first-time contributors warmly, set expectations, and flag for extra-careful review.

**Customization points:**
- Welcome message tone and content
- Links to include (CONTRIBUTING.md, code of conduct, Discord, etc.)
- Whether to auto-assign a reviewer from a mentor pool
- Whether to label `good-first-review` for team awareness

**Required labels:** `first-timer`

**Note:** Does not request changes or add friction. Pure welcome + guidance.

---

### incident

**Event:** `issues.labeled` with label `incident`
**Purpose:** When someone labels an issue as an incident, the agent immediately structures the response: severity assessment, tracker comment, team notification.

**Customization points:**
- Severity keywords (what words in title/body indicate SEV1 vs SEV2 vs SEV3)
- Team @mentions per severity level
- Runbook location (path in repo or external URL)
- Incident template (what sections to include in tracking comment)
- Escalation thresholds

**Required labels:** `incident`, `sev1`, `sev2`, `sev3`

**Agent-applied labels:** `awful:incident-tracked`

---

## Composition Patterns

### Basic OSS

Minimal automation for an open source project:

```
triage + stale + first-timer
```

- Every issue gets classified
- Inactive issues get cleaned up
- New contributors get welcomed

### Full CI

For a project with active development and documentation:

```
triage + pr-review + docs + release
```

- Issues classified on arrival
- PRs get automated code review
- Docs stay current with code
- Releases are automated

### Production Service

Maximum automation for a production service:

```
triage + pr-review + docs + release + stale + first-timer + incident
```

All 7 templates. Incident response is the addition that makes this appropriate for production.

### Custom Design

For requirements that don't fit built-in templates:

1. Run `/awful:design` — architect agent interviews you and builds a custom blueprint
2. Run `/awful:gen` — generator produces workflow files
3. Use built-in templates as starting points in the blueprint

Templates are referenced in the generator's system prompt. When an agent role in the blueprint matches a template name, the generator uses that template as a base and applies blueprint customizations on top.
