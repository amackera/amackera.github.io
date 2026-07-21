# OG card template

Reference implementation for post social cards (1200×630). This is the
hand-built template behind the card in `content/posts/norns-elixir-sdk-v0.1/`.
The goal (see the automation task) is to turn this into an automatic
per-post generator so every new post gets a branded card.

## Files

- `template.html` — the card layout (dark brand bg, radial glows, norns rune
  logo + wordmark, headline, subtitle, footer badge, code panel). The
  per-post parts are the `<h1>` headline / `.ver`, the `.sub` subtitle, and
  the `.card` code panel — those are what a generator should templatize from
  front matter.
- `norns-glyph.png` — the norns rune, center-cropped from
  `nornscode.com/assets/norns-logo.png` to remove margins. It is a black glyph
  on white; the template whitens it with an inline SVG `feColorMatrix`
  luminance filter (a naive `invert` yields a white square).

## Rendering

No SVG converter is installed; render with headless Chrome:

```sh
"/Applications/Google Chrome.app/Contents/MacOS/Google Chrome" \
  --headless --disable-gpu --no-sandbox --hide-scrollbars \
  --allow-file-access-from-files --force-device-scale-factor=2 \
  --window-size=1200,630 \
  --screenshot=out.png \
  "file://$PWD/template.html"
```

Output is 2400×1260 (2×); that's fine for OG (X downscales).

## Brand tokens

bg `#0a0c0f` · text `#e7ecf2` · secondary `#aab2bd` · muted `#6b7480` ·
accent gold `#d4a45a` · blue `#7ca8d1` · monospace throughout
(JetBrains Mono preferred; falls back to system mono without the font).
Canonical layout reference: `nornscode.com/assets/og.svg`.
