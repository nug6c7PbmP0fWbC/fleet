# OpsX Plan Command

Generate an execution plan for infrastructure or configuration changes before applying them.

## Usage

```
/opsx:plan [target] [--format=<format>] [--out=<file>]
```

## Arguments

- `target` — Optional. Specific resource, module, or path to plan. Defaults to the current working context.
- `--format` — Output format: `table` (default), `json`, or `markdown`.
- `--out` — Write the plan to a file instead of stdout.

## Description

The `plan` command performs a dry-run analysis of pending changes and produces a human-readable diff of what *would* happen if `/opsx:apply` were executed. No changes are made to any environment.

This command is intended to be run **before** `/opsx:apply` to review the blast radius of a change, catch unintended modifications, and build a shared understanding across the team.

## Behavior

1. **Resolve context** — Determine the current environment (staging, production, etc.) from the active config or explicit flags.
2. **Detect drift** — Compare the desired state (local config/code) against the last-known applied state stored in `.claude/state/`.
3. **Classify changes** — Each change is labeled as one of:
   - `ADD` — New resource or configuration to be created.
   - `MODIFY` — Existing resource with one or more fields changing.
   - `DESTROY` — Resource to be removed.
   - `NO-OP` — No change detected.
4. **Risk scoring** — Assign a risk level (`low`, `medium`, `high`, `critical`) based on the type and scope of change.
5. **Output summary** — Print a structured plan with change counts and risk summary.

## Output Example

```
OpsX Plan — fleet/production
─────────────────────────────────────────────────────
Context : production
Target  : all
Generated: 2024-11-15T14:32:01Z

Changes detected: 4
  + 1 to add
  ~ 2 to modify
  - 1 to destroy

─────────────────────────────────────────────────────
RESOURCE                          ACTION    RISK
─────────────────────────────────────────────────────
fleet/config/osquery_config       MODIFY    low
fleet/config/smtp_settings        MODIFY    medium
fleet/integrations/jira           ADD       low
fleet/labels/legacy_windows_xp   DESTROY   high
─────────────────────────────────────────────────────

Risk Summary: 1 high-risk change detected.
Review DESTROY operations carefully before applying.

Run `/opsx:apply` to execute this plan.
Run `/opsx:diff fleet/labels/legacy_windows_xp` for details.
```

## Risk Levels

| Level    | Description                                                                 |
|----------|-----------------------------------------------------------------------------|
| low      | Additive or non-breaking changes (new integrations, label additions, etc.)  |
| medium   | Config updates that may affect behavior (SMTP, webhook URLs, thresholds)    |
| high     | Destructive operations or changes to auth/security configuration            |
| critical | Changes to production secrets, TLS certs, or database migrations            |

## Flags

| Flag         | Description                                              |
|--------------|----------------------------------------------------------|
| `--format`   | `table` (default), `json`, `markdown`                    |
| `--out`      | Path to write plan output (e.g., `--out=plan.json`)      |
| `--no-color` | Disable ANSI color output for CI environments            |
| `--env`      | Override environment detection (e.g., `--env=staging`)   |

## Integration with Other Commands

- Run `/opsx:status` first to confirm the current environment state.
- Use `/opsx:diff <resource>` to inspect individual resource changes in detail.
- Pass the plan output file to `/opsx:apply --plan=<file>` to apply exactly what was planned.
- Use `/opsx:rollback` if an applied plan produces unexpected results.

## Notes

- Plans are not saved automatically. Use `--out` to persist a plan for audit trails or async review.
- In CI pipelines, pipe plan output to a PR comment using `--format=markdown`.
- The plan command never modifies state. It is always safe to run.
