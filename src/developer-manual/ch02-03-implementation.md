# Implementation 

Four components implement the data pipeline described in the [Methodology](ch02-01-methodology.md) chapter. Each is a standalone tool with its own repository, build, and release lifecycle. The GitHub Actions workflow that chains them together is documented in the [User Manual](../user-manual/ch01-02-example-workflow.md).

| Component | Language | Pipeline Stage | Description |
|-----------|----------|----------------|-------------|
| [Create MRVA Analysis](ch02-04-create-mrva-analysis.md) | Bash / API | Raw | Submits a CodeQL variant analysis via the GitHub Code Scanning REST API. |
| [sarif-sql](ch02-05-sarif-sql.md) | Go | Curated → Unified | Downloads SARIF artifacts and transforms them into a normalized SQLite database. |
| [mrva-prep](ch02-06-mrva-prep.md) | Go | Optimized | Adds indexes, pre-aggregates dashboard metrics, and compresses the database. |
| [mrva-reports](ch02-07-mrva-report.md) | C# / Blazor WASM | Presentation | Renders the SQLite database as a static single-page dashboard in the browser. |