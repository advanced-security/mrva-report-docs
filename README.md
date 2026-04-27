# MRVA Report Documentation

Documentation for the Multi-Repository Variant Analysis (MRVA) reporting pipeline. This is a system that automates downloading, transforming, and visualizing [CodeQL MRVA](https://docs.github.com/en/code-security/concepts/code-scanning/multi-repository-variant-analysis) results into a single interactive report.

A live example is available at [ghas-projects.github.io/mrva-deploy](https://ghas-projects.github.io/mrva-deploy/).

## Reading the documentation

The published book is available at [advanced-security.github.io/mrva-report-docs](https://advanced-security.github.io/mrva-report-docs/).

## Building locally

### Prerequisites

1. Install [Rust](https://www.rust-lang.org/tools/install)
2. Install [mdbook](https://rust-lang.github.io/mdBook/guide/installation.html): `cargo install mdbook`
3. Install [mdbook-mermaid](https://github.com/badboy/mdbook-mermaid): `cargo install mdbook-mermaid`
4. Install [mdbook-asciinema](https://github.com/fluentci-io/mdbook-asciinema): `cargo install mdbook-asciinema`

### Preview

```sh
mdbook serve --open
```

### Build

```sh
mdbook build
```

Output is written to the `book/` directory.

## Contributing

See [CONTRIBUTING.md](CONTRIBUTING.md).

## License

This project is licensed under the [MIT License](LICENSE).
