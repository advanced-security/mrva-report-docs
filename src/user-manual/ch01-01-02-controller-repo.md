# Controller Repository

MRVA uses GitHub Actions to execute queries. A controller repository must exist to orchestrate the analysis. Requirements:

- The repository must contain at least one commit.
- No dedicated workflow file is required, GitHub creates one automatically.

## Self-Hosted Runners (Optional)

Self-hosted runners are supported via the `MRVA_RUNNER_OS` action variable (not a secret) on the controller repository. The value is a JSON array mapping to the `runs-on` field:

```json
["self-hosted", "some-label", "another-label"]
```

If this variable is not set, the analysis runs on GitHub-hosted runners.
