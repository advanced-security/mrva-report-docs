# Authentication

A GitHub Personal Access Token (PAT) or GitHub App credentials are required. The token must have **read access** to:

- The controller repository used to orchestrate the analysis.
- All target repositories included in the analysis.

## Personal Access Token

Pass the token via the `--token` flag:

```bash
sarif-sql analysis download \
  --analysis-id 22021 \
  --controller-repo org/controller \
  --directory ./analyses/22021-org-controller \
  --token "$GITHUB_TOKEN"
```

## GitHub App

For automated or organizational workflows, a GitHub App provides fine-grained, scoped access without personal tokens. Pass the app ID and private key:

```bash
sarif-sql analysis download \
  --analysis-id 22021 \
  --controller-repo org/controller \
  --directory ./analyses/22021-org-controller \
  --app-id "12345" \
  --private-key "$(cat private-key.pem)"
```

The CLI handles the full authentication flow automatically:

1. Generates a JWT from the app ID and private key.
2. Discovers the installation ID for the controller repository.
3. Obtains an installation access token.
4. Refreshes the token transparently via a custom HTTP transport when it expires.

## GitHub Enterprise Server

For GitHub Enterprise Server (GHES) deployments, override the API base URL with the `--base-url` flag:

```bash
sarif-sql analysis download \
  --analysis-id 22021 \
  --controller-repo org/controller \
  --directory ./analyses/22021-org-controller \
  --token "$GITHUB_TOKEN" \
  --base-url "https://github.example.com/api/v3"
```

The default is `https://api.github.com`. This flag applies to all `analysis` subcommands.
