# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a personal website built with Hugo, deployed to GitHub Pages at mackeracher.com.

## Commands

**Create a new post:**
```bash
hugo new posts/my-post-title.md
```

**Local development:**
```bash
hugo server -D    # Start dev server with drafts at localhost:1313
```

**Build for production:**
```bash
hugo --gc --minify
```

Output goes to `public/` directory.

## Architecture

- **Hugo static site** using the [typo theme](https://github.com/tomfran/typo) (git submodule in `themes/typo/`)
- **Configuration:** `hugo.toml` contains site settings, menu, social links
- **Content:** Markdown files in `content/` (posts, cv, photos sections)
- **Layout overrides:** `layouts/_default/home.html` customizes the homepage
- **Deployment:** GitHub Actions (`.github/workflows/hugo.yaml`) builds and deploys on push to main

## Social cards (OG/Twitter)

Every post under `content/posts/` automatically gets a branded 1200x630 card
at build time — nothing to do for a new post beyond writing `title` and
`description` in front matter. See `assets/og/README.md` for how it works,
the brand tokens, and how to hand-build a custom card (as
`norns-v0.3`, `introducing-mimir`, and `norns-elixir-sdk-v0.1` do) when you
want more control than the auto-generated headline + subtitle layout.

## Post front matter

Beyond `title`, `date`, `description`, every post carries a `genre`: `essay`,
`note`, or `project` (`archetypes/posts.md` starts new posts as `note`). Only
the homepage uses it: essays get cards, everything else gets a title list
below them, except project posts, which are left to the Building section
from `data/projects.toml`. It isn't shown anywhere. A post with no genre is
treated as a note. Don't call
this key `kind` or `type`: both are reserved by Hugo and 0.128 (the CI
version) refuses `kind`.

Optional keys:

- `audience`: one sentence shown in an "Assumed audience" box under the title.
- `lastmod`: shown as "Updated <date>" next to the written date when it
  differs from `date`. Only set it for real revisions, not typo fixes.
- `showUpdated = true`: show only the last git commit date (content/now.md
  uses this, so editing it is all it takes to bump the date).
- `series` / `seriesBlurb`: see data/series.toml.

Every post page ends with a "Mentioned in" list, built at compile time from
other pages whose markdown links to it (layouts/partials/backlinks.html).

Card thumbnails on the homepage and /posts are the post's social card
(layouts/partials/thumb.html), so a hand-built `images` card shows there too.
`cardThumbs = false` in hugo.toml turns them off.
