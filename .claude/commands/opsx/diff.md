# opsx diff

Analyze and summarize differences between Fleet versions, configurations, or database migrations.

## Usage

```
/opsx diff [target] [options]
```

## Arguments

- `target` — What to diff: `migrations`, `config`, `schema`, `api`, or a specific file path
- `--from` — Source version, branch, or commit (default: main)
- `--to` — Target version, branch, or commit (default: current HEAD)
- `--format` — Output format: `summary`, `detailed`, `json` (default: summary)
- `--focus` — Narrow diff to a domain: `auth`, `mdm`, `osquery`, `vulnerabilities`, `policies`

## Behavior

When invoked, this command will:

1. **Identify the diff target** — Determine whether the user wants to compare migrations, API surface, config schema, or arbitrary files.

2. **Collect relevant changes** — Use git diff or file comparison to gather the raw differences.

3. **Analyze impact** — For each change, assess:
   - Breaking vs. non-breaking changes
   - Security implications
   - Database migration ordering and safety
   - API backward compatibility
   - Configuration deprecations or new required fields

4. **Produce a structured summary** — Group findings by category and severity.

## Examples

### Compare database migrations between branches
```
/opsx diff migrations --from main --to feature/mdm-improvements
```

Expected output:
```
Migration Diff Summary
======================
New migrations (3):
  + 20240115120000_add_mdm_profile_labels.go
  + 20240116083000_add_script_timeout_column.go
  + 20240117140000_index_host_mdm_actions.go

Analysis:
  ✓ All migrations are additive (no destructive changes)
  ✓ Index migration includes IF NOT EXISTS guard
  ⚠ Script timeout column has no default — verify backfill logic
```

### Summarize API changes
```
/opsx diff api --from v4.45.0 --to v4.46.0 --format detailed
```

### Check config schema drift
```
/opsx diff config --focus auth
```

## Output Format

### Summary (default)
High-level bullet points grouped by: Added, Modified, Removed, Warnings.

### Detailed
Full context per change including file path, line numbers, and inline explanation of impact.

### JSON
Machine-readable output suitable for piping into other commands or CI checks:
```json
{
  "from": "main",
  "to": "HEAD",
  "migrations": {
    "added": [...],
    "modified": [],
    "removed": []
  },
  "warnings": [...],
  "breaking_changes": []
}
```

## Integration with Other Commands

- Pipe into `/opsx apply` to review changes before applying migrations
- Use before `/opsx archive` to document what changed in a release
- Combine with `/opsx explore` to deep-dive into a specific changed area

## Notes

- For migration diffs, always check that new migrations have timestamps greater than all existing ones
- Flag any migration that drops columns, tables, or indexes as high-risk
- When diffing configs, highlight any new required fields that lack defaults (deployment risk)
- API diffs should note if any endpoint changes affect the Fleet UI, fleetctl, or the REST API clients
