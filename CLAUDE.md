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
