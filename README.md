# CakePHP 2 Documentation (Piece of Cake 2 fork)

Documentation for [pieceofcake2/cakephp](https://github.com/pieceofcake2/cakephp),
a community-maintained fork of CakePHP 2.x for PHP 8, maintained by the
[Piece of Cake 2](https://github.com/pieceofcake2) project.

This repository is a fork of the `2.x` branch of
[cakephp/docs](https://github.com/cakephp/docs/tree/2.x) (the CakePHP Cookbook).
The book describes CakePHP 2 as it works on the fork today (Composer, PHP 8,
the modern directory layout). Differences from the original book are listed on
[Changes from the Original Book](https://pieceofcake2.github.io/docs/appendices/changes-from-the-original-book.html).

The theme lives in [pieceofcake2/docs-skeleton](https://github.com/pieceofcake2/docs-skeleton)
(a fork of cakephp/docs-skeleton) and is cloned at build time.

## Documentation

- Website: https://pieceofcake2.github.io/docs/ (Japanese: https://pieceofcake2.github.io/docs/ja/)
- Markdown: [docs/en/index.md](docs/en/index.md) (Japanese: [docs/ja/index.md](docs/ja/index.md)) (for AI tools and offline reading)

For AI coding tools, install the Markdown docs of the CakePHP 2 book as a dev dependency: `composer require --dev pieceofcake2/docs` (files in `vendor/pieceofcake2/docs/docs/en/`).

## Using the Docs with AI Tools

The Markdown pages can be installed with Composer, so that AI coding tools and
agents can read them locally. Install them as a development dependency:

```bash
composer require --dev pieceofcake2/docs
```

The pages are then in `vendor/pieceofcake2/docs/`:

- `docs/en/`: the English pages, for example
  `vendor/pieceofcake2/docs/docs/en/getting-started.md`
- `docs/ja/`: the Japanese pages, with the same structure
- `toc_en.json` and `toc_ja.json`: the table of contents of each language

Point your AI tool to that directory, for example with a line in your agent
instructions (`CLAUDE.md`, `AGENTS.md` or `.cursorrules`):

```md
The CakePHP 2 documentation is in vendor/pieceofcake2/docs/docs/en/.
```

## Contributing

The general contribution guide of the organization is
[CONTRIBUTING.md](https://github.com/pieceofcake2/.github/blob/master/CONTRIBUTING.md).
How to edit this book (local development, repository layout, editing and
translation guidelines, formatting guide) is described on the book's
Contributing › Documentation page:
https://pieceofcake2.github.io/docs/contributing/documentation.html
(Markdown: [docs/en/contributing/documentation.md](docs/en/contributing/documentation.md)).

Open pull requests against `2.x`. You can also edit a page online with GitHub:
the "Edit this page on GitHub" link at the bottom of every page opens GitHub's
editor for that page. To preview your changes locally, run
`./bin/dev-server.sh`.

## Origin

This package is a maintained fork of [cakephp/docs](https://github.com/cakephp/docs) (not published
on Packagist, last upstream commit [`6b9d755`](https://github.com/cakephp/docs/commit/6b9d755) on
`2.x`), originally by the CakePHP community (Cake Software Foundation, Inc.). It is maintained for
CakePHP 2 on modern PHP by [pieceofcake2](https://github.com/pieceofcake2). Changes from the
original are listed in the
[commit history since the fork](https://github.com/pieceofcake2/docs/compare/6b9d755...2.x).

## License

See [LICENSE](LICENSE).

- Documentation content: [CC BY-NC-SA 4.0](https://creativecommons.org/licenses/by-nc-sa/4.0/),
  © Cake Software Foundation, Inc., with modifications © 2025-present Piece of Cake 2.
- Theme: MIT, [pieceofcake2/docs-skeleton](https://github.com/pieceofcake2/docs-skeleton)
  (fork of [cakephp/docs-skeleton](https://github.com/cakephp/docs-skeleton)).

This project is not affiliated with or endorsed by the Cake Software Foundation.
CakePHP is a trademark of the Cake Software Foundation.
