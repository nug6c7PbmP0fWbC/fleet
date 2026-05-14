# Status Command

Provide a comprehensive operational status report for the Fleet deployment, summarizing health, configuration drift, and pending actions.

## Usage

```
/opsx:status [--env <environment>] [--verbose] [--format <text|json|markdown>]
```

## Arguments

- `--env`: Target environment (default: inferred from context or `production`)
- `--verbose`: Include detailed diagnostics and raw metric values
- `--format`: Output format — `text` (default), `json`, or `markdown`

## What This Command Does

1. **Reads current state** from Terraform state files, Kubernetes manifests, and Fleet API
2. **Compares against desired state** defined in the repository
3. **Identifies drift** between what is deployed and what is committed
4. **Summarizes pending operations** (unapplied diffs, queued migrations, stale archives)
5. **Reports health signals** from Fleet server, MySQL, Redis, and S3/GCS

## Steps

### 1. Gather Environment Context

Determine which environment to inspect:

```bash
# If --env is not provided, check for active kubectl context
kubectl config current-context

# List available environments
ls infra/environments/
```

### 2. Check Fleet Server Health

```bash
# Fleet liveness probe
curl -sf https://<FLEET_HOSTNAME>/healthz | jq .

# Fleet version
curl -sf https://<FLEET_HOSTNAME>/api/v1/fleet/version | jq '.version'

# Active host count
fleetctl get hosts --json | jq 'length'
```

### 3. Check Infrastructure Drift

Run a non-destructive plan to detect drift without applying changes:

```bash
cd infra/environments/<env>
terraform plan -detailed-exitcode -out=/tmp/status.tfplan 2>&1
# Exit code 0 = no changes, 2 = changes detected, 1 = error
```

Interpret exit codes:
- `0` — Infrastructure matches desired state ✅
- `2` — Drift detected, summarize changed resources ⚠️
- `1` — Plan failed, report error and suggest remediation ❌

### 4. Check Kubernetes Workload Health

```bash
# Pod status for Fleet namespace
kubectl get pods -n fleet -o wide

# Recent events (warnings only)
kubectl get events -n fleet --field-selector type=Warning --sort-by='.lastTimestamp' | tail -20

# Deployment rollout status
kubectl rollout status deployment/fleet -n fleet
```

### 5. Check Database and Cache

```bash
# MySQL connectivity and replication lag (if applicable)
fleetctl debug db-migrations --dry-run

# Redis connectivity
kubectl exec -n fleet deployment/fleet -- fleet debug redis-ping
```

### 6. Check Pending Migrations

```bash
# List unapplied database migrations
fleetctl debug db-migrations --show-pending
```

### 7. Summarize Archived and Staged Diffs

Check if any diffs from `/opsx:diff` or archives from `/opsx:archive` are pending review:

```bash
# Look for staged diff files
ls -lt .claude/ops/diffs/ 2>/dev/null | head -10

# Look for recent archives
ls -lt .claude/ops/archives/ 2>/dev/null | head -10
```

## Output Format

### Text (default)

```
═══════════════════════════════════════════
 Fleet OpSx Status — production
 2024-01-15 14:32:07 UTC
═══════════════════════════════════════════

 FLEET SERVER
  Version : 4.42.0
  Health  : ✅ Healthy
  Hosts   : 1,247 online

 INFRASTRUCTURE
  Terraform : ⚠️  2 resources drifted
  Changed   : aws_security_group.fleet_lb
              aws_ecs_service.fleet

 KUBERNETES
  fleet/fleet-server  : ✅ 3/3 Running
  fleet/fleet-worker  : ✅ 2/2 Running
  Recent Warnings     : 0

 DATABASE
  MySQL   : ✅ Connected
  Pending migrations: 0
  Redis   : ✅ Connected

 PENDING OPERATIONS
  Staged diffs   : 1 (infra-ecs-update-2024-01-14.diff)
  Recent archives: 2

 RECOMMENDED ACTIONS
  1. Review staged diff: /opsx:diff --show infra-ecs-update-2024-01-14
  2. Apply infrastructure drift: /opsx:apply --env production
═══════════════════════════════════════════
```

### JSON

When `--format json` is specified, return a structured object suitable for programmatic consumption or piping into other commands.

## Error Handling

- If Fleet API is unreachable, mark server health as `UNKNOWN` and continue with infrastructure checks
- If Terraform state is locked, report the lock holder and timestamp
- If kubectl context does not match the requested environment, warn before proceeding
- Never make changes — this command is strictly read-only

## Related Commands

- `/opsx:diff` — Generate a detailed diff for a specific change
- `/opsx:apply` — Apply pending infrastructure changes
- `/opsx:explore` — Deep-dive exploration of a specific resource or subsystem
- `/opsx:archive` — Archive completed operational artifacts
