# OpSx Help Command

Provide a comprehensive overview of all available OpSx commands, their usage, and examples.

## Usage

```
/opsx help [command]
```

## Arguments

- `command` (optional): Get detailed help for a specific command

## Behavior

When invoked without arguments, display a summary of all available OpSx commands with brief descriptions.

When invoked with a specific command name, display detailed help for that command including:
- Full description
- All available flags and options
- Usage examples
- Related commands

## Command Reference

Display the following command list when no argument is provided:

### Infrastructure Commands

| Command | Description |
|---------|-------------|
| `init` | Initialize a new OpSx environment or configuration |
| `plan` | Generate and preview an execution plan for infrastructure changes |
| `apply` | Apply planned infrastructure changes to the target environment |
| `rollback` | Revert the most recent apply operation to the previous state |
| `validate` | Validate configuration files and environment setup |

### Observability Commands

| Command | Description |
|---------|-------------|
| `status` | Show current status of all managed resources and services |
| `logs` | Stream or retrieve logs from fleet services and infrastructure |
| `diff` | Show differences between current and desired infrastructure state |

### Data Commands

| Command | Description |
|---------|-------------|
| `sync` | Synchronize configuration or data between environments |
| `archive` | Archive logs, configs, or state snapshots for long-term storage |
| `explore` | Interactively browse infrastructure resources and their relationships |

### Meta Commands

| Command | Description |
|---------|-------------|
| `test` | Run integration and smoke tests against the target environment |
| `help` | Show this help message or detailed help for a specific command |

## Global Flags

The following flags are available across all OpSx commands:

- `--env <environment>` — Target environment (e.g., `production`, `staging`, `dev`). Defaults to value in `OPSX_ENV` env var.
- `--config <path>` — Path to OpSx configuration file. Defaults to `.opsx/config.yaml`.
- `--dry-run` — Preview actions without making any changes (supported by most commands).
- `--verbose` — Enable verbose output for debugging.
- `--json` — Output results as JSON for programmatic use.
- `--no-color` — Disable colored terminal output.

## Environment Variables

| Variable | Description |
|----------|-------------|
| `OPSX_ENV` | Default target environment |
| `OPSX_CONFIG` | Path to configuration file |
| `OPSX_LOG_LEVEL` | Logging verbosity (`debug`, `info`, `warn`, `error`) |
| `OPSX_STATE_BUCKET` | S3 bucket or GCS bucket for remote state storage |
| `FLEET_API_TOKEN` | Fleet API token for authenticated operations |

## Quick Start

```bash
# Initialize a new environment
/opsx init --env staging

# Validate your configuration
/opsx validate

# Preview changes before applying
/opsx plan --env staging

# Apply changes
/opsx apply --env staging

# Check status after deployment
/opsx status --env staging

# View recent logs
/opsx logs --env staging --tail 100
```

## Getting Help

For detailed help on any command, run:

```
/opsx help <command>
```

For example:
- `/opsx help apply` — Detailed apply command documentation
- `/opsx help logs` — Log streaming options and filters
- `/opsx help rollback` — Rollback strategies and safety checks

## Related Documentation

- See `.claude/README.md` for full OpSx architecture overview
- See `.claude/CLAUDE.md` for Claude-specific integration notes
- Fleet documentation: https://fleetdm.com/docs
