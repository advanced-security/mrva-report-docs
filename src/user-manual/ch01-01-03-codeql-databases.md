# CodeQL Database Availability

MRVA queries run against CodeQL databases stored on GitHub. A repository cannot participate in variant analysis until a valid CodeQL database exists for the target language. Each code scanning workflow run generates a fresh database snapshot per analyzed language, with the most recent version persisted automatically.

Large-scale MRVA is only feasible after CodeQL has been fully rolled out across the target repositories for the language under analysis.
