# OpsX Test Command

Run tests against Fleet infrastructure configurations and validate deployment readiness.

## Usage

```
/opsx test [target] [options]
```

## Arguments

- `target` (optional): Specific component or environment to test. Defaults to all.
  - `config` — Validate configuration files
  - `connectivity` — Test network connectivity to Fleet services
  - `auth` — Verify authentication and authorization
  - `db` — Test database connectivity and migrations
  - `fleet` — Run Fleet-specific health checks

## Options

- `--env <environment>` — Target environment (dev, staging, prod)
- `--verbose` — Show detailed test output
- `--fail-fast` — Stop on first failure
- `--timeout <seconds>` — Test timeout per check (default: 30)
- `--output <format>` — Output format: `table` (default), `json`, `junit`

## Behavior

1. **Pre-flight checks** — Verify required tools and credentials are available
2. **Configuration validation** — Parse and lint all config files in the workspace
3. **Connectivity tests** — Ping Fleet API endpoints and dependent services
4. **Authentication tests** — Validate API keys, tokens, and certificates
5. **Database tests** — Check MySQL/PostgreSQL connectivity and schema state
6. **Fleet health checks** — Query `/api/v1/fleet/status` and component health
7. **Summary report** — Display pass/fail/skip counts with actionable errors

## Examples

```bash
# Run all tests against current environment
/opsx test

# Test only database connectivity in staging
/opsx test db --env staging

# Full verbose test with JUnit output for CI
/opsx test --env prod --verbose --output junit

# Quick connectivity check with fast failure
/opsx test connectivity --fail-fast --timeout 10
```

## Test Categories

### Config Tests
- YAML/JSON syntax validation
- Required field presence
- Value type and range checks
- Cross-reference consistency (e.g., referenced secrets exist)

### Connectivity Tests
- Fleet server reachability
- Redis/cache layer ping
- S3/object storage access
- SMTP relay (if configured)
- SSO/IdP endpoint availability

### Auth Tests
- Fleet API token validity
- Certificate expiration warnings (< 30 days = warn, < 7 days = fail)
- IAM role/service account permissions
- Vault/secrets manager access

### Database Tests
- Connection pool availability
- Pending migration count
- Replication lag (if replica configured)
- Slow query log accessibility

### Fleet Health Tests
- API server `/healthz` endpoint
- Osquery result ingestion pipeline
- Live query worker availability
- Scheduled query execution status
- MDM enrollment service (if enabled)

## Output Example

```
OpsX Test Results — staging (2024-01-15 14:32:01)
══════════════════════════════════════════════════

[CONFIG]       ✓ fleet.yml syntax valid
[CONFIG]       ✓ All required fields present
[CONFIG]       ✗ osquery_result_log_plugin: value 'filesystem' deprecated in Fleet 4.x

[CONNECTIVITY] ✓ Fleet API (fleet.staging.example.com:443)
[CONNECTIVITY] ✓ Redis (redis.staging.internal:6379)
[CONNECTIVITY] ✗ SMTP relay (mail.example.com:587) — connection refused

[AUTH]         ✓ Fleet API token valid (expires in 45 days)
[AUTH]         ⚠ TLS certificate expires in 22 days
[AUTH]         ✓ S3 bucket read/write permissions

[DATABASE]     ✓ MySQL connection established
[DATABASE]     ✓ No pending migrations
[DATABASE]     ✓ Replication lag < 1s

[FLEET]        ✓ /healthz — ok
[FLEET]        ✓ Osquery ingestion pipeline active
[FLEET]        ✗ MDM enrollment service not responding

──────────────────────────────────────────────────
Results: 11 passed, 3 failed, 1 warning
Exit code: 1 (failures detected)
```

## Exit Codes

| Code | Meaning |
|------|---------|
| `0`  | All tests passed |
| `1`  | One or more tests failed |
| `2`  | Test execution error (tool missing, auth error, etc.) |
| `3`  | Timeout exceeded |

## Integration with CI/CD

Use `--output junit` to generate `test-results.xml` compatible with GitHub Actions, Jenkins, and GitLab CI:

```yaml
# .github/workflows/deploy.yml
- name: Pre-deployment tests
  run: /opsx test --env ${{ env.DEPLOY_ENV }} --output junit
- name: Upload results
  uses: actions/upload-artifact@v3
  with:
    name: opsx-test-results
    path: test-results.xml
```

## Related Commands

- [`/opsx validate`](./validate.md) — Static config validation only (no live checks)
- [`/opsx status`](./status.md) — Runtime status of deployed Fleet instance
- [`/opsx plan`](./plan.md) — Preview changes before applying
