# opsx init

Initialize a new opsx environment configuration for a Fleet deployment target.

## Usage

```
/opsx init [environment] [--template <template>] [--region <region>] [--dry-run]
```

## Description

Scaffolds the necessary configuration files and directory structure for managing a Fleet deployment environment with opsx. This command should be run once per environment before using other opsx commands.

## Arguments

- `environment` — Name of the environment to initialize (e.g., `production`, `staging`, `dev`). Defaults to `staging` if not provided.

## Options

| Flag | Description | Default |
|------|-------------|--------|
| `--template` | Base template to use (`aws-ecs`, `aws-k8s`, `gcp-gke`, `bare-metal`) | `aws-ecs` |
| `--region` | Cloud provider region for the deployment | `us-east-1` |
| `--dry-run` | Preview what would be created without writing files | `false` |
| `--force` | Overwrite existing configuration if present | `false` |

## What Gets Created

Running `init` creates the following structure under `.opsx/`:

```
.opsx/
  environments/
    <environment>/
      config.yaml        # Core environment configuration
      secrets.yaml.tmpl  # Secret references (never stores actual values)
      network.yaml       # VPC/networking topology
      scaling.yaml       # Auto-scaling policies
  templates/
    <template>/          # Symlinked or copied base templates
  .opsx-lock.json        # Lock file tracking initialized state
```

## Example `config.yaml` Output

```yaml
apiVersion: opsx/v1
kind: EnvironmentConfig
metadata:
  name: staging
  created_at: "2024-01-15T10:30:00Z"
  template: aws-ecs
spec:
  region: us-east-1
  fleet:
    version: latest
    image: fleetdm/fleet
    replicas: 2
  database:
    engine: mysql
    version: "8.0"
    instance_class: db.t3.medium
  redis:
    node_type: cache.t3.micro
    num_cache_nodes: 1
  tls:
    enabled: true
    provider: acm
```

## Behavior

1. **Checks for existing config** — If `.opsx/environments/<environment>/` already exists and `--force` is not set, the command exits with an error and instructions to use `--force` or choose a different environment name.

2. **Validates template** — Ensures the specified template is supported. Unsupported templates produce a clear error listing valid options.

3. **Generates secrets template** — Creates `secrets.yaml.tmpl` with placeholder references (e.g., `${SSM:/fleet/staging/db_password}`) rather than actual values. Actual secrets must be populated separately via your secrets manager.

4. **Writes lock file** — Updates `.opsx-lock.json` with the environment metadata so other commands can detect initialized environments.

5. **Prints next steps** — On success, outputs a summary of created files and the recommended next command (`/opsx plan <environment>`).

## Examples

### Initialize a staging environment with defaults
```
/opsx init staging
```

### Initialize production on GKE in us-central1
```
/opsx init production --template gcp-gke --region us-central1
```

### Preview initialization without writing files
```
/opsx init dev --dry-run
```

### Reinitialize an existing environment
```
/opsx init staging --force
```

## Notes

- The `secrets.yaml.tmpl` file is safe to commit — it contains only references, not values.
- After init, run `/opsx validate <environment>` to confirm the configuration is complete before planning.
- Environment names must be lowercase alphanumeric with hyphens only (e.g., `prod-us-east`, `staging-v2`).
- The lock file (`.opsx-lock.json`) should be committed to track which environments have been initialized across the team.

## Related Commands

- `/opsx validate` — Validate configuration after init
- `/opsx plan` — Generate an execution plan for the environment
- `/opsx status` — Check current deployment status
