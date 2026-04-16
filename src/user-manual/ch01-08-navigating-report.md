# Navigating the Report

The MRVA report is a single-page application that runs entirely in the browser. It uses a two-phase loading strategy: the dashboard renders instantly from a lightweight JSON file while the full SQLite database downloads in the background. Once the database loads, the Alerts, Repositories, and Rules pages become available.

## Dashboard (`/`)

The landing page. Displays the full analysis overview using pre-aggregated metrics.

### Analysis Metadata

A summary card showing the analysis ID, date, start/end times, tool name, controller repository, query language, state, status, failure reason (if any), and the Actions workflow run ID.

### Repository Breakdown

Clickable cards showing counts for each repository category:

| Card | Description | Click Action |
|------|-------------|--------------|
| Total | All repositories in the analysis. | Navigate to `/repo`. |
| Scanned | Repositories successfully scanned. | Navigate to `/repo?status=succeeded`. |
| Skipped | Repositories skipped due to access mismatch. | Navigate to `/repo?status=access_mismatch`. |
| Not Found | Repositories not found. | Navigate to `/repo?status=not_found`. |
| No CodeQL DB | Repositories without a CodeQL database. | Navigate to `/repo?status=no_codeql_db`. |
| Over Limit | Repositories exceeding the analysis limit. | Navigate to `/repo?status=over_limit`. |

### Summary Cards

Three cards showing total alerts, repositories, and rules. Each card navigates to its respective list page when clicked.

### Severity Pie Chart

Alert distribution by severity level. Clicking a slice navigates to the alerts page filtered by that severity.

### Top 10 Tables

- Top Rules - Rules ranked by alert count. Clicking a row navigates to alerts filtered by that rule.
- Top Repositories - Repositories ranked by alert count.
- Top File Paths - File path / repository combinations ranked by alert count.

### Coverage Charts

Two pie charts showing:
- Repository coverage - Proportion of repositories with and without alerts.
- Rule coverage - Proportion of rules with and without alerts.

<br>
<div style="width:70vw; margin-left:calc(50% - 35vw);">
<video src="../videos/dashboard.mp4" autoplay loop muted playsinline style="width:100%"></video>
</div>


## Alerts (`/alert`)

A server-side paginated data grid displaying all alerts. This page waits for the full database to finish loading before rendering.

### Columns

| Column | Description |
|--------|-------------|
| Rule | CodeQL rule identifier. |
| Kind | Rule kind (e.g., `problem`, `path-problem`). |
| Repository | Repository full name. |
| Severity | Alert severity level. |
| File Path | Source file path where the alert was found. |

### Search

Free-text search with 800 ms debounce. Searches across rule ID, rule kind, repository name, severity, file path, and message columns.

### Alert Detail

Click any row to open a detail dialog showing:
- Full file location (path, line/column ranges).
- Alert message.
- Code snippets (source, sink, context).
- Code flow step count.
- Result fingerprint.

### Query Parameters

| Parameter | Description | Example |
|-----------|-------------|---------|
| `search` | Pre-fill the search box. | `/alert?search=sql-injection` |

<br>
<br>
<div style="width:70vw; margin-left:calc(50% - 35vw);">
<video src="../videos/alerts.mp4" autoplay loop muted playsinline style="width:100%"></video>
</div>

## Repositories (`/repo`)

A client-side data grid displaying all repositories with alert counts.

### Columns

| Column | Description |
|--------|-------------|
| Name | Repository full name. |
| URL | Link to the repository on GitHub. |
| Status | Analysis status (succeeded, access_mismatch, not_found, no_codeql_db, over_limit). |
| Alerts | Number of alerts found in the repository. |

### Search

Client-side quick filter matching across all visible columns.

### Query Parameters

| Parameter | Description | Example |
|-----------|-------------|---------|
| `hasAlerts` | Filter to repositories with (`true`) or without (`false`) alerts. | `/repo?hasAlerts=true` |
| `status` | Filter by analysis status. | `/repo?status=succeeded` |

<br>
<br>
<div style="width:70vw; margin-left:calc(50% - 35vw);">
<video src="../videos/repositories.mp4" autoplay loop muted playsinline style="width:100%"></video>
</div>

## Rules (`/rule`)

A client-side data grid displaying all rules with alert counts.

### Columns

| Column | Description |
|--------|-------------|
| ID | CodeQL rule identifier. |
| Description | Rule description. |
| Severity | Rule severity level. |
| Kind | Rule kind. |
| Property Tags | Comma-separated rule tags. |
| Alerts | Number of alerts triggered by this rule. |

### Search

Client-side quick filter matching across all fields including property tags.

### Query Parameters

| Parameter | Description | Example |
|-----------|-------------|---------|
| `hasAlerts` | Filter to rules with (`true`) or without (`false`) alerts. | `/rule?hasAlerts=true` |

### Rule Detail

Double-click a row to navigate to the rule detail page with a full breakdown of alerts for that rule.

<div style="width:70vw; margin-left:calc(50% - 35vw);">
<video src="../videos/rules.mp4" autoplay loop muted playsinline style="width:100%"></video>
</div>
