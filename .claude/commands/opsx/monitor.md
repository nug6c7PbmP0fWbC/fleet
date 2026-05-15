# Monitor Command

Real-time monitoring of Fleet infrastructure resources, services, and health metrics.

## Usage

```
/opsx monitor [resource] [options]
```

## Arguments

- `resource` — The resource type to monitor (optional, defaults to `all`)
  - `all` — Monitor all resources
  - `services` — Fleet services (fleet-server, mysql, redis, etc.)
  - `hosts` — Enrolled host counts and activity
  - `queries` — Live query execution and results
  - `ingestion` — Host data ingestion pipeline
  - `errors` — Error rates and recent errors

## Options

- `--interval <seconds>` — Refresh interval in seconds (default: `10`)
- `--duration <minutes>` — How long to monitor before stopping (default: indefinite)
- `--env <environment>` — Target environment (default: from context)
- `--threshold <metric=value>` — Alert when metric exceeds threshold
- `--output <format>` — Output format: `live`, `json`, `csv` (default: `live`)
- `--no-color` — Disable colored output

## Examples

### Monitor all resources
```
/opsx monitor
```

### Monitor Fleet services every 5 seconds
```
/opsx monitor services --interval 5
```

### Monitor host ingestion with alert threshold
```
/opsx monitor ingestion --threshold error_rate=0.05
```

### Monitor for 30 minutes and output as JSON
```
/opsx monitor --duration 30 --output json
```

## Output

### Live Dashboard (default)

```
Fleet Monitor — production — 2024-01-15 14:32:01 UTC
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

SERVICES
  fleet-server     ✓ healthy   cpu: 12%   mem: 1.2GB   uptime: 14d 3h
  mysql            ✓ healthy   cpu: 8%    mem: 4.1GB   connections: 42/200
  redis            ✓ healthy   cpu: 2%    mem: 512MB   keys: 18,432
  nginx            ✓ healthy   cpu: 1%    mem: 128MB   req/s: 847

HOSTS
  total enrolled:  24,891
  online (15min):  18,203  (73.1%)
  new today:       +127
  failing policy:  342  (1.4%)

INGESTION (last 60s)
  host checkins:   1,847/min
  results stored:  42,103/min
  error rate:      0.02%
  queue depth:     1,203

QUERIES
  live queries:    3 active
  scheduled runs:  892 (last hour)
  avg duration:    143ms

[q] quit  [r] refresh  [d] details  [?] help
```

### JSON Output

When `--output json` is specified, emits newline-delimited JSON snapshots:

```json
{"timestamp":"2024-01-15T14:32:01Z","env":"production","services":{"fleet-server":{"status":"healthy","cpu_pct":12,"mem_bytes":1288490188}},"hosts":{"total":24891,"online":18203},"ingestion":{"checkins_per_min":1847,"error_rate":0.0002}}
```

## Metrics Reference

| Metric | Description | Alert Default |
|--------|-------------|---------------|
| `cpu_pct` | CPU utilization percentage | 80% |
| `mem_pct` | Memory utilization percentage | 85% |
| `error_rate` | Errors per total requests | 5% |
| `queue_depth` | Ingestion queue backlog | 10,000 |
| `host_online_pct` | Percentage of hosts online | <50% |
| `query_duration_p99` | 99th percentile query duration | 5000ms |

## Behavior

1. Reads environment configuration from current ops context
2. Connects to configured metrics endpoints (Prometheus, CloudWatch, or direct)
3. Polls at specified interval and renders updated dashboard
4. Triggers alerts to stdout (or configured webhook) when thresholds are exceeded
5. On exit, prints a summary of any threshold violations observed

## Related Commands

- `/opsx status` — Point-in-time status snapshot
- `/opsx logs` — Stream or search service logs
- `/opsx explore` — Browse infrastructure resources interactively
