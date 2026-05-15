# opsx secrets

Manage secrets and sensitive configuration for Fleet infrastructure deployments.

## Usage

```
opsx secrets <subcommand> [options]
```

## Subcommands

### list
List all secrets for the current environment (names only, no values).

```
opsx secrets list [--env <environment>] [--namespace <namespace>]
```

### get
Retrieve a specific secret value (requires elevated permissions).

```
opsx secrets get <secret-name> [--env <environment>] [--output <format>]
```

### set
Create or update a secret value.

```
opsx secrets set <secret-name> <value> [--env <environment>]
opsx secrets set <secret-name> --from-file <path> [--env <environment>]
opsx secrets set <secret-name> --from-env <ENV_VAR> [--env <environment>]
```

### delete
Remove a secret from the store.

```
opsx secrets delete <secret-name> [--env <environment>] [--confirm]
```

### rotate
Rotate a secret to a new value, updating all references.

```
opsx secrets rotate <secret-name> [--env <environment>] [--dry-run]
```

### sync
Sync secrets from one environment to another.

```
opsx secrets sync --from <source-env> --to <target-env> [--keys <key1,key2>]
```

### audit
Show access history and change log for secrets.

```
opsx secrets audit [--secret <name>] [--since <duration>] [--env <environment>]
```

## Options

| Flag | Description | Default |
|------|-------------|--------|
| `--env` | Target environment (prod/staging/dev) | current context |
| `--namespace` | Kubernetes namespace or secret scope | `fleet` |
| `--output` | Output format: `json`, `yaml`, `raw` | `raw` |
| `--confirm` | Skip confirmation prompts | `false` |
| `--dry-run` | Preview changes without applying | `false` |
| `--from-file` | Read secret value from file | — |
| `--from-env` | Read secret value from environment variable | — |

## Secret Backends

opsx supports multiple secret backends configured via `opsx.yaml`:

- **AWS Secrets Manager** — default for production
- **HashiCorp Vault** — for multi-cloud or on-prem
- **Kubernetes Secrets** — for cluster-scoped secrets
- **1Password** — for team-shared credentials
- **Local `.env`** — for development only (never commit)

## Examples

```bash
# List all secrets in staging
opsx secrets list --env staging

# Get the Fleet JWT signing key
opsx secrets get fleet/jwt-key --env prod

# Set a new database password
opsx secrets set fleet/db-password --from-env DB_PASSWORD --env staging

# Rotate the SMTP credentials
opsx secrets rotate fleet/smtp-credentials --env prod --dry-run

# Sync non-sensitive config secrets from staging to dev
opsx secrets sync --from staging --to dev --keys fleet/osquery-enroll-secret

# Audit recent changes to production secrets
opsx secrets audit --env prod --since 7d
```

## Security Notes

- Secret values are **never logged** to stdout, files, or audit trails
- All `get` operations require MFA confirmation in production
- Secrets are encrypted at rest using envelope encryption
- Rotation automatically updates Kubernetes secret references and triggers rolling restarts
- Use `--dry-run` with `rotate` to preview which deployments will be affected

## Fleet-Specific Secrets

Common Fleet secrets managed by opsx:

| Secret Name | Description |
|-------------|-------------|
| `fleet/jwt-key` | JWT signing key for Fleet server |
| `fleet/db-password` | MySQL/PostgreSQL database password |
| `fleet/redis-password` | Redis auth token |
| `fleet/osquery-enroll-secret` | osquery enrollment secret |
| `fleet/smtp-credentials` | SMTP username and password |
| `fleet/s3-credentials` | S3 access key and secret |
| `fleet/license-key` | Fleet Premium license key |
| `fleet/sso-metadata` | SAML IdP metadata or cert |

## Related Commands

- [`opsx apply`](apply.md) — Apply configuration that references secrets
- [`opsx validate`](validate.md) — Validate that required secrets exist before deployment
- [`opsx diff`](diff.md) — Show config diffs (secrets are redacted)
