## Contributing

[fork]: https://github.com/advanced-security/sarif-sql/fork
[pr]: https://github.com/advanced-security/sarif-sql/compare
[style]: https://github.com/advanced-security/sarif-sql/blob/main/.golangci.yaml

Hi there! We're thrilled that you'd like to contribute to this project. Your help is essential for keeping it great.

Contributions to this project are [released](https://help.github.com/articles/github-terms-of-service/#6-contributions-under-repository-license) to the public under the [project's open source license](LICENSE.txt).

Please note that this project is released with a [Contributor Code of Conduct](CODE_OF_CONDUCT.md). By participating in this project you agree to abide by its terms.

## Prerequisites for running code

These are one time installations required to be able to test your changes locally as part of the pull request (PR) submission process.

1. install [Rust](https://www.rust-lang.org/tools/install) (required by `mdbook`)
1. install [mdbook](https://rust-lang.github.io/mdBook/guide/installation.html): `cargo install mdbook`
1. install [mdbook-mermaid](https://github.com/badboy/mdbook-mermaid): `cargo install mdbook-mermaid`
1. install [mdbook-asciinema](https://github.com/fluentci-io/mdbook-asciinema): `cargo install mdbook-asciinema`

To preview the book locally:

```sh
mdbook serve --open
```

## Submitting a pull request

1. [Fork][fork] and clone the repository
1. Create a new branch: `git checkout -b my-branch-name`
1. Make your change.
1. Push to your fork and [submit a pull request][pr]
1. Make sure all checks on the PR pass.
1. Pat yourself on the back and wait for your pull request to be reviewed and merged.

Here are a few things you can do that will increase the likelihood of your pull request being accepted:

- Follow the [style guide][style].
- Keep your change as focused as possible. If there are multiple changes you would like to make that are not dependent upon each other, consider submitting them as separate pull requests.
- Write a [good commit message](http://tbaggery.com/2008/04/19/a-note-about-git-commit-messages.html).

## Resources

- [How to Contribute to Open Source](https://opensource.guide/how-to-contribute/)
- [Using Pull Requests](https://help.github.com/articles/about-pull-requests/)
- [GitHub Help](https://help.github.com)