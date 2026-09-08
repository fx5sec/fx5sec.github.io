# fx5's Blog

Digital forensics articles, browser artifact guides, and CTF write-ups built with [Zensical](https://zensical.org/).

Site address: [fx5sec.github.io](https://fx5sec.github.io/).

## Preview locally

Use Python 3.13 and a project virtual environment:

```sh
python3.13 -m venv .venv
source .venv/bin/activate
python -m pip install -r requirements.txt
zensical serve
```

Build the static site with:

```sh
source .venv/bin/activate
zensical build --clean --strict
```

Content and assets live in `docs/`; configuration lives in `zensical.toml`. The build writes `site/`, which is generated and excluded from version control.
`requirements.txt` pins the tested Zensical release; Python package dependencies are resolved by pip when installing it.

## GitHub Pages

The public repository is intended to be `fx5sec/fx5sec.github.io`. In the repository's **Settings > Pages > Build and deployment**, select **GitHub Actions**.
The workflow in `.github/workflows/docs.yml` builds and deploys on pushes to `main`. It can also be run manually from the Actions tab using `main`.
It uploads only the generated `site/` artifact.

The repository contains the site source and publishing files. Local tooling, review drafts, working notes, and build output are excluded by `.gitignore`.
Only content ready for publication belongs in `docs/`.

See [Zensical's publishing instructions](https://zensical.org/docs/publish-your-site/#github-pages)
and [GitHub's Pages workflow documentation](https://docs.github.com/en/pages/getting-started-with-github-pages/using-custom-workflows-with-github-pages).
