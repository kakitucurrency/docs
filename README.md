# docs.kakitu.org (retired)

This repository used to publish the Nano-era Kakitu documentation. That content described a
product that no longer exists and has been removed (it stays in git history).

The site now builds exactly one page, `docs/index.md`, which redirects to
**https://developer.kakitu.org** (source: `kakitucurrency/kakitu-docs`).

Deploy: a push to `main` runs `.github/workflows/deploy.yml`, which builds with MkDocs
Material, asserts the build contains a single page, and runs `mkdocs gh-deploy --force`
to the `gh-pages` branch that GitHub Pages serves under the `docs/CNAME` domain.
