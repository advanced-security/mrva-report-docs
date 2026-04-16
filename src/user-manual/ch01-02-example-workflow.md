# Example GitHub Actions Workflow

The workflow below automates the entire post-analysis pipeline: downloading SARIF artifacts, transforming them into a SQLite database, preparing dashboard metrics, building the Blazor WebAssembly report, and deploying it to GitHub Pages. It is triggered manually via `workflow_dispatch` and requires two inputs: the MRVA analysis ID and the controller repository.

Before using this workflow, ensure GitHub Pages is enabled on the deployment repository (Settings → Pages → Source: GitHub Actions) and a repository secret named `TOKEN` is configured with a PAT or GitHub App token that has read access to the controller and target repositories.

The [Workflow Steps](ch01-02-01-workflow-steps.md) chapter provides a detailed breakdown of each step.

```yaml
name: Deploy to GitHub Pages

on:
  workflow_dispatch:
    inputs:
      analysis-id:
        description: 'MRVA analysis ID'
        required: true
        type: string
      controller-repo:
        description: 'Controller repository (owner/name)'
        required: true
        type: string

permissions:
  contents: read
  pages: write
  id-token: write

concurrency:
  group: "pages"
  cancel-in-progress: false

env:
  DATA_DIR: mrva-reports/src/WebAssembly/wwwroot/data

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout mrva-reports
        uses: actions/checkout@v4
        with:
          repository: ghas-projects/mrva-reports
          path: mrva-reports
          token: ${{ secrets.TOKEN }}

      - name: Set analysis directory
        run: |
          SANITIZED_REPO="${{ inputs.controller-repo }}"
          SANITIZED_REPO="${SANITIZED_REPO//\//-}"
          echo "ANALYSIS_DIR=analyses/${{ inputs.analysis-id }}-${SANITIZED_REPO}" >> "$GITHUB_ENV"

      # ── sarif-sql: download SARIF data and transform to SQLite ───────
      - name: Download sarif-sql CLI
        run: |
          gh release download --repo ghas-projects/sarif-sql \
            --pattern 'sarif-sql-linux-amd64' \
            --output sarif-sql
          chmod +x sarif-sql
        env:
          GH_TOKEN: ${{ secrets.GITHUB_TOKEN }}

      - name: sarif-sql analysis start
        run: |
          ./sarif-sql analysis start \
            --analysis-id ${{ inputs.analysis-id }} \
            --controller-repo ${{ inputs.controller-repo }} \
            --token ${{ secrets.TOKEN }}

      - name: sarif-sql analysis download
        run: |
          ./sarif-sql analysis download \
            --analysis-id ${{ inputs.analysis-id }} \
            --controller-repo ${{ inputs.controller-repo }} \
            --directory ${{ env.ANALYSIS_DIR }} \
            --token ${{ secrets.TOKEN }}

      - name: sarif-sql analysis transform
        run: |
          mkdir -p ${{ env.DATA_DIR }}
          ./sarif-sql transform \
            --analysis-id ${{ inputs.analysis-id }} \
            --controller-repo ${{ inputs.controller-repo }} \
            --sarif-directory ${{ env.ANALYSIS_DIR }} \
            --output ${{ env.DATA_DIR }}

      # ── mrva-prep: index, dashboard & compress ───────────────────────
      - name: Download mrva-prep CLI
        run: |
          gh release download --repo ghas-projects/mrva-prep \
            --pattern 'mrva-prep-linux-amd64' \
            --output mrva-prep
          chmod +x mrva-prep
        env:
          GH_TOKEN: ${{ secrets.GITHUB_TOKEN }}

      - name: mrva-prep index
        run: ./mrva-prep index --db ${{ env.DATA_DIR }}/mrva-analysis.db

      - name: mrva-prep dashboard
        run: ./mrva-prep dashboard --db ${{ env.DATA_DIR }}/mrva-analysis.db --output ${{ env.DATA_DIR }}

      # ── Build & publish Blazor WASM ──────────────────────────────────
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

      - name: Rewrite base href for GitHub Pages
        run: sed -i 's|<base href="/" />|<base href="/mrva-site/" />|g' release/wwwroot/index.html

      - name: Add .nojekyll file
        run: touch release/wwwroot/.nojekyll

      - name: Upload artifact
        uses: actions/upload-pages-artifact@v3
        with:
          path: release/wwwroot

  deploy:
    environment:
      name: github-pages
      url: ${{ steps.deployment.outputs.page_url }}
    runs-on: ubuntu-latest
    needs: build
    steps:
      - name: Deploy to GitHub Pages
        id: deployment
        uses: actions/deploy-pages@v4
```