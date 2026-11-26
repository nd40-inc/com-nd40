# Redeploy and verify the Nd40 website

How to publish changes to https://nd40.com and check them. For what the repo contains, see `README.md`.

## Rules

- Do not add analytics, bot protection or any other product unless James asks.
- Do not touch other Workers, Pages projects, DNS records or zone settings in the account. Other live projects run there.
- Ask before deleting anything.

## Prerequisites

Run `npx wrangler login` as the nd40 account. `wrangler.jsonc` pins that account's `account_id`, so a deploy cannot land elsewhere. Confirm with `npx wrangler whoami`.

## 1. Dry run

From the repo root:

```sh
npx wrangler deploy --dry-run
```

Expect it to read about 24–26 asset files from `./website` (`_headers` is processed as a Cloudflare Assets config file rather than served directly). If the count is very different from that, stop and report.

## 2. Deploy

```sh
npx wrangler deploy
```

## 3. Verify

Check each of these against `https://nd40.com`:

| Request | Expected |
|---|---|
| `/` | 200, the home page with six cards |
| `/work`, `/case-studies`, `/careers`, `/contact`, `/request-access`, `/terms`, `/privacy`, `/security` | 200 each |
| `/assets/site.css` | 200, `text/css` |
| `/assets/fonts/oxanium.woff2` | 200, `Cache-Control: public, max-age=31536000, immutable` |
| `/favicon.ico`, `/favicon.svg` | 200 |
| `/robots.txt`, `/sitemap.xml`, `/.well-known/security.txt` | 200 |
| `/does-not-exist` | 404, showing the "Page not found" page |
| `http://nd40.com/` | 301 to `https://nd40.com/` |
| `https://www.nd40.com/work` | 301 to `https://nd40.com/work` |

Then open the home page in a browser and confirm:

- The wordmark reads "Nd40" with an orange "40" whose zero has a dot in it.
- Text is in the Oxanium typeface (squared letterforms), not a default serif or sans.
- Clicking the logo from any subpage returns to the home page.
- Every footer link opens its page.

If any check fails, roll back (below) and report.

## Report back

- The Worker name and the deployment or version ID
- The result of each check
- Anything you skipped, and why

## Rollback

```sh
npx wrangler rollback
```

This returns the Worker to its previous version. To take the site down entirely, remove the routes from `wrangler.jsonc` and redeploy, or delete the Worker. Ask James before deleting anything.
