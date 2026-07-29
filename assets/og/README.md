# OG / Twitter card generation

Every post under `content/posts/` gets a branded 1200x630 social card. By
default this is fully automatic: write `title` and `description` in front
matter, run `hugo`, done. A post can opt out of the automatic card by setting
its own `images` front-matter param (a few posts do this — see "Hand-built
cards" below).

The default card carries this *site's* brand (mackeracher.com), not the
Norns project's — see "Two brands" below before reaching for `template.html`.

## How the automatic pipeline works

1. `assets/og/bg.png` is a pre-rendered 2400x1260 (2x) background: dark bg,
   radial glows, the "Anson MacKeracher" wordmark top-left, and a
   `mackeracher.com` footer badge. It has no post-specific text on it — it's
   shared by every auto-generated card. See "Regenerating the background"
   below if the brand ever changes.
2. `layouts/partials/og/generate.html` takes a page, word-wraps its `.Title`
   and `.Description` (falling back to the site description if a post has
   none) using `layouts/partials/og/wrap.html`, and stamps them onto a copy
   of `bg.png` with Hugo's `images.Text` filter. The headline picks the
   largest size (from a fixed candidate list) that still wraps the title to
   two lines or fewer; the subtitle wraps to up to two lines at a fixed size.
   Text is rendered with the bundled JetBrains Mono TTFs in
   `assets/fonts/JetBrainsMono/` (Hugo's image filters need a real TTF/OTF
   resource — the theme's webfonts are woff2/woff and don't work here).
3. The result is resized to exactly 1200x630 and copied to
   `og/generated/<slug>-og.png` as a page-independent resource (via
   `resources.Copy`), so it gets a stable, predictable URL.
4. `layouts/partials/og/image.html` resolves the image list for a page: an
   explicit `.Params.images` wins if set; otherwise, for any regular page in
   the `posts` section, it calls the generator above. Both
   `layouts/partials/opengraph.html` (og:image) and
   `layouts/partials/hooks/head_end.html` (twitter:card /
   twitter:image) call this same partial (via `partialCached`, keyed on the
   page, so the image is only generated once per build) — that's the only
   plumbing involved; nothing else needs to change per-post.

Non-post pages (home, cv, projects, photos, the posts list) never get a
generated card — the `og/image.html` partial only fires for pages in the
`posts` section.

## Two brands

There are two distinct visual identities in this directory — don't mix them
up:

- **`bg.png` / `bg-template.html`** — this site's brand (mackeracher.com):
  plain wordmark, no logo, `mackeracher.com` footer. This is what every
  auto-generated card uses. It's what you want for personal posts, essays,
  anything that isn't specifically about the Norns project.
- **`template.html`** — the Norns project's brand (norns rune logo,
  `open source · mit · hex.pm` footer, matching `nornscode.com`). Only use
  this for posts that are actually about Norns/its SDKs (the three hand-built
  cards below all are). Don't apply it to unrelated posts — it reads as
  mislabeling the post as Norns project content when it's shown in a social
  preview.

## Hand-built cards

Three posts (`norns-v0.3`, `introducing-mimir`, `norns-elixir-sdk-v0.1`) have
a manually authored `images` front-matter param and are never touched by the
generator. All three are genuinely about Norns, so they use `template.html`
(headline + subtitle + a bespoke syntax-highlighted code panel) — reach for
it when a *Norns* post deserves more than the generic headline/subtitle
treatment and you're willing to do the render step by hand:

```sh
"/Applications/Google Chrome.app/Contents/MacOS/Google Chrome" \
  --headless --disable-gpu --no-sandbox --hide-scrollbars \
  --allow-file-access-from-files --force-device-scale-factor=2 \
  --window-size=1200,630 \
  --screenshot=out.png \
  "file://$PWD/template.html"
```

Output is 2400x1260 (2x); that's fine for OG (X downscales). This is the
"worse, manual" fallback path mentioned in the original automation task —
prefer the automatic pipeline for everything that isn't Norns-branded, or if
`images.Text` fidelity genuinely isn't enough for what a specific post needs.

## Regenerating the background

`bg-template.html` is the source for `bg.png` — same dark bg/glows as
`template.html`, but with the Norns logo/wordmark/footer swapped for this
site's own (see "Two brands" above), and no headline/subtitle/code-panel
(those get stamped in per-post at build time). If the brand changes, edit it
and re-render with the same headless-Chrome command as above (targeting
`bg-template.html`, output to `bg.png`).

## Brand tokens

bg `#0a0c0f` · text `#e7ecf2` · secondary `#aab2bd` · muted `#6b7480` ·
accent gold `#d4a45a` · blue `#7ca8d1` · monospace throughout (JetBrains
Mono). Shared between both brands above; only the wordmark/logo and footer
text differ.

`norns-glyph.png` is the norns rune, center-cropped from
`nornscode.com/assets/norns-logo.png` to remove margins, used only by
`template.html` (the Norns brand). It's a black glyph on white; whitened with
an inline SVG `feColorMatrix` luminance filter (a naive `invert` yields a
white square).
