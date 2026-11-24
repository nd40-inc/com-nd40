# Deploy the Nd40 website to Cloudflare

Instructions for an agent with access to the Nd40 Cloudflare account.

## Goal

Publish the static site in `website/` as a Cloudflare Worker with static assets, confirm it works on the `workers.dev` address, then attach the production domain.

## What you are given

This folder (`Nd40/`) contains:

- `website/`: the complete site. Plain HTML and CSS, no build step, no JavaScript, no server code.
  - `index.html` (home), ten subpages, and `404.html`
  - `assets/site.css` and `assets/fonts/` (self-hosted fonts, about 3 MB in total)
  - `favicon.ico`, `favicon.svg`, `apple-touch-icon.png`
- `wrangler.jsonc`: ready-made config that serves `./website`.

## Inputs to get from James before the production step

- **Domain:** the hostname the site should live on, written as `[DOMAIN]` below. Not decided in this document. Do not guess it.
- **Account:** which Cloudflare account to deploy to, if more than one is available.

## Rules

- Do not edit anything inside `website/`. The files are final. If something looks wrong, report it and stop.
- Do not add analytics, redirects, bot protection, or any other product unless James asks.
- Do not touch any other Worker, Pages project, DNS record or zone setting in the account. Other live projects run there.
- If a Worker named `nd40-website` already exists, stop and ask before overwriting it.
- If `[DOMAIN]` already has DNS records or is attached to something else, stop and report what is there. Do not replace it.

## Steps

### 1. Check access

From the `Nd40/` folder:

```sh
npx wrangler whoami
```

Confirm the account shown is the intended one. If several accounts are listed, set `CLOUDFLARE_ACCOUNT_ID` to the right one, or add `"account_id"` to `wrangler.jsonc`.

### 2. Dry run

```sh
npx wrangler deploy --dry-run
```

Expect it to read the assets from `./website`. The folder holds 23 files, and Wrangler 4.167 reported 25 in a test run of this exact config. If the count is very different, stop and report.

### 3. Deploy to workers.dev

```sh
npx wrangler deploy
```

Note the `https://nd40-website.<subdomain>.workers.dev` address it prints.

### 4. Verify on workers.dev

Check each of these against the workers.dev address:

| Request | Expected |
|---|---|
| `/` | 200, the home page with six cards |
| `/about`, `/work`, `/careers`, `/contact`, `/manta`, `/documentation`, `/request-access`, `/terms`, `/privacy`, `/security` | 200 each |
| `/about.html` | redirects to `/about` (normal for `auto-trailing-slash`) |
| `/assets/site.css` | 200, `text/css` |
| `/assets/fonts/oxanium.ttf` | 200 |
| `/favicon.ico` | 200 |
| `/does-not-exist` | 404, showing the "Page not found" page |

Then open the home page in a browser and confirm:

- The wordmark reads "Nd40" with an orange "40" whose zero has a dot in it.
- Text is in the Oxanium typeface (squared letterforms), not a default serif or sans.
- Clicking the logo from any subpage returns to the home page.
- Every footer link opens its page.

If any check fails, stop and report. Do not continue to the domain step.

### 5. Attach the production domain

Only after step 4 passes and James has given `[DOMAIN]`.

In `wrangler.jsonc`, add inside the top-level object:

```jsonc
"routes": [{ "pattern": "[DOMAIN]", "custom_domain": true }]
```

Then:

```sh
npx wrangler deploy
```

The zone for `[DOMAIN]` must already be on this Cloudflare account. Cloudflare creates the DNS record and certificate for a custom domain itself. If the deploy reports that a record already exists for that hostname, stop and report it.

If James also wants `www.[DOMAIN]`, add it as a second route entry and ask him which of the two should be the main address.

### 6. Verify on the domain

Repeat the table from step 4 against `https://[DOMAIN]`. Certificates can take a few minutes. Also confirm `http://[DOMAIN]` redirects to `https`.

## Report back

- The workers.dev URL and the production URL
- The Worker name and the deployment or version ID
- The result of each check in steps 4 and 6
- Anything you changed in `wrangler.jsonc`
- Anything you skipped, and why

## Known state of the site

These are expected and are not deployment errors:

- Subpage body text in [square brackets] is placeholder copy.
- The Terms, Privacy and Security pages have no real content yet.

## Rollback

```sh
npx wrangler rollback
```

This returns the Worker to its previous version. To take the site down entirely, remove the route from `wrangler.jsonc` and redeploy, or delete the Worker. Ask James before deleting anything.
