---
name: chirpy-upgrade
description: Use when upgrading this repository's Jekyll Chirpy theme, especially for merging a new upstream Chirpy release into wabol/wabol.github.io, resolving repo-specific conflicts, rebuilding assets, verifying with Docker Ruby 3.4, and switching between custom domain and github.io deployment.
---

# Chirpy Upgrade

This skill is for upgrading the Chirpy theme in this repository: `/Users/jiangbo/Downloads/workspace/lyducka.github.io`.

Use it when the task is to:

- upgrade Chirpy to a newer upstream tag
- merge upstream `cotes2020/jekyll-theme-chirpy` into this site repo
- fix upgrade conflicts in `_config.yml`, layouts, workflows, and built assets
- validate the upgraded site without relying on the host Ruby
- switch Pages deployment between a custom domain and `wabol.github.io`

## Repository assumptions

- Site repo: `wabol/wabol.github.io`
- Upstream theme repo: `cotes2020/jekyll-theme-chirpy`
- Pages deploy branch: `master`
- Preferred upgrade branch pattern: `codex/upgrade-chirpy-X.Y.Z`
- Current Pages workflow is `.github/workflows/pages-deploy.yml`
- Host Ruby may be too old; prefer Docker for build verification

## Safe upgrade workflow

1. Confirm clean working tree.
2. Create a rollback tag before touching the upgrade branch.
3. Create a dedicated upgrade branch from `master`.
4. Fetch upstream tags.
5. Merge the target Chirpy tag into the upgrade branch.
6. Resolve conflicts using repo-specific rules below.
7. Rebuild JS/CSS assets if the upgraded theme expects generated files.
8. Verify build and HTML output with Docker Ruby 3.4.
9. Push the branch and open a PR into `master`.
10. Only after checks pass, merge and then adjust GitHub Pages settings if domain behavior changed.

## Commands

Run from the repo root.

```bash
git status --short
git tag before-chirpy-<version>
git switch -c codex/upgrade-chirpy-<version>
git fetch upstream --tags
git merge --no-ff v<version>
```

## Conflict resolution rules for this repo

Prefer upstream for:

- `Gemfile`
- `.gitignore` except when this repo intentionally tracks built assets
- bundled tutorial posts under `_posts/2019-*`

Prefer repo-specific values on top of v7 config structure for:

- `_config.yml`
- `.github/workflows/pages-deploy.yml`
- custom analytics settings
- comments provider settings
- avatar, title, and site URL
- domain choice (`CNAME` present or absent)

Prefer upstream layout structure for:

- `_layouts/default.html`
- `_includes/*`
- `_javascript/*`
- `_sass/*`

Then re-apply repo-specific behavior only if still needed.

## Repo-specific config mapping

When migrating to Chirpy v7-style config:

- `google_analytics.id` -> `analytics.google.id`
- `img_cdn` -> `cdn`
- `comments.active` -> `comments.provider`
- custom analytics script should be replaced by supported analytics includes, or removed

This repo previously used:

- `analytics.google.id`
- optional `analytics.umami`
- `comments.provider: giscus`
- `cdn: "https://chirpy-img.netlify.app"`
- `url: "https://wabol.github.io"`

## Built assets rule

This repo's Pages workflow does **not** build frontend assets before Jekyll build.

That means these generated files must exist in git when the upgraded theme expects them:

- `assets/js/dist/*.min.js`
- any vendored Sass entry the build imports

After upgrade, rebuild assets locally:

```bash
npm install
npm run build
```

If Jekyll on CI fails with:

```text
Can't find stylesheet to import.
@use 'vendors/bootstrap';
```

check whether `_sass/vendors/_bootstrap.scss` exists locally but was ignored by git. If so, force-add it:

```bash
git add -f _sass/vendors/_bootstrap.scss
```

## Verification workflow

Do not trust the host Ruby version on this machine. Use Docker Ruby 3.4.

Start Docker Desktop if needed, then run:

```bash
docker run --rm -u 501:20 \
  -v /Users/jiangbo/Downloads/workspace/lyducka.github.io:/app \
  -w /app ruby:3.4 bash -lc \
  "bundle config set --local path vendor/bundle && \
   bundle exec jekyll build --trace && \
   bundle exec htmlproofer _site --disable-external --ignore-urls '/^http:\\/\\/127.0.0.1/,/^http:\\/\\/0.0.0.0/,/^http:\\/\\/localhost/'"
```

Expected outcome:

- `jekyll build` succeeds
- `htmlproofer` succeeds

Warnings to investigate if they appear:

- Liquid warnings from literal template syntax inside posts
- category page destination conflicts caused by inconsistent category capitalization

## Known repo-specific fixes

### 1. Literal templating inside posts

If a post contains Go-template-like strings such as `{{.Service.Name}}`, add:

```yaml
render_with_liquid: false
```

to that post's front matter.

### 2. Category slug conflicts

Normalize category casing. For this repo, these should stay capitalized consistently:

- `Linux`
- `Docker`
- `Kubernetes`

Do not mix them with `linux`, `docker`, or `kubernetes`.

### 3. Hanging page load from analytics

If the page renders but the browser tab never finishes loading, inspect external scripts in the live HTML.

This repo previously had a hanging Umami script at:

```text
https://tj.698423.xyz/script.js
```

If it hangs, disable Umami in `_config.yml` by clearing:

```yml
analytics:
  umami:
    id:
    domain:
```

## Domain switching

To switch back to default GitHub Pages domain:

- delete `CNAME`
- keep `_config.yml` `url` aligned with `https://wabol.github.io`
- after merge, clear `Custom domain` in GitHub `Settings -> Pages`

To use a custom domain again:

- restore `CNAME`
- set Pages custom domain in GitHub settings
- ensure DNS points to GitHub Pages

## PR guidance

Open the PR from:

- base repo: `wabol/wabol.github.io`
- base branch: `master`
- compare branch: `codex/upgrade-chirpy-X.Y.Z`

Do **not** open a PR against `cotes2020/jekyll-theme-chirpy`.

Suggested PR title:

```text
chore: upgrade Chirpy to vX.Y.Z
```

Suggested summary:

- upgrade Chirpy from current version to target version
- migrate config to v7 schema if needed
- rebuild JS assets required by Pages deploy
- adjust domain behavior if `CNAME` changed
- verify with Docker `jekyll build` and `htmlproofer`

## Final pre-merge checklist

- upgrade branch pushed
- PR targets `wabol/wabol.github.io:master`
- CI, Lint JS, Lint SCSS, commit message lint, and CodeQL are green
- Pages build for the branch is green if present
- no missing generated assets
- no hanging third-party analytics scripts
- `CNAME` state matches desired domain behavior
