# Introduction

This section documents the architecture, design decisions, and internals of each component in the MRVA pipeline. It is intended for developers who want to understand, modify, or extend the system. If you only need to run the tools and generate reports, see the [User Manual](../user-manual/ch01-00-introduction.html).

## Chapters

### [Methodology](ch02-01-methodology.md)

Describes the four-stage data pipeline: **Raw** → **Curated** → **Unified** → **Optimized**. Each stage defines the shape and purpose of the data as it moves from SARIF JSON to a query-ready SQLite database.

### [Implementation](ch02-03-implementation.md)

Documents the four components that realize the pipeline:

- [**Create MRVA Analysis**](ch02-04-create-mrva-analysis.md) - Submitting a CodeQL variant analysis via the GitHub Code Scanning REST API. Covers controller repository requirements, runner configuration, query pack bundling, and the API request/response schema.
- [**sarif-sql**](ch02-05-sarif-sql.md) - A Go CLI that downloads SARIF artifacts and transforms them into a normalized SQLite database. Documents commands, global flags, authentication modes, and the database schema.
- [**mrva-prep**](ch02-06-mrva-prep.md) - A Go CLI that adds query-optimized indexes, extracts pre-aggregated dashboard metrics to `dashboard.json`, and gzip-compresses the database. Documents commands, flags, and aggregation queries.
- [**mrva-reports**](ch02-07-mrva-report.md) - A Blazor WebAssembly application that renders the SQLite database as a static single-page dashboard in the browser. Covers the solution structure, technology stack, two-phase loading architecture, JavaScript interop, and SPA routing on GitHub Pages.
