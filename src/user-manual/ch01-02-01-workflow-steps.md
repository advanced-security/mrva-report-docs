# Workflow Steps

This section breaks down each step of the GitHub Actions workflow from the [Example GitHub Actions Workflow](ch01-02-example-workflow.md). Understanding what each step does helps with troubleshooting, customization, and running the pipeline manually.

## Step 1: Checkout `mrva-reports`

```yaml
- name: Checkout mrva-reports
  uses: actions/checkout@v4
  with:
    repository: ghas-projects/mrva-reports
    path: mrva-reports
    token: ${{ secrets.TOKEN }}
```

Clones the Blazor WebAssembly reporting application into the `mrva-reports/` directory. The `TOKEN` secret must have read access to this repository.

## Step 2: Set Analysis Directory

```yaml
- name: Set analysis directory
  run: |
    SANITIZED_REPO="${{ inputs.controller-repo }}"
    SANITIZED_REPO="${SANITIZED_REPO//\//-}"
    echo "ANALYSIS_DIR=analyses/${{ inputs.analysis-id }}-${SANITIZED_REPO}" >> "$GITHUB_ENV"
```

Constructs a sanitized directory name from the analysis ID and controller repo (replacing `/` with `-`). This path is used by subsequent steps to store downloaded artifacts.

## Step 3: Download and Run `sarif-sql`

```yaml
- name: Download sarif-sql CLI
  run: |
    gh release download --repo ghas-projects/sarif-sql \
      --pattern 'sarif-sql-linux-amd64' \
      --output sarif-sql
    chmod +x sarif-sql
```

Downloads the `sarif-sql` binary from the latest release. Then three commands run in sequence:

### `analysis start`

Creates the local workspace directory structure at `./analyses/{id}-{controller}/`.

### `analysis download`

Authenticates with the GitHub API, fetches the analysis summary, and concurrently downloads SARIF artifacts for all repositories with `analysis_status: succeeded`. Outputs `analysis.json`, `repos.json`, and individual `.sarif` files.

### `transform`

Parses all SARIF files and writes the normalized data to `mrva-analysis.db` in the Blazor app's `wwwroot/data/` directory.

## Step 4: Download and Run `mrva-prep`

```yaml
- name: Download mrva-prep CLI
  run: |
    gh release download --repo ghas-projects/mrva-prep \
      --pattern 'mrva-prep-linux-amd64' \
      --output mrva-prep
    chmod +x mrva-prep
```

Downloads the `mrva-prep` binary from the latest release. Then two commands run:

### `index`

Creates query-optimized indexes on the `alert` table (`rule_row_id`, `repository_row_id`), runs `ANALYZE` to update planner statistics, and `VACUUM` to compact the database.

### `dashboard`

Pre-aggregates dashboard metrics (scalar counts, severity distribution, top 10 rules/repos/file paths) into `dashboard.json`. This file enables the report's instant first-paint while the full database loads in the background.

## Step 5: Build Blazor WASM Application

```yaml
- name: Setup .NET
  uses: actions/setup-dotnet@v5
  with:
    dotnet-version: '10.0.x'

- name: Install wasm-tools workload
  run: dotnet workload install wasm-tools

- name: Restore dependencies
  run: dotnet restore mrva-reports/src/WebAssembly/WebAssembly.csproj

- name: Publish
  run: dotnet publish mrva-reports/src/WebAssembly/WebAssembly.csproj -c Release -o release --nologo
```

Installs .NET 10.0, the WebAssembly AOT compilation workload, restores NuGet packages, and publishes the Blazor WASM app with AOT compilation. A custom MSBuild target (`CompressDatabase`) automatically gzip-compresses the SQLite database at level 9 and removes the uncompressed original.

## Step 6: Configure for GitHub Pages

```yaml
- name: Rewrite base href for GitHub Pages
  run: sed -i 's|<base href="/" />|<base href="/mrva-site/" />|g' release/wwwroot/index.html

- name: Add .nojekyll file
  run: touch release/wwwroot/.nojekyll
```

Rewrites the `<base href>` tag to match the GitHub Pages path prefix. The `.nojekyll` file prevents GitHub Pages from processing files through Jekyll, which would break the Blazor app's static assets.

## Step 7: Deploy

```yaml
- name: Upload artifact
  uses: actions/upload-pages-artifact@v3
  with:
    path: release/wwwroot

- name: Deploy to GitHub Pages
  uses: actions/deploy-pages@v4
```

Uploads the published static site as a Pages artifact and deploys it. The report is available at `https://<owner>.github.io/<repo>/` once deployment completes.
