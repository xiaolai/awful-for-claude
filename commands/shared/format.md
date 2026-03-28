---
description: "Shared: select output format based on gh-aw availability and user preference"
user-invocable: false
---

# Format Selection

Determine the output format for workflow file generation.

## Logic

1. Check `.claude/awful.local.md` (or `.github/awful-config.yml`) for an explicit `output_format` setting:
   - If `output_format: ghaw` → use **gh-aw markdown format**
   - If `output_format: actions` → use **GitHub Actions YAML format**
   - If `output_format: auto` or not set → continue to step 2

2. If `ghaw: true` from detect.md results → use **gh-aw markdown format**

3. Otherwise → use **GitHub Actions YAML format**

## Format Comparison

| Aspect | gh-aw Markdown | GitHub Actions YAML |
|--------|----------------|---------------------|
| File extension | `.md` | `.yml` |
| Agent prompt location | Markdown body of workflow file | Separate prompt file |
| Safe-outputs buffer | Yes (with `safe-outputs: true`) | No |
| Compilation step | Required (`gh aw compile`) | Not required |
| Network isolation | Partial (safe-outputs) | None |
| Prerequisite | `gh aw` extension installed | GitHub CLI + Claude CLI |

## Output

Return the selected format to the calling command:

```
Output format: {gh-aw markdown | GitHub Actions YAML}
Reason: {explicit config | gh-aw available | gh-aw not available, using YAML fallback}
```
