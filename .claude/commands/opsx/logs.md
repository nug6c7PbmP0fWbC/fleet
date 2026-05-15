# OpsX Logs Command

Stream and inspect operational logs from Fleet infrastructure components.

## Usage

```
/opsx logs [component] [options]
```

## Arguments

- `component` — Target component to fetch logs from (e.g., `fleet-server`, `mysql`, `redis`, `fleet-worker`). Defaults to `fleet-server`.

## Options

| Flag | Description | Default |
|------|-------------|---------|
| `--tail <n>` | Number of lines to show from the end | `100` |
| `--follow` / `-f` | Stream logs in real-time | `false` |
| `--since <duration>` | Show logs since a relative time (e.g., `5m`, `1h`, `24h`) | `1h` |
| `--level <level>` | Filter by log level: `debug`, `info`, `warn`, `error` | (all) |
| `--grep <pattern>` | Filter log lines matching a regex pattern | (none) |
| `--env <env>` | Target environment: `prod`, `staging`, `dev` | `dev` |
| `--json` | Output raw JSON log lines without formatting | `false` |
| `--no-color` | Disable colored output | `false` |

## Examples

### Tail the last 200 lines from the Fleet server
```
/opsx logs fleet-server --tail 200
```

### Stream live logs from the worker process
```
/opsx logs fleet-worker --follow
```

### Show errors from the last 30 minutes in staging
```
/opsx logs fleet-server --since 30m --level error --env staging
```

### Search for a specific host UUID in recent logs
```
/opsx logs fleet-server --grep "abc123-host-uuid" --since 2h
```

### Export raw JSON logs for parsing
```
/opsx logs mysql --json --since 1h > mysql-logs.json
```

## Behavior

1. **Authentication** — Resolves credentials for the target environment using the configured cloud provider (AWS CloudWatch, GCP Cloud Logging, or local Docker). Fails fast if credentials are missing or expired.

2. **Component resolution** — Maps the component name to the underlying log group or container name. Known aliases:
   - `fleet-server` → `/ecs/fleet-server` (prod/staging) or container `fleet_fleet-server_1` (dev)
   - `fleet-worker` → `/ecs/fleet-worker` or container `fleet_fleet-worker_1`
   - `mysql` → `/rds/fleet-mysql` or container `fleet_mysql_1`
   - `redis` → `/elasticache/fleet-redis` or container `fleet_redis_1`
   - `nginx` → `/ecs/fleet-nginx` or container `fleet_nginx_1`

3. **Formatting** — By default, JSON log lines are pretty-printed with:
   - Timestamp highlighted in gray
   - Log level color-coded (debug=blue, info=green, warn=yellow, error=red)
   - Message field bolded
   - Remaining fields printed as `key=value` pairs

4. **Filtering** — `--level` filtering is applied client-side after fetching. `--grep` uses Go-compatible regex syntax.

5. **Rate limiting** — When using `--follow` in production, a warning is displayed if log volume exceeds 500 lines/minute to avoid runaway costs.

## Error Handling

- If the component name is unrecognized, the command lists available components and exits with code `1`.
- If the environment is unreachable, a clear error with suggested remediation steps is shown.
- Ctrl+C gracefully stops `--follow` streams and prints a summary of lines received.

## Notes

- In `dev` environments, logs are sourced from Docker Compose via `docker logs`.
- In `staging` and `prod`, logs are fetched from AWS CloudWatch Logs by default. Set `OPSX_LOG_BACKEND=gcp` to use GCP Cloud Logging instead.
- Log retention periods: `prod` = 90 days, `staging` = 14 days, `dev` = session only.
