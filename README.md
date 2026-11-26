# epidbot-docs

[![DOI](https://zenodo.org/badge/DOI/10.5281/zenodo.24560475.svg)](https://doi.org/10.5281/zenodo.24560475)

> 📚 **Cite this documentation via its DOI:** [https://doi.org/10.5281/zenodo.24560475](https://doi.org/10.5281/zenodo.24560475)
> This DOI identifies the current release (v2.9.0); [Zenodo](https://zenodo.org) mints a fresh DOI for every tagged release.

Documentation site for [EpiDBot](https://github.com/Deeplearn-PeD/EpiDBot) — an
AI-powered assistant for analyzing Brazilian public health data (DATASUS).

**Browse the docs:** https://deeplearn.ltd/epidbot-docs/

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
