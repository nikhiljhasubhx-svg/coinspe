<<<<<<< HEAD
# coinspe.github.io

Coinspe’s public API documentation lives here. The site is built with **MkDocs + Material**, deployed to GitHub Pages, and mirrors the Coinspe PG v1.0.0.2 spec (JWT + HMAC auth, payment links, market prices, and webhook flows).

## Stack

| Layer | Tech |
| --- | --- |
| Static site generator | MkDocs 1.6 + Material theme |
| Language | Markdown + YAML configuration |
| Hosting | GitHub Pages (`gh-pages` branch) via Actions |

## Local development

```bash
python3 -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt
mkdocs serve
```

Open http://127.0.0.1:8000 to live-preview the docs while editing Markdown files inside `docs/`.

## Project layout

```
docs/
  index.md                      # Landing page
  getting-started/              # Quickstart + auth details
  api-reference/                # Endpoint specs + sample code
  guides/errors.md              # Error catalog
  resources/                    # Changelog + support
.github/workflows/deploy.yml    # CI build + gh-pages deploy
mkdocs.yml                      # Site navigation + theme config
```

## Deploying

1. Commit your changes to `main`.
2. GitHub Actions (workflow `Deploy Docs`) installs dependencies, runs `mkdocs build --strict`, and publishes `site/` to the `gh-pages` branch.
3. GitHub Pages serves `https://coinspe.github.io`.

> Run `mkdocs gh-deploy --force` locally only if you need to push a hotfix without waiting for CI (rare).

## Maintenance tips

- Update `requirements.txt` when bumping MkDocs or Material.
- Keep the changelog (`docs/resources/changelog.md`) in sync with production releases.
- Rotate any credentials used in examples; never commit live tokens or secrets.

## Prerequisites

- Python 3.8+
- `pip`

## Getting started

```bash
python3 -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt
mkdocs serve
```

## Deployment to GitHub Pages

1. Commit the docs to the `coinspe.github.io` repository.
2. Enable GitHub Pages (Source: GitHub Actions).
3. Use the included workflow or run `mkdocs gh-deploy --force` to push the `gh-pages` branch.
=======
# coinspe.github.io
>>>>>>> 9d64e69c08cd25741966b3c0563c8983d91d917e
