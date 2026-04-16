# CLI Reference

Consolidated reference for all CLI tools in the MRVA workflow.

## sarif-sql

Go CLI for downloading and transforming MRVA results into a SQLite database.

### Global Flags

| Flag | Required | Description |
|------|----------|-------------|
| `--analysis-id` | Yes | MRVA analysis ID. |
| `--controller-repo` | Yes | Controller repository (`owner/name`). |

### Authentication Flags

Used by all `analysis` subcommands. The two methods are mutually exclusive. You can either provide a token or an app-id and private key.

| Flag | Description |
|------|-------------|
| `--token` | GitHub Personal Access Token. |
| `--app-id` | GitHub App ID. |
| `--private-key` | GitHub App private key (PEM content). |
| `--base-url` | GitHub API base URL (default: `https://api.github.com`). For GHES. |

### Commands

#### `analysis start`

Initialize the local workspace directory.

```bash
sarif-sql analysis start \
  --analysis-id <id> \
  --controller-repo <owner/name> \
  --token "$GITHUB_TOKEN"
```

#### `analysis summary`

Fetch analysis status and generate a markdown report.

```bash
sarif-sql analysis summary \
  --analysis-id <id> \
  --controller-repo <owner/name> \
  --token "$GITHUB_TOKEN"
```

#### `analysis download`

Download SARIF artifacts for all scanned repositories.

```bash
sarif-sql analysis download \
  --analysis-id <id> \
  --controller-repo <owner/name> \
  --directory <path> \
  --token "$GITHUB_TOKEN"
```

| Flag | Required | Description |
|------|----------|-------------|
| `--directory` | Yes | Output directory for downloaded artifacts. |

#### `transform`

Parse SARIF files and write a normalized SQLite database.

```bash
sarif-sql transform \
  --analysis-id <id> \
  --controller-repo <owner/name> \
  --sarif-directory <path> \
  --output <path>
```

| Flag | Required | Default | Description |
|------|----------|---------|-------------|
| `--sarif-directory` | Yes | — | Directory containing SARIF files, `analysis.json`, and `repos.json`. |
| `--output` | No | `./output` | Output directory for `mrva-analysis.db`. |

---

## mrva-prep

Go CLI for optimizing the SQLite database for the reporting UI.

### Global Flags

| Flag | Default | Description |
|------|---------|-------------|
| `--db`, `-d` | `mrva.db` | Path to the SQLite database. |

### Commands

#### `index`

Create query-optimized indexes, run `ANALYZE`, and `VACUUM`.

```bash
mrva-prep index --db <path>
```

#### `dashboard`

Pre-aggregate dashboard metrics to `dashboard.json`.

```bash
mrva-prep dashboard --db <path> --output <dir>
```

| Flag | Default | Description |
|------|---------|-------------|
| `--output`, `-o` | `.` | Directory to write `dashboard.json`. |

#### `compress`

Gzip-compress the database (level 9). **Local development only.**

```bash
mrva-prep compress --db <path>
```

#### `all`

Run `index` → `dashboard` → `compress` in sequence.

```bash
mrva-prep all --db <path> --output <dir>
```

| Flag | Default | Description |
|------|---------|-------------|
| `--output`, `-o` | `.` | Directory to write `dashboard.json`. |

