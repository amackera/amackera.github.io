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
