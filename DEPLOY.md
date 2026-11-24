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

Expect it to read about 25 asset files from `./website`. `git ls-files website | wc -l` currently gives 23 tracked files; Wrangler counted 25 in a test run of this config. If the count is very different from that, stop and report.

## 2. Deploy

```sh
npx wrangler deploy
```

## 3. Verify

Check each of these against `https://nd40.com`:

| Request | Expected |
|---|---|
| `/` | 200, the home page with six cards |
| `/about`, `/work`, `/careers`, `/contact`, `/manta`, `/documentation`, `/request-access`, `/terms`, `/privacy`, `/security` | 200 each |
| `/about.html` | redirects to `/about` (normal for `auto-trailing-slash`) |
| `/assets/site.css` | 200, `text/css` |
| `/assets/fonts/oxanium.ttf` | 200 |
| `/favicon.ico` | 200 |
| `/does-not-exist` | 404, showing the "Page not found" page |
| `http://nd40.com/` | 301 to `https://nd40.com/` |
| `https://www.nd40.com/about` | 301 to `https://nd40.com/about` |

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
