# Nd40

- **icons/**: PNGs at 16, 32, 48, 64, 128, 256, 512 and 1024 px, transparent outside the rounded corners, plus `favicon.ico`. Sizes 64 px and below use the small version (no 40 in the corner) so they stay legible.
- **print/**: `nd40-icon.pdf` is vector and scales to any size. `nd40-icon-4096.png` is a 4096 px raster at 300 dpi (about 13.6 in / 34.7 cm square).
- **github/**: `nd40-github.png`, 1024 px, full bleed with no border. Upload as the GitHub avatar; GitHub rounds the corners itself.
- **svg/**: `nd40-icon.svg`, the master icon with lettering converted to outlines, so it needs no fonts.
- **website/**: the static site (plain HTML and one CSS file, no build step, no JavaScript). Open `index.html` to preview it. The logo returns home from every page.
- **LICENSES/fonts/**: licence texts for the self-hosted fonts.
- **wrangler.jsonc**: Cloudflare Worker config that serves `website/` as static assets.

## Deployment

The site is the Worker `nd40-website` on the Cloudflare account "nd40" (the `account_id` is pinned in `wrangler.jsonc`, so a deploy cannot land on another account). It is served on the custom domains `nd40.com` and `www.nd40.com`. The workers.dev address is off.

Two settings live in the Cloudflare dashboard, not in this repo: "Always Use HTTPS" is on, and a Single Redirect rule sends `www` to the root domain (301, query string kept).

To deploy, run `npx wrangler login` as the nd40 account, then `npx wrangler deploy`. To roll back, run `npx wrangler rollback`. See [DEPLOY.md](DEPLOY.md) for the checks.
