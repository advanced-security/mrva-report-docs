# Create MRVA Analysis

In this workflow, the main mechanism for creating a MRVA analysis is via the [Create a CodeQL variant analysis](https://docs.github.com/en/rest/code-scanning/code-scanning?apiVersion=2026-03-10#create-a-codeql-variant-analysis) API. This will produce the raw data.

## Prerequisites

### CodeQL Database Availability

MRVA queries run against CodeQL databases stored on GitHub. A repository cannot participate in variant analysis until a valid CodeQL database exists for the target language. Each code scanning workflow run generates a fresh database snapshot per analyzed language, with the most recent version persisted automatically. Large-scale MRVA is only feasible after CodeQL has been fully rolled out across the target repositories for the language under analysis.

### Controller Repository

MRVA uses GitHub Actions to execute queries. No dedicated workflow file is required, but a controller repository must exist to orchestrate the analysis. The controller repository must contain at least one commit.

### Runner Configuration

Self-hosted runners are supported via the `MRVA_RUNNER_OS` action variable (not a secret) on the controller repository. The value is a JSON array mapping to the `runs-on` field:

```json
["self-hosted", "some-label", "another-label"]
```

## Query Packs

A CodeQL query pack is a distributable unit that bundles queries, query suites, optional libraries, and a manifest (`codeql-pack.yml`). Packs are versioned, portable, and define dependencies on other packs. The `query_pack` field in the API request specifies which queries MRVA executes. You can read more about creating CodeQL packs [here](https://docs.github.com/en/code-security/tutorials/customize-code-scanning/creating-and-working-with-codeql-packs).

## API

### Request Body

| Field                | Type               | Required | Description |
|---------------------|--------------------|----------|-------------|
| `language`          | `string`           | Yes      | Target language (`javascript`, `typescript`, `python`, `java`, `cpp`, `csharp`, `go`, `ruby`). Must match the CodeQL database language. One database exists per language per repository. |
| `query_pack`        | `string` (Base64)  | Yes      | Base64-encoded CodeQL query pack tarball (`.tgz`). |
| `repositories`      | `string[]`         | One of   | Explicit repository list in `owner/name` format. Mutually exclusive with `repository_owners`. |
| `repository_owners` | `string[]`         | One of   | Organization handles. Targets all eligible repositories under each owner. Mutually exclusive with `repositories`. |

### Bundle, Encode, and Submit

The following example script bundles a CodeQL query pack, Base64-encodes it, constructs the JSON payload, and submits the variant analysis request. The payload is written to a temporary file because the encoded query pack exceeds command-line argument limits. `repos.json` contains a JSON array of `"owner/name"` strings. `GITHUB_TOKEN` must be set as an environment variable. There are multiple ways to achieve this, the purpose of this example is to demonstrate some of the considerations that should be made when calling the create endpoint, particularly when exceeding length limits.

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

if ! command -v codeql &>/dev/null; then
  echo "Error: 'codeql' CLI not found in PATH." >&2
  exit 1
fi

# Bundle query pack
echo "==> Installing pack dependencies..."
codeql pack install "$PACK_DIR"

echo "==> Bundling CodeQL pack..."
codeql pack bundle \
  --output="$BUNDLE_OUTPUT" \
  "$PACK_DIR"

echo "==> Base64-encoding bundle..."
base64 -w 0 "$BUNDLE_OUTPUT" > "$ENCODED_OUTPUT"

BYTE_SIZE=$(wc -c < "$BUNDLE_OUTPUT")
ENCODED_SIZE=$(wc -c < "$ENCODED_OUTPUT")
echo "  Bundle:  $BUNDLE_OUTPUT ($BYTE_SIZE bytes)"
echo "  Base64:  $ENCODED_OUTPUT ($ENCODED_SIZE bytes)"

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

The following demonstrates the full workflow: executing the scripts above with the required flags, bundling and encoding the query pack, and submitting the variant analysis request to the GitHub API. Note the `id` field in the API response as it is required in subsequent steps of this workflow. See the [official documentation](https://docs.github.com/en/rest/code-scanning/code-scanning?apiVersion=2026-03-10#create-a-codeql-variant-analysis) for full response details.


{{ #asciinema ../cast/create-mrva-analysis.cast }}
