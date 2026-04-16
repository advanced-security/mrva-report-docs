# Target Repository List

Prepare a JSON file listing the repositories to analyze. The file should contain an array of `"owner/name"` strings:

```json
[
  "org/repo-one",
  "org/repo-two",
  "org/repo-three"
]
```

Save this as `repos.json` (or any filename, you will pass it as an argument). Alternatively, you can target entire organizations using the `repository_owners` field in the API request instead of listing individual repositories.
