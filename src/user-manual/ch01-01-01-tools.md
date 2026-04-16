# Tools

| Tool | Version | Purpose |
|------|---------|---------|
| [CodeQL CLI](https://github.com/github/codeql-cli-binaries/releases) | Latest | Bundle query packs for the create analysis API call. |
| [sarif-sql](https://github.com/ghas-projects/sarif-sql/releases) | Latest | Download SARIF artifacts and transform them to SQLite. |
| [mrva-prep](https://github.com/ghas-projects/mrva-prep/releases) | Latest | Index, pre-aggregate, and compress the SQLite database. |
| [jq](https://jqlang.github.io/jq/) | 1.6+ | Construct the JSON payload for the create analysis API call. |
| [curl](https://curl.se/) | 7.x+ | Submit the API request. |
| [.NET SDK](https://dotnet.microsoft.com/download) | 10.0 | Build the Blazor WebAssembly report (local development only). |

`sarif-sql` and `mrva-prep` are distributed as standalone binaries, so no Go toolchain is required.
