---
description: "Shared: detect repo type, existing workflows, and gh-aw CLI availability"
user-invocable: false
---

# Detect

Run detection checks and return results as structured data for use by init, design, gen, and map commands.

## Checks

### 1. gh-aw Availability

```bash
gh aw --version 2>/dev/null
```

- If exits 0 → `ghaw: true`, capture version string
- If exits non-zero or not found → `ghaw: false`

### 2. Existing GitHub Actions Workflows

```bash
ls .github/workflows/ 2>/dev/null
```

- If directory exists and has files → `existing_workflows: [list of .yml/.yaml files]`
- If directory exists but empty → `existing_workflows: []`
- If directory doesn't exist → `existing_workflows: null`

### 3. Existing awful Blueprint

```bash
ls workflows/BLUEPRINT.md 2>/dev/null
```

- If exists → `blueprint: true`, read and parse YAML frontmatter to get agent list
- If not found → `blueprint: false`

### 4. Repo Type Detection

Check for ecosystem-specific files in priority order:

| File | Repo Type |
|------|-----------|
| `package.json` | Node.js |
| `Cargo.toml` | Rust |
| `pyproject.toml` or `setup.py` or `requirements.txt` | Python |
| `go.mod` | Go |
| `Gemfile` | Ruby |
| `pom.xml` or `build.gradle` | Java/JVM |
| `composer.json` | PHP |
| `.terraform/` or `*.tf` | Terraform/Infrastructure |
| `Dockerfile` or `docker-compose.yml` | Container |
| `*.github/workflows/*.yml` only | GitHub Actions meta-repo |

If multiple match, report all. If none match → `repo_type: unknown`.

Also check:
- `CONTRIBUTING.md` exists? → `has_contributing: true/false`
- `CODE_OF_CONDUCT.md` exists? → `has_coc: true/false`
- Is a fork? Check `git remote get-url origin` and compare to expected

### 5. Existing Label Configuration

```bash
ls .github/labels.yml 2>/dev/null
```

- If exists → `labels_config: true`
- If not → `labels_config: false`

## Output

Return results as a structured summary for the calling command:

```
Detection results:
- gh-aw: {available vX.Y.Z | not available}
- Repo type: {type(s) detected | unknown}
- Existing workflows: {n files | none}
- Blueprint: {found (agents: X) | not found}
- CONTRIBUTING.md: {found | not found}
- CODE_OF_CONDUCT.md: {found | not found}
- labels.yml: {found | not found}
```
