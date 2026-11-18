# AGENTS.md

Context for anyone (human or agent) working on **rodolfocaldeira.com**.

## How the site is put together

This repo (`rodolfocaldeira/rodolfocaldeira.github.io`) is the GitHub **user site**,
and its custom domain is `rodolfocaldeira.com`. Because it is the user site, every
other repo of mine that has GitHub Pages turned on is automatically served at
`rodolfocaldeira.com/<repo-name>/`, with HTTPS. Nothing has to be copied in here.

| What | Repo | URL | Publishes on |
|---|---|---|---|
| Main site | `rodolfocaldeira.github.io` (this repo, was `rodolfocaldeira.com`) | `/` | push to `master`; `.github/workflows/pages.yml` publishes `public/` |
| Blog (Jekyll) | `blog` (was `rodolfocaldeira.github.com`) | `/blog/` | push to `master`; GitHub builds Jekyll (`baseurl: /blog`) |
| Expenses calculator | `expenses-calculator` | `/expenses-calculator/` | push to `gh-pages` |
| CV | static files in `public/cv/` of this repo | `/cv/google/` | push to this repo's `master` |
| Old blog domain | `blog-redirect` | `blog.rodolfocaldeira.com/*` redirects to `/blog/*` | push to `main` |
| Old labs path | `public/labs/expenses-calculator/index.html` | `/labs/expenses-calculator/` redirects to `/expenses-calculator/` | with this repo |

**To publish a new one-page project:** create a repo, turn on Pages (root or `docs/`
of `main`, or a GitHub Actions workflow), and it goes live at
`rodolfocaldeira.com/<repo>/`. Use relative or base-path-aware asset URLs.

**DNS:** apex A/AAAA records point at GitHub Pages; `www` and `blog` are CNAMEs to
`rodolfocaldeira.github.io`. Domain, DNS and email details are kept privately, not in this repo.

## TODO

- [x] Rename blog repo `rodolfocaldeira.github.com` to `blog`, remove its CNAME, set `baseurl: /blog`, and prefix internal links with `site.baseurl`
- [x] Replace the deprecated `pygments: true` with `highlighter: rouge` in the blog
- [x] Rename this repo from `rodolfocaldeira.com` to `rodolfocaldeira.github.io` (user site)
- [x] Publish from `master/public/` via GitHub Actions instead of a hand-maintained `gh-pages` branch
- [x] Bring `public/` in line with what was live on `gh-pages` (CNAME, icons, `cv/`)
- [x] Remove the Capistrano deploy (`Capfile`, `config/deploy*`) and the `expenses-calculator` submodule
- [x] Serve expenses calculator from its own repo at `/expenses-calculator/`; redirect `/labs/expenses-calculator/`
- [x] Create `blog-redirect` repo so old `blog.rodolfocaldeira.com/*` links go to `/blog/*`
- [x] Enforce HTTPS on every Pages site
- [x] ~~Get the Pages certificate to cover `www.rodolfocaldeira.com`~~ Won't fix: `http://www.` redirects fine and the apex works over HTTPS; only `https://www.` errors (see learnings)
- [x] Delete the stale `gh-pages` branch (deleted 2026-11-18; its last commit was `a056eb5`, recoverable with `git push origin a056eb5:refs/heads/gh-pages` if the object is still on GitHub)
- [x] ~~Publish the CV from its own repo~~ Decided against it: the CV stays as static files in `public/cv/`, with its source kept elsewhere (a private GitHub repo can't serve Pages on the Free plan)
- [ ] Optional: move `expenses-calculator` from `gh-pages` to `main`, so it has a single branch

## Findings & learnings

- **The user site's custom domain applies to all project sites.** This is what removes
  the need to copy sub-projects into `gh-pages`.
- **Only one repo can be the user site.** The legacy `<user>.github.com` name still counted,
  so the blog had to be renamed before this repo could become `rodolfocaldeira.github.io`.
- **Cause of the old blog's "not secure" warning:** GitHub never issued a certificate for
  `blog.rodolfocaldeira.com`. Attaching the domain to a fresh repo (`blog-redirect`)
  triggered a new certificate within a few minutes.
- **Clearing a custom domain through the API commits a "Delete CNAME" to the source branch**
  (legacy branch builds). Pull or rebase before pushing afterwards.
- **GitHub reuses an existing valid certificate.** Removing and re-adding the domain
  doesn't force a new one, even after a 2-hour gap, so the apex-only certificate stayed.
  Untried options: make `www` the canonical custom domain (the apex would then redirect to it),
  or ask GitHub Support to re-issue the certificate.
- **GitHub Pages can't send a real 301 from one domain to another.** `blog-redirect` uses a
  JavaScript redirect in `index.html` and `404.html`.
- **With an Actions-based Pages source, the custom domain lives in repo settings.**
  `public/CNAME` is kept anyway for clarity.
- **Pages from a private repo needs GitHub Pro,** and the published site is public either way.

## Handy checks

```bash
for p in "" blog/ expenses-calculator/ labs/expenses-calculator/ cv/google/; do curl -s -o /dev/null -w "%{http_code} /$p\n" "https://rodolfocaldeira.com/$p"; done
```

```bash
gh api repos/rodolfocaldeira/rodolfocaldeira.github.io/pages --jq '.https_certificate|{state,domains}'
```
