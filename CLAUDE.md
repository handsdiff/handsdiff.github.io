# handsdiff/handsdiff.github.io

Quartz v5 site that publishes the Obsidian vault at `~/Documents/Notes`
(repo: `handsdiff/notes`, public) to GitHub Pages at
`https://handsdiff.github.io/`.

This repo was renamed from `handsdiff/garden` to `handsdiff/handsdiff.github.io`
so the site is served at the bare domain root (no `/garden` subpath). As a
result `baseUrl` is `handsdiff.github.io` (no path) and the `cname` plugin is
disabled (not needed — this repo's name *is* the GitHub Pages domain, so no
custom domain / CNAME record is required).

## Architecture

- This repo holds the full Quartz v5 project (cloned from
  `jackyzha0/quartz`, default branch `v5`).
- `content/` is NOT the source of truth — it's overwritten on every
  workflow run by `.github/workflows/deploy.yml`'s "Sync vault content"
  step, which does `rm -rf content && git clone --depth 1
  https://github.com/handsdiff/notes.git content && rm -rf content/.git`.
  Never edit files in `content/` directly; edit the vault instead.
- Deploy workflow triggers: push to `v5`, `*/10 * * * *` cron, and
  `workflow_dispatch`. End-to-end latency from a vault edit to the live
  site is roughly 10-20 min (obsidian-git's ~10min auto-push + this
  repo's ~10min cron + ~1-2min build).

## Identity / remotes

- Git identity for this repo is `handsdiff` /
  `239876380+handsdiff@users.noreply.github.com` — completely separate
  from the user's other (non-public) GitHub identity. Do not introduce
  that other identity, or any reference to its username, into this
  repo's history or config.
- `origin` -> `git@github.com-handsdiff:handsdiff/handsdiff.github.io.git`
  (uses the `github.com-handsdiff` Host alias in `~/.ssh/config`, key
  `~/.ssh/id_ed25519_handsdiff`).
- `upstream` -> `https://github.com/jackyzha0/quartz.git` (for pulling
  Quartz updates via `npx quartz update` if desired).

## Gotchas discovered during setup

- **Don't add `content/` to `.gitignore`.** Quartz's build uses
  globby's `isGitIgnored()` to filter content files — anything matched
  by `.gitignore` (tracked or not) is silently excluded from the build,
  which produced a site with 0 pages. `content/` is instead listed in
  `.git/info/exclude` (local-only; not read by globby's
  `isGitIgnored()`), so `git status` stays clean without breaking the
  build.
- **`actions/configure-pages@v5` is required** before
  `upload-pages-artifact`/`deploy-pages` — Quartz's hosting docs example
  omits it, but without it `deploy-pages` fails with
  `HttpError: Not Found`.
- `quartz.config.yaml` `pageTitle: "Quartz 5"` is just the unedited
  default placeholder, not special branding — change it if desired.
