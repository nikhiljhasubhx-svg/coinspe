# Coinspe API Docs

MkDocs + Material site for `coinspe.github.io`.

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
