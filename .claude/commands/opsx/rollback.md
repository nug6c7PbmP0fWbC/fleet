# Rollback Command

Rollback a Fleet deployment to a previous version or state.

## Usage

```
/opsx rollback [target] [options]
```

## Arguments

- `target` — The deployment target to rollback (e.g., `production`, `staging`, `fleet-server`)
- `--to <version>` — Specific version or git SHA to rollback to
- `--steps <n>` — Number of deployment steps to roll back (default: 1)
- `--dry-run` — Preview rollback actions without executing
- `--force` — Skip confirmation prompts
- `--timeout <duration>` — Maximum time to wait for rollback completion (default: 10m)

## Description

The rollback command reverts a Fleet deployment to a previous known-good state. It supports rolling back:

- **Application versions** — Revert to a previous container image or binary
- **Database migrations** — Run down migrations to revert schema changes
- **Configuration changes** — Restore previous environment configuration
- **Infrastructure state** — Revert Terraform or Kubernetes manifests

## Process

1. **Pre-flight checks** — Validate current deployment state and target version availability
2. **Backup current state** — Snapshot current config and record rollback metadata
3. **Stop traffic** — Drain connections and pause health checks
4. **Apply rollback** — Deploy previous version or restore configuration
5. **Run down migrations** — If schema changes are involved, execute rollback migrations
6. **Verify health** — Wait for services to pass health checks
7. **Resume traffic** — Re-enable load balancer routing
8. **Notify** — Post rollback summary to configured channels

## Examples

### Rollback production to previous deploy
```
/opsx rollback production
```

### Rollback to a specific version
```
/opsx rollback production --to v4.52.1
```

### Rollback staging two steps
```
/opsx rollback staging --steps 2
```

### Preview rollback without executing
```
/opsx rollback production --dry-run
```

## Safety Checks

Before executing a rollback, the agent will:

- Confirm the target version exists and is accessible
- Check if database migrations need to be reversed (warns if irreversible)
- Verify rollback won't break dependent services
- Require explicit confirmation unless `--force` is passed
- Detect if a rollback is already in progress

## Database Migration Warnings

If the rollback involves reversing database migrations, the agent will:

1. List all migrations that will be reversed
2. Flag any **destructive** down migrations (data loss risk)
3. Require additional confirmation for destructive operations
4. Recommend taking a database snapshot before proceeding

## Output

The rollback command produces a structured summary:

```
Rollback Summary
================
Target:          production
From version:    v4.53.0 (deployed 2h ago)
To version:      v4.52.1
Migrations:      2 down migrations
Estimated time:  ~4 minutes
Status:          SUCCESS

Steps completed:
  ✓ Pre-flight checks passed
  ✓ State backup created
  ✓ Traffic drained (45 connections)
  ✓ Rolled back to v4.52.1
  ✓ Reversed 2 migrations
  ✓ Health checks passed (3/3 instances)
  ✓ Traffic resumed
  ✓ Slack notification sent
```

## Related Commands

- `/opsx deploy` — Deploy a new version
- `/opsx status` — Check current deployment status
- `/opsx diff` — Compare versions before rolling back
- `/opsx archive` — Archive deployment artifacts
