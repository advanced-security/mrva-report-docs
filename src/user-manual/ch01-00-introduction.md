# User Manual

This manual covers the end-to-end workflow for running a Multi-Repository Variant Analysis (MRVA), transforming the results into a queryable database, and deploying an interactive report.

## MRVA Workflow

1. **Create an MRVA analysis** - submit a CodeQL query pack to the GitHub API, targeting up to 1,000 repositories in a single operation.
2. **Download and transform** - use the `sarif-sql` CLI to retrieve SARIF artifacts and normalize them into a SQLite database.
3. **Prepare the report** - use the `mrva-prep` CLI to add indexes, pre-aggregate dashboard metrics, and compress the database.
4. **Deploy** - publish an interactive Blazor WebAssembly dashboard to GitHub Pages via a GitHub Actions workflow.
5. **Navigate the report** - explore alerts, repositories, rules, and severity breakdowns in the browser.

Each step can be run independently, but the typical flow proceeds in order. A fully automated CI/CD workflow is also provided that chains steps 2–4 into a single dispatch.

## Workflow Overview

```mermaid
flowchart TD
    A["<b>Create CodeQL Variant Analysis</b><br/>GitHub API"] --> B["Executes query against<br/>up to 1,000 repositories"]

    B --> C["<b>sarif-sql CLI</b>"]
    C --> C1["analysis start → Create workspace directory"]
    C --> C2["analysis summary → Check analysis status"]
    C --> C3["analysis download → analysis.json + repos.json + *.sarif"]
    C3 --> C4["transform → mrva-analysis.db (SQLite)"]

    C4 --> D["<b>mrva-prep CLI</b>"]
    D --> D1["index → Query-optimized indexes"]
    D1 --> D2["dashboard → Pre-aggregated metrics (dashboard.json)"]
    D --> D3["compress → Gzip-compressed database (local dev only)"]

    D2 --> E["<b>Deploy</b><br/>GitHub Actions"]
    E --> E1["dotnet publish + GitHub Pages"]

    E1 --> F["<b>Interactive Report</b><br/>Browser"]
    F --> F1["Dashboard → KPI cards, charts, top-10 tables"]
    F --> F2["Alerts → Paginated, searchable alert grid"]
    F --> F3["Repositories → Repository list with alert counts"]
    F --> F4["Rules → Rule list with alert breakdowns"]

    style A fill:#2d333b,stroke:#444,color:#adbac7
    style C fill:#2d333b,stroke:#444,color:#adbac7
    style D fill:#2d333b,stroke:#444,color:#adbac7
    style E fill:#2d333b,stroke:#444,color:#adbac7
    style F fill:#2d333b,stroke:#444,color:#adbac7
```
