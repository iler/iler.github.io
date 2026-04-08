# Hugo Blog Setup Design

**Date:** 2026-04-08  
**Status:** Approved

## Context

Ilari has an existing Ghost blog at ilar.in with 11 posts (2017–2020), mostly technical networking content. The goal is to migrate to Hugo on GitHub Pages (iler.github.io), pointed at the custom domain ilar.in, with automatic deployment via GitHub Actions. The repo is currently empty.

---

## Architecture

### Repository layout

```
iler.github.io/
├── .github/
│   └── workflows/
│       └── deploy.yml         # CI/CD: build Hugo + deploy to GitHub Pages
├── themes/
│   └── PaperMod/              # git submodule → upstream PaperMod repo
├── content/
│   └── posts/                 # migrated blog posts as .md files
├── static/
│   ├── CNAME                  # contains "ilar.in"
│   └── images/                # images downloaded from Ghost
├── archetypes/
│   └── default.md
└── hugo.toml                  # site configuration
```

### Hugo configuration (`hugo.toml`)

- `baseURL = "https://ilar.in/"`
- `theme = "PaperMod"`
- `enableRobotsTXT = true`
- Taxonomies: tags
- PaperMod params: `homeInfoParams` (intro text), `ShowReadingTime`, `ShowWordCount`, `ShowPostNavLinks`, `ShowBreadCrumbs`, `ShowCodeCopyButtons`
- No comments configured

### Theme management

PaperMod added as a git submodule:

```sh
git submodule add --depth=1 https://github.com/adityatelange/hugo-PaperMod.git themes/PaperMod
```

To update later: `git submodule update --remote --merge`

---

## GitHub Actions Workflow

File: `.github/workflows/deploy.yml`

Triggers on push to `main` and supports `workflow_dispatch` (manual trigger from GitHub UI).

```yaml
on:
  push:
    branches: [main]
  workflow_dispatch:

permissions:
  contents: read
  pages: write
  id-token: write

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
        with:
          submodules: true
          fetch-depth: 0        # needed for Hugo .GitInfo and lastmod
      - uses: peaceiris/actions-hugo@v3
        with:
          hugo-version: 'latest'
          extended: true        # PaperMod requires Hugo extended (SCSS)
      - run: hugo --minify
      - uses: actions/upload-pages-artifact@v3
        with:
          path: ./public

  deploy:
    needs: build
    runs-on: ubuntu-latest
    environment:
      name: github-pages
      url: ${{ steps.deployment.outputs.page_url }}
    steps:
      - uses: actions/deploy-pages@v4
        id: deployment
```

GitHub Pages must be configured to use **GitHub Actions** as the source (Settings → Pages → Source → GitHub Actions).

---

## Custom Domain (ilar.in)

### GitHub side
- `static/CNAME` contains the single line `ilar.in`
- Hugo copies this to `public/CNAME` on build
- GitHub Pages reads it and configures the custom domain automatically
- GitHub Pages provisions and auto-renews TLS via Let's Encrypt

### DNS side (manual step by user)
The user must configure DNS at their domain registrar. Two options:

**Option A — Apex domain (recommended):** Add four A records pointing `ilar.in` to GitHub Pages IPs:
```
185.199.108.153
185.199.109.153
185.199.110.153
185.199.111.153
```
Also add AAAA records for IPv6 if desired.

**Option B — CNAME subdomain:** If using `www.ilar.in`, add a CNAME record pointing to `iler.github.io`.

Since `ilar.in` is the apex domain (no subdomain), Option A (A records) is required by DNS standards. A CNAME at the apex is not valid DNS.

---

## Content Migration

### 11 posts to migrate

| Slug | Date | Last Modified |
|------|------|---------------|
| dyndns-with-namecheap-on-unifi-usg | 2020-03-27 | 2020-03-27 |
| wireguard-with-unifi-usg | 2020-03-23 | 2020-08-27 |
| my-home-network-setup | 2020-03-23 | 2020-03-23 |
| two-factor-authentication-how-to-do-secure-usable-way | 2020-03-22 | 2020-03-22 |
| when-your-pet-gets-sick | 2018-01-03 | 2018-01-03 |
| sleep | 2018-01-02 | 2018-01-02 |
| my-new-years-resolutions | 2018-01-01 | 2018-01-01 |
| ipv6-not-so-ready-yet | 2017-04-17 | 2017-04-17 |
| upgrading-firmware-for-netgear-gs105ev2-switch-on-mac-os | 2017-04-08 | 2017-04-08 |
| critical-thinking-outside-april-1st | 2017-04-01 | 2017-04-01 |
| place-for-my-daily-thoughts | 2017-04-01 | 2017-04-01 |

### Front matter format

```markdown
---
title: "Post Title"
date: 2020-03-23T00:00:00+00:00
lastmod: 2020-08-27T00:00:00+00:00
tags: ["tag1", "tag2"]
draft: false
---
```

### Conversion process

1. Fetch each post's HTML from ilar.in
2. Convert HTML body to Markdown (strip Ghost-specific elements, preserve code blocks with language hints)
3. Download all images to `static/images/<slug>/` and update src references to `/images/<slug>/filename`
4. Write to `content/posts/<slug>.md`

### URL compatibility

Hugo will generate URLs matching the original Ghost slugs (e.g., `/wireguard-with-unifi-usg/`), so existing bookmarks and links remain valid after cutover.

---

## Verification

1. Run `hugo server` locally after setup — site should build and serve at `http://localhost:1313`
2. All 11 posts visible with correct dates and tags
3. Images render correctly (no broken image links)
4. Push to `main` → GitHub Actions deploys → check `https://iler.github.io` renders correctly
5. Configure DNS → wait for propagation → check `https://ilar.in` works with valid TLS
6. Confirm GitHub Pages Settings shows "Your site is live at https://ilar.in"
