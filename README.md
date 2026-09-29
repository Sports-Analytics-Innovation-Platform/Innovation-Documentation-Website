# Docs site

Documentation website for the NBA Analytics & Optimisation Engine (COMS3011A).
Built with [MkDocs Material](https://squidfunk.github.io/mkdocs-material/), deployed to
GitHub Pages automatically on push to `main` (see `.github/workflows/deploy-docs.yml`).

## Working on it locally

```bash
pip install -r requirements.txt
mkdocs serve
```

Then open http://127.0.0.1:8000 — it live-reloads as you edit files in `docs/`.

## Adding a page

1. Create the `.md` file under `docs/`.
2. Add it to the `nav:` block in `mkdocs.yml` so it's reachable from the site navigation.
3. Commit and push — the site rebuilds and deploys automatically.
