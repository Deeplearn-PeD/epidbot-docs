# epidbot-docs

Documentation site for [EpiDBot](https://github.com/Deeplearn-PeD/EpiDBot) — an
AI-powered assistant for analyzing Brazilian public health data (DATASUS).

**Browse the docs:** https://deeplearn-ped.github.io/epidbot-docs/

Trilingual (English / Português / Español), built with
[MkDocs Material](https://squidfunk.github.io/mkdocs-material/) and deployed to
GitHub Pages automatically on every push to `main`.

## Editing

All sources live under `docs/`. Page structure and navigation are defined in
`mkdocs.yml`. Translations live in `docs/pt/` and `docs/es/`.

## Local preview

```bash
pip install -r requirements-docs.txt
mkdocs serve
```

## PDF export

The trilingual PDF takes a long time to build, so it is disabled by default:

```bash
ENABLE_PDF_EXPORT=1 mkdocs build   # outputs site/EpidBot_Documentation.pdf
```

Or trigger the **Build PDF** workflow from the Actions tab (artifact download).

## Deployment

`mkdocs gh-deploy --force` runs via GitHub Actions (`.github/workflows/deploy-docs.yml`)
and publishes the built site to the `gh-pages` branch.
