---
name: typst-project-scaffold
description: Impose a standard shape on a Typst project - a Makefile, typstyle, cspell, vendored fonts, and GitHub CI that builds the PDF and attaches it to tagged releases. Use when starting a Typst document or slide deck, when adding formatting, spell-check or CI to an existing one, or when asked to scaffold or standardize a Typst repo.
---

# typst-project-scaffold

Every file below is the whole thing, not a starting point. Add a config key when a run fails without it, never in advance.

## Steps

1. Read the repo first. An existing deck already has a shape; the tooling fits it, not the other way round.
2. Get the current versions from the registries - `curl -s https://api.github.com/repos/typst/typst/releases/latest`, the same for `typstyle-rs/typstyle`, `npm view cspell version`, and the GitHub Marketplace pins for the actions. The versions written below are stale by the time you read them.
3. Format soft-wrapped: `--line-width 100000 --wrap-text=fill`. Typst joins source lines into a paragraph anyway, so a hard wrap only decides where diffs land, and a one-word edit reflows a paragraph. Run it once over the existing file and check the PDF text is unchanged (`pdftotext` both and diff) before keeping the result.
4. Write the files, then run `make check` and fix what it reports.

## Layout

- One `.typ` at the root named for the document. No `src/`, no per-slide files until the deck is genuinely unmanageable in one buffer.
- `assets/` for images and generated plots. Generated charts are committed - the generator usually lives in another repo, and CI must not need it.
- `.fonts/` for every font the document names, vendored. Without it a CI build silently substitutes and the PDF differs from what was on screen.
- The built PDF is gitignored. It is a release artifact, not a source file.

## Makefile

The single entry point. CI runs the same targets, so there is one definition of "correct".

```make
# Fonts are vendored under .fonts/, so a local build matches CI byte for byte.
FONTS  := --font-path .fonts
# Soft wrap: a width nothing reaches, plus fill to join prose that arrived wrapped.
TYPSTYLE := typstyle --line-width 100000 --wrap-text=fill
CSPELL := npx -y cspell@10 lint --no-progress --dot
ASSETS := $(shell find assets -type f)

<doc>.pdf: <doc>.typ $(ASSETS)
	typst compile $(FONTS) <doc>.typ

watch:
	typst watch $(FONTS) <doc>.typ

fmt:
	$(TYPSTYLE) -i <doc>.typ

check: <doc>.pdf
	$(TYPSTYLE) --check <doc>.typ
	$(CSPELL) .

.PHONY: watch fmt check
```

The PDF target is first, so bare `make` builds and is incremental. `check` depends on it, because a deck that does not compile is the only failure that matters.

## cspell.json

```jsonc
{
  "$schema": "https://raw.githubusercontent.com/streetsidesoftware/cspell/main/cspell.schema.json",
  "version": "0.2",
  // Both dictionaries, when British prose carries American technical terms.
  "language": "en,en-GB",
  "ignorePaths": [".fonts/**", "assets/**", "*.pdf", "LICENSE", ".git/**"],
  "ignoreRegExpList": ["Urls", "HexValues"],
  "words": [],
}
```

Fill `words` from the run, not from imagination: `npx -y cspell@10 lint --dot --words-only --unique .` prints exactly what to paste, lowercased and sorted. `Urls` and `HexValues` come first - without them the list fills up with URL slugs and palette hex, and stops being a vocabulary anyone reads. cspell has no Typst parser, so it reads the file as text and flags identifiers; that is the cost of covering prose, and the word list absorbs it.

Do not reach for a Typst linter. `typst compile` is the linter: an undefined function or a bad argument is a compile error, and there is no third-party rule set worth the pin.

## .github/workflows/ci.yml

```yaml
name: CI

on:
  push:
  pull_request:

jobs:
  check:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v7.0.1
      - uses: typst-community/setup-typst@v5.2.0
        with:
          typst-version: "<ver>"
      # typstyle's npm package lags the releases, so take the release binary.
      - name: Install typstyle
        run: |
          sudo curl -fsSL -o /usr/local/bin/typstyle \
            https://github.com/typstyle-rs/typstyle/releases/download/v<ver>/typstyle-x86_64-unknown-linux-musl
          sudo chmod +x /usr/local/bin/typstyle
      - run: make check
```

## .github/workflows/release.yml

A tag builds the PDF and attaches it, so a link to the deck is a link to a release, not to a file someone rebuilt by hand.

```yaml
name: release

on:
  push:
    tags: ["v*"]

permissions:
  contents: write

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v7.0.1
      - uses: typst-community/setup-typst@v5.2.0
        with:
          typst-version: "<ver>"
      - run: typst compile --font-path .fonts <doc>.typ <doc>.pdf
      - uses: softprops/action-gh-release@v3.0.2
        with:
          files: <doc>.pdf
```

## Skipped on purpose

- **pre-commit.** Two tools and one Makefile do not need a hook framework. Add it when the tool list grows past what `make check` can hold in one screen.
- **A typstyle config file.** 0.15 ignores `typstyle.toml`; the flags live in the Makefile, where CI reads them too.
- **markdownlint.** Worth it once the repo carries real prose in `.md`. A README does not qualify. Markdown gets soft-wrapped by hand for the same reason as the deck.

## Typst friction

- `typst watch` reloads on save, but a package version bump needs a restart.
- Package imports pin a version (`@preview/touying:0.6.1`). A minor bump can move layout, so bump one package at a time and look at the PDF.
- Fonts resolve from `--font-path` plus the system set. If a font is installed locally but not vendored, CI is the first place you learn.
