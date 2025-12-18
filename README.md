# mackeracher.com

Personal website built with [Hugo](https://gohugo.io/) using the [typo](https://github.com/tomfran/typo) theme.

## Development

### Prerequisites

- [Hugo](https://gohugo.io/installation/) (extended version)

### Running locally

```bash
hugo server -D
```

The site will be available at `http://localhost:1313/`.

### Building

```bash
hugo
```

Output is generated in the `public/` directory.

## Deployment

The site is automatically deployed to GitHub Pages via GitHub Actions when changes are pushed to the main branch.
