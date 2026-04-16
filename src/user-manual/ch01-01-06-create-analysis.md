# Creating an MRVA Analysis

A MRVA analysis is created by submitting a CodeQL query pack to the [Create a CodeQL variant analysis](https://docs.github.com/en/rest/code-scanning/code-scanning?apiVersion=2026-03-10#create-a-codeql-variant-analysis) API endpoint. This triggers a GitHub Actions workflow that executes the queries against the target repositories.

## Step 1: Prepare the Query Pack

A CodeQL query pack bundles queries, query suites, optional libraries, and a `codeql-pack.yml` manifest. Install dependencies and create a distributable bundle:

```bash
codeql pack install path/to/query-pack
codeql pack bundle --output query-pack-bundle.tar.gz path/to/query-pack
```

You can read more about creating query packs in the [GitHub documentation](https://docs.github.com/en/code-security/tutorials/customize-code-scanning/creating-and-working-with-codeql-packs).

## Step 2: Base64-Encode the Bundle

The API expects the query pack as a Base64-encoded string:

```bash
base64 -w 0 query-pack-bundle.tar.gz > query-pack-base64.txt
```

## Step 3: Construct the Request Payload

The payload requires the target language, the encoded query pack, and a repository target:

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `language` | `string` | Yes | Target language: `javascript`, `typescript`, `python`, `java`, `cpp`, `csharp`, `go`, `ruby`. |
| `query_pack` | `string` (Base64) | Yes | Base64-encoded `.tgz` query pack. |
| `repositories` | `string[]` | One of | Explicit list of repositories in `owner/name` format. |
| `repository_owners` | `string[]` | One of | Organization handles. Targets all eligible repositories under each owner. |

`repositories` and `repository_owners` are mutually exclusive — use one or the other.

Build the JSON payload with `jq`:

```bash
jq -n \
  --arg language "go" \
  --rawfile query_pack query-pack-base64.txt \
  --slurpfile repositories repos.json \
  '{language: $language, query_pack: $query_pack, repositories: $repositories[0]}' \
  > payload.json
```

The payload is written to a file because the encoded query pack typically exceeds command-line argument limits.

## Step 4: Submit the Request

```bash
curl -L \
  -X POST \
  -H "Accept: application/vnd.github+json" \
  -H "Authorization: Bearer ${GITHUB_TOKEN}" \
  -H "X-GitHub-Api-Version: 2026-03-10" \
  -H "Content-Type: application/json" \
  -d @payload.json \
  "https://api.github.com/repos/${CONTROLLER_REPO}/code-scanning/codeql/variant-analyses"
```

Replace `${CONTROLLER_REPO}` with your controller repository in `owner/name` format.

## Step 5: Note the Analysis ID

The API response includes an `id` field. This is the `analysisId` required by all subsequent steps:

```json
{
  "id": 22021,
  "controller_repo": { ... },
  "query_language": "go",
  "status": "in_progress",
  ...
}
```

Record this value - you will pass it as `--analysis-id` to the `sarif-sql` and `mrva-prep` commands.

## Complete Script

The following script combines all steps into a single executable:

```bash
#!/usr/bin/env bash
set -euo pipefail

usage() {
  echo "Usage: $0 -c <controller_repo> -l <language> -p <pack_dir> -r <repos_file>" >&2
  echo "  -c  Controller repository (owner/name)" >&2
  echo "  -l  CodeQL language" >&2
  echo "  -p  Query pack source directory" >&2
  echo "  -r  Path to repos.json" >&2
  exit 1
}

while getopts "c:l:p:r:" opt; do
  case "$opt" in
    c) CONTROLLER_REPO="$OPTARG" ;;
    l) LANGUAGE="$OPTARG" ;;
    p) PACK_DIR="$OPTARG" ;;
    r) REPOS_FILE="$OPTARG" ;;
    *) usage ;;
  esac
done

: "${CONTROLLER_REPO:?Missing -c <controller_repo>}"
: "${LANGUAGE:?Missing -l <language>}"
: "${PACK_DIR:?Missing -p <pack_dir>}"
: "${REPOS_FILE:?Missing -r <repos_file>}"

BUNDLE_OUTPUT="$(basename "$PACK_DIR")-bundle.tar.gz"
ENCODED_OUTPUT="$(basename "$PACK_DIR")-base64.txt"

SCRIPT_DIR="$(cd "$(dirname "${BASH_SOURCE[0]}")" && pwd)"
cd "$SCRIPT_DIR"

# Bundle query pack
echo "==> Installing pack dependencies..."
codeql pack install "$PACK_DIR"

echo "==> Bundling CodeQL pack..."
codeql pack bundle --output="$BUNDLE_OUTPUT" "$PACK_DIR"

echo "==> Base64-encoding bundle..."
base64 -w 0 "$BUNDLE_OUTPUT" > "$ENCODED_OUTPUT"

# Submit variant analysis
PAYLOAD=$(mktemp)
trap 'rm -f "$PAYLOAD"' EXIT

jq -n \
  --arg language "$LANGUAGE" \
  --rawfile query_pack "$SCRIPT_DIR/$ENCODED_OUTPUT" \
  --slurpfile repositories "$SCRIPT_DIR/$REPOS_FILE" \
  '{language: $language, query_pack: $query_pack, repositories: $repositories[0]}' \
  > "$PAYLOAD"

echo "==> Submitting variant analysis..."
curl -L \
  -X POST \
  -H "Accept: application/vnd.github+json" \
  -H "Authorization: Bearer ${GITHUB_TOKEN}" \
  -H "X-GitHub-Api-Version: 2026-03-10" \
  -H "Content-Type: application/json" \
  -d @"$PAYLOAD" \
  "https://api.github.com/repos/${CONTROLLER_REPO}/code-scanning/codeql/variant-analyses"
```

**Usage:**

```bash
export GITHUB_TOKEN="ghp_..."

./create-analysis.sh \
  -c "org/controller" \
  -l "go" \
  -p "./my-query-pack" \
  -r "./repos.json"
```
