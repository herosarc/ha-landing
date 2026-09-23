# ha-landing

Hero's ARC production front door. Static site, locked StoryBrand house copy, deployed to GitHub Pages.

- **Live URL:** https://herosarc.github.io/ha-landing/
- **Copy source of truth:** [`HOUSE-COPY.md`](./HOUSE-COPY.md) (locked, Flow OK 2026-09-22). Casing is "Hero's ARC". Customer is the hero, Hero's ARC is the guide. Mind the kill list: no occupancy/MHV, no $500 trades bot, no naming the engine, no fake stats, and no em dashes.
- **Not** the old herosarc.ai AI phone / trades concierge / Okara-era story.

## Structure

```
site/            The deployed site (plain HTML/CSS, no build step)
  index.html     Front door: H1, sections 01 to 08, CTAs
  call/          "The Call" endpoint page. Swap this when the phone line is live.
  styles.css     Warm, plain, mobile-first styles
  404.html       Fallback page
HOUSE-COPY.md    Locked copy doc
```

## CTAs and how to wire them later

- **Primary, The Call:** every "The Call" button points at `site/call/`. There is no phone endpoint yet, so that page collects the call by email (`chris@herosarc.ai`, subject "The Call"). When a phone number or booking link exists, edit `site/call/index.html` once and every button on the site is wired.
- **Secondary, Meet your first power:** anchors to the First Power section, which soft-sells build-run packages (one job plus one agent, Chief of Staff) and links the free Chief walkthrough at [Rock the Grok](https://rock-the-grok.surge.sh) with a [github.io mirror](https://herosarc.github.io/rock-the-grok/) for when the sibling agent ships it.
- **Intake:** secondary text path via `mailto:chris@herosarc.ai?subject=Intake` in section 08 and the footer.

## Deploying

Push to `main`. The workflow in `.github/workflows/pages.yml` uploads `site/` and deploys it to GitHub Pages (it also enables Pages on first run). Local preview:

```bash
cd site && python3 -m http.server 8080
```

Note: the site is served under the `/ha-landing/` subpath on github.io, so internal links are relative. If you later serve it at a domain root, the relative links keep working; only `404.html` hardcodes the `/ha-landing/` prefix and would need updating.

## Pointing herosarc.ai at this site later (Cloudflare)

Do not do this yet. herosarc.ai currently serves the Lovable site, and changing DNS will take it down. Get Chris's go-ahead first.

When ready:

1. **GitHub side:** repo Settings, Pages, Custom domain: enter `herosarc.ai` and save. This commits a `CNAME` file; since we deploy via Actions, instead add a `site/CNAME` file containing `herosarc.ai` so it survives deploys. Keep "Enforce HTTPS" checked once the cert is issued.
2. **Cloudflare side (herosarc.ai zone):**
   - Apex `herosarc.ai`: CNAME to `herosarc.github.io` (Cloudflare flattens CNAME at the apex automatically). Alternatively, four A records: `185.199.108.153`, `185.199.109.153`, `185.199.110.153`, `185.199.111.153`.
   - `www`: CNAME to `herosarc.github.io`.
   - Set both records to **DNS only (grey cloud)** at first so GitHub can issue the TLS certificate. You can turn the proxy (orange cloud) back on after "Enforce HTTPS" is active, but grey cloud is the simpler steady state.
   - Remove or rename the existing records that point at Lovable (do not delete them until the new site is confirmed live; note their values somewhere first).
3. **Verify:** `dig herosarc.ai +short` shows the GitHub Pages IPs, `https://herosarc.ai` returns the front door with a valid cert, then update the 404 page prefix as noted above.

Once the custom domain is live, github.io URLs 301 to it, so nothing shared earlier breaks.
