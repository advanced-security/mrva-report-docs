# Preparing the Report

After transforming the SARIF data into a SQLite database with `sarif-sql`, use the `mrva-prep` CLI to optimize the database for the reporting UI.

## Install mrva-prep

Download the latest release binary from the [releases page](https://github.com/ghas-projects/mrva-prep/releases):

```bash
# Linux (amd64)
gh release download --repo ghas-projects/mrva-prep \
  --pattern 'mrva-prep-linux-amd64' \
  --output mrva-prep
chmod +x mrva-prep
```

## Global Flags

All `mrva-prep` commands share one flag:

| Flag | Default | Description |
|------|---------|-------------|
| `--db`, `-d` | `mrva.db` | Path to the SQLite database. |

Run any command with `--help` for full usage details.

## Step 1: Create Indexes

Add query-optimized indexes that match the access patterns of the Blazor WebAssembly UI:

```bash
mrva-prep index --db ./output/mrva-analysis.db
```

This creates two indexes:

| Index | Column | Purpose |
|-------|--------|---------|
| `idx_alert_rule_row_id` | `alert.rule_row_id` | Accelerates alert-to-rule joins (severity lookups, rule-grouped counts). |
| `idx_alert_repository_row_id` | `alert.repository_row_id` | Accelerates alert-to-repository joins (per-repo alert counts). |

After creating the indexes, the command runs `ANALYZE` (updates query planner statistics) and `VACUUM` (reclaims space and defragments the file).

## Step 2: Generate Dashboard Metrics

Pre-aggregate the dashboard statistics into a lightweight JSON file:

```bash
mrva-prep dashboard \
  --db ./output/mrva-analysis.db \
  --output ./output
```

| Flag | Default | Description |
|------|---------|-------------|
| `--output`, `-o` | `.` | Directory to write `dashboard.json` into. |

The command executes five aggregation queries and writes the results to `dashboard.json`:

| Metric | Description |
|--------|-------------|
| Scalar counts | Total alerts, repositories, rules, and distinct counts of repos/rules with alerts. |
| Severity counts | Alert count grouped by severity level. |
| Top 10 rules | Rules ranked by alert count. |
| Top 10 repositories | Repositories ranked by alert count. |
| Top 10 file paths | File path / repository combinations ranked by alert count. |

The dashboard JSON also includes the full analysis metadata (tool version, query language, timestamps, repository category counts).

This file enables the report's instant first-paint — the dashboard page renders from this < 2 KB file while the full database loads in the background.

## Step 3: Compress the Database (Local Development Only)

For local development, create a gzip-compressed copy of the database:

```bash
mrva-prep compress --db ./output/mrva-analysis.db
```

This writes `mrva-analysis.db.gz` alongside the original file using maximum compression (gzip level 9). The Blazor WASM UI fetches and decompresses this file client-side.

> **Note:** This step is **not required for production deployments**. The `dotnet publish` step in the CI/CD pipeline handles gzip compression automatically. The `compress` command exists only for local testing.

## Run All Steps at Once

The `all` command runs index, dashboard, and compress in sequence:

```bash
mrva-prep all \
  --db ./output/mrva-analysis.db \
  --output ./output
```

| Flag | Default | Description |
|------|---------|-------------|
| `--output`, `-o` | `.` | Directory to write `dashboard.json` into. |

## End-to-End Example

Continuing from the `sarif-sql transform` output:

```bash
DB_PATH="./output/mrva-analysis.db"

# Add indexes
mrva-prep index --db "$DB_PATH"

# Generate dashboard metrics
mrva-prep dashboard --db "$DB_PATH" --output ./output

# (Local dev only) Compress for browser delivery
mrva-prep compress --db "$DB_PATH"
```

After this step, the `./output` directory contains:
- `mrva-analysis.db` — indexed and vacuumed SQLite database.
- `dashboard.json` — pre-aggregated dashboard metrics.
- `mrva-analysis.db.gz` — gzip-compressed database (if `compress` was run).
