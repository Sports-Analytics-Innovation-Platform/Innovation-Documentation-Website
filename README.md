# Docs site

Documentation website for the FPL Analytics & Optimisation Engine (COMS3011A).
Built with [MkDocs Material](https://squidfunk.github.io/mkdocs-material/), deployed to
GitHub Pages automatically on push to `main`.

## First-time repo setup (do this once)

1. Push this content to your empty GitHub repo (see commands below).
2. In the repo: **Settings → Pages → Source → Deploy from a branch → `gh-pages` / `root`**.
   (The `gh-pages` branch doesn't exist yet — it's created the first time the Action runs.)
3. Push to `main` once, wait for the "Deploy docs site" Action to finish (Actions tab),
   then refresh Settings → Pages — it'll show your live URL.
4. Update `repo_url` in `mkdocs.yml` to your actual repo URL.

## Push this scaffold to your repo

```bash
cd docs-site
git init
git add .
git commit -m "docs: scaffold MkDocs Material site"
git branch -M main
git remote add origin https://github.com/YOUR_ORG/YOUR_DOCS_REPO.git
git push -u origin main
```

## Working on it locally

```bash
pip install -r requirements.txt
mkdocs serve
```

Then open http://127.0.0.1:8000 — it live-reloads as you edit files in `docs/`.

## Adding a page

1. Create the `.md` file under `docs/` (or use an existing stub — every doc from the plan
   already has a placeholder file so the nav doesn't break).
2. If it's a genuinely new page (not one of the stubs), add it to the `nav:` block in
   `mkdocs.yml`.
3. Commit and push — the site rebuilds automatically.

## Structure

Every stub file in `docs/` corresponds to one of the documents on the team's doc list.
Delete the "Status: TODO" notice block at the top once you've actually written the content.
