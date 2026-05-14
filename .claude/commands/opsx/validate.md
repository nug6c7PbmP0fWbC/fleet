# Validate Command

Validates Fleet configuration, policies, and infrastructure state against expected baselines without making any changes.

## Usage

```
/opsx validate [--scope <scope>] [--strict] [--format <format>]
```

## Arguments

- `--scope` — Limit validation to a specific area: `config`, `policies`, `queries`, `teams`, `integrations`, `all` (default: `all`)
- `--strict` — Treat warnings as errors and fail validation
- `--format` — Output format: `table` (default), `json`, `summary`

## What This Command Does

This command performs read-only validation of the current Fleet state:

1. **Config Validation** — Checks that all configuration files are syntactically valid YAML/JSON and conform to Fleet schema
2. **Policy Drift Detection** — Compares live policies against source-of-truth definitions in the repository
3. **Query Syntax Validation** — Validates osquery SQL syntax for all saved queries
4. **Team Consistency** — Ensures team memberships and role assignments are consistent
5. **Integration Health** — Verifies external integrations (Jira, Zendesk, MDM) are reachable and configured correctly
6. **Label References** — Checks that all label references in policies and queries point to existing labels

## Validation Rules

### Errors (block apply)
- Missing required fields in configuration
- Invalid YAML/JSON syntax
- References to non-existent teams, labels, or users
- Duplicate policy names within the same scope
- Invalid osquery SQL syntax

### Warnings (logged but non-blocking unless --strict)
- Policies without descriptions
- Queries that have never been run
- Teams with no members
- Deprecated configuration options
- Integrations with stale credentials (>90 days)

## Example Output

```
✓ Config files: 12 valid
✓ Policies: 47 match baseline
⚠ Queries: 3 warnings (no recent execution)
✗ Teams: 1 error — team "legacy-ops" references deleted label "ubuntu-18"
✓ Integrations: all reachable

Validation result: FAILED (1 error, 3 warnings)
Run with --format json for machine-readable output.
```

## Integration with Other Commands

- Run `validate` before `apply` to catch issues early
- `plan` automatically runs `validate` internally
- CI pipelines should run `validate --strict` on pull requests
- Use `validate --scope config` for fast pre-flight checks

## Notes

- Validation is always read-only and safe to run in production
- Does not require elevated permissions beyond read access
- Network calls are made only for integration health checks; use `--scope config,policies,queries` to skip them
- Exit codes: `0` = pass, `1` = errors found, `2` = warnings found (only relevant without `--strict`)
