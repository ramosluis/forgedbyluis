# Forged by Luis

Source for [forgedbyluis.com](https://forgedbyluis.com/), my personal blog. It's mostly book reviews, plus build logs for my projects (like a 3D printed R2-D2) and the occasional musing.

Built with [Hugo](https://gohugo.io/) on top of the [Hugoplate](https://github.com/zeon-studio/hugoplate) theme, with custom layouts for reviews, projects and musings. Hosted on Cloudflare Pages.

## Stack

| Piece | What it does |
| --- | --- |
| Hugo 0.165.0 extended | Static site generator |
| Hugoplate 3.5.1 | Base theme, in `themes/hugoplate` (never edited directly) |
| Tailwind CSS v4 | Styling; colors and fonts come from `data/theme.json` |
| Cloudflare Pages | Builds and hosts the site from `main` |
| Giscus | Comments, stored in this repo's GitHub Discussions |
| Umami (self-hosted) | Privacy-friendly analytics |
| Obsidian + Templater | Writing new posts |

## Running it locally

### Prerequisites

The versions must match what Cloudflare uses, so they're pinned:

- **Hugo extended 0.165.0**, pinned per folder with [mise](https://mise.jdx.dev/) (`mise.toml`). It must also match `HUGO_VERSION` in `netlify.toml` and in the Cloudflare Pages settings.
- **Go 1.25.1** (for Hugo Modules)
- **Node 24.11.0** with npm

```bash
mise install      # installs the pinned Hugo
npm install
```

### Scripts

Always use the npm scripts, not bare `hugo` commands. They run the Tailwind/theme generation steps Hugo needs.

| Command | What it does |
| --- | --- |
| `npm run dev` | Local server at http://localhost:1313 (drafts are hidden) |
| `npm run build` | Production build into `public/` (what Cloudflare runs) |
| `npm run preview` | Serves a production-like build locally |
| `npm run new -- <path>` | New post from an archetype, e.g. `npm run new -- reviews/wyrd-sisters/index.md` |
| `npm run update-modules` | Updates the Hugo Modules |

If the server gets stuck on port 1313: `npx kill-port 1313 -y && npm run dev`.

Restart the dev server after adding Tailwind classes that aren't used anywhere else yet. Classes are only generated from `hugo_stats.json`.

## Project layout

```
├── archetypes/          # front matter for npm run new (reviews, musings, projects)
├── assets/
│   ├── css/custom.css   # small overrides, inside @layer base / components
│   └── images/          # favicon.png, og-image.png
├── config/_default/     # params.toml (site params, Giscus, Umami), menus.en.toml
├── content/
│   ├── templates/       # Obsidian Templater templates (outside contentDir, never built)
│   └── english/         # the actual site content
│       ├── _index.md    # homepage intro + "Currently reading"
│       ├── reviews/     # one folder per book
│       ├── projects/    # one folder per project, one subfolder per update
│       ├── musings/
│       └── about/, contact/, my-rating-system/
├── data/
│   ├── theme.json       # colors (light + dark) and fonts
│   └── social.json      # footer social links
├── layouts/             # my templates; these override the theme
│   ├── home.html
│   ├── list.html, single.html, term.html
│   ├── reviews/, projects/, musings/
│   ├── _partials/       # post-row, rating, comments, featured-photo, basic-seo
│   ├── _shortcodes/     # rating, figure
│   └── _markup/         # render hooks: callouts (blockquote), images (lightbox)
├── static/_redirects    # Cloudflare redirects (old /posts/ URLs)
├── hugo.toml            # baseURL, taxonomies, plugin list
└── themes/hugoplate/    # the theme, untouched
```

## Content

Every post is a page bundle: a folder with an `index.md` and its images. Sections use `_index.md`.

### Reviews

`content/english/reviews/<book>/index.md`, with the cover saved as `featured.webp` (2:3, ideally 800×1200, at least 460×690).

```yaml
---
title: "Wyrd Sisters"
date: 2025-07-22
draft: false
description: "…"              # SEO and link previews
summary: "…"                  # shown in the feed
tags: [fantasy]               # genres only
series: ["Discworld"]         # optional
series_order: 6
subseries: ["Discworld - Witches"]   # optional
subseries_order: 2
book:
  author: "Terry Pratchett"
  rating: 4                   # 1–5, or 0 for DNF
  original_language: "Spanish"   # only for non-English originals
  goodreads: "https://…"      # optional
---
```

### Projects

Each project is a folder with an `_index.md` (title, description, status). Each update is a subfolder with an `index.md`. Updates are numbered by date automatically.

```
projects/r2-d2/_index.md
projects/r2-d2/printing-the-dome/index.md
```

### Musings

`content/english/musings/<slug>/index.md`. Add `show_in: ["reviews"]` to also list a musing on the Reviews page (for example, reading wrap-ups).

### Writing extras

- `{{< rating 4 >}}` shows inline stars (0 = DNF).
- Obsidian-style callouts work: `> [!note]`, `> [!tip]`, `> [!warning]`, `> [!quote]`. Add `-` or `+` after the type to make them collapsible.
- Clicking an image opens it full size (GLightbox).
- `featured.*` in a bundle becomes the post's photo. Add `featured_alt` and `caption` in the front matter.

## Writing with Obsidian

The Obsidian vault is the `content/` folder. Templates live in `content/templates/`, and the Templater plugin uses them to ask for the title, rating and so on, then create the folder and `index.md` in the right place. The templates are:

- New review
- New musing
- New project
- New project update

`.obsidian/` and `.trash/` are gitignored. New posts start as `draft: true`, so they won't show in `npm run dev` until that's set to `false`.

## Deployment

Every push to `main` deploys to production on Cloudflare Pages. Other branches get preview URLs.

- Build command: `npm run build`
- Output directory: `public`
- Environment variables: `HUGO_VERSION=0.165.0`, `GO_VERSION=1.25.1`, `NODE_VERSION=24.11.0`

To bump Hugo, update `mise.toml`, `netlify.toml` and the Cloudflare variable together.

Redirects live in `static/_redirects`. The first matching rule wins, so specific rules go above the `/posts/*` catch-all.

## Credits

- Theme: [Hugoplate](https://github.com/zeon-studio/hugoplate) by [Zeon Studio](https://zeon.studio/), MIT licensed. See [`LICENSE`](LICENSE).
- Hugo Modules by [Gethugothemes](https://gethugothemes.com/hugo-modules).
- Icons: [Font Awesome](https://fontawesome.com/).
- Fonts: [Figtree](https://fonts.google.com/specimen/Figtree) and [Newsreader](https://fonts.google.com/specimen/Newsreader).

The theme code is MIT licensed. The writing and photos on the site are © Luis, all rights reserved.