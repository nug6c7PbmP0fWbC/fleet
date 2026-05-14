# opsx sync

Synchronize local Fleet configuration state with the remote Fleet instance, detecting drift and optionally reconciling differences.

## Usage

```
/opsx sync [--dry-run] [--direction <push|pull|both>] [--scope <scope>] [--force]
```

## Arguments

- `--dry-run` — Show what would be synced without making changes (default: false)
- `--direction` — Direction of sync: `push` (local → remote), `pull` (remote → local), `both` (default: `both`)
- `--scope` — Limit sync to a specific resource type: `policies`, `queries`, `config`, `labels`, `teams`, `packs`, `enroll_secrets` (default: all)
- `--force` — Skip confirmation prompts and overwrite conflicts with the chosen direction

## Description

The `sync` command compares local YAML/JSON configuration files against the live Fleet instance and identifies:

1. **Local-only resources** — exist in local files but not in Fleet (would be created on push)
2. **Remote-only resources** — exist in Fleet but not locally (would be deleted on push, or created locally on pull)
3. **Diverged resources** — exist in both but differ (conflict requiring resolution)
4. **In-sync resources** — identical in both locations

Sync uses `fleetctl` under the hood and respects the active context set via `fleetctl config set-context`.

## Workflow

### Step 1 — Detect active context

Check `fleetctl config get-context` to determine which Fleet instance is targeted. If no context is set, prompt the user to configure one before proceeding.

### Step 2 — Inventory local files

Walk the working directory (or `FLEET_CONFIG_DIR` env var if set) for:
- `*.yml` / `*.yaml` files containing Fleet resource definitions
- `*.json` files with Fleet resource definitions
- Subdirectories following the convention: `policies/`, `queries/`, `labels/`, `teams/`

### Step 3 — Fetch remote state

Run `fleetctl get` for each resource type in scope:

```bash
fleetctl get policies --json
fleetctl get queries --json
fleetctl get config --json
fleetctl get labels --json
fleetctl get teams --json
```

### Step 4 — Diff and classify

For each resource, compute a normalized diff (strip read-only fields like `id`, `created_at`, `updated_at`) and classify as: `in-sync`, `local-only`, `remote-only`, or `diverged`.

### Step 5 — Present summary

Display a table:

```
Resource Type   Name                    Status
─────────────   ─────────────────────   ──────────────
policy          No USB storage          in-sync
policy          Require FileVault        diverged  ← local differs
query           Get running processes   local-only
label           Engineering Macs        remote-only
```

For `diverged` resources, show a condensed diff unless `--dry-run` is passed (in which case always show full diff).

### Step 6 — Reconcile (if not dry-run)

Based on `--direction`:

- **push**: Apply local files to Fleet via `fleetctl apply -f <file>`. Remove remote-only resources only if `--force` is passed (otherwise warn).
- **pull**: Write remote resources to local files. Overwrite diverged local files only if `--force` is passed.
- **both**: Push local-only, pull remote-only, and prompt for each diverged resource unless `--force` is set.

## Output

On success:
```
✓ Sync complete
  pushed: 2 resources
  pulled: 1 resource  
  skipped (conflicts): 1 resource
  in-sync: 4 resources
```

On failure, print the `fleetctl` error output and exit non-zero.

## Environment Variables

- `FLEET_CONFIG_DIR` — Override the directory scanned for local config files (default: current working directory)
- `FLEET_CONTEXT` — Override the active fleetctl context
- `FLEET_DRY_RUN` — Set to `1` to default to dry-run mode

## Examples

```bash
# Preview all differences without making changes
/opsx sync --dry-run

# Push only local policy changes to Fleet
/opsx sync --direction push --scope policies

# Pull all remote config down to local files, overwriting conflicts
/opsx sync --direction pull --force

# Full bidirectional sync with confirmation prompts
/opsx sync
```

## Related Commands

- `/opsx diff` — Show differences without syncing
- `/opsx apply` — Apply a specific local file to Fleet
- `/opsx plan` — Plan changes before applying
- `/opsx status` — Show current Fleet instance status
