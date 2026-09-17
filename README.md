# KayRich Insurance Reporting

Static reporting hub for KayRich Insurance Agency, deployed on Vercel.

**Live site:** the root URL (`/`) is the report hub, with one card per report. Each report has an "← All Reports" button at the top of its sidebar that returns to the hub (hidden when printed).

## Repository layout

```
public/            # everything in here is deployed and publicly reachable
  index.html       # report hub (served at /), one card per report
  reports/
    campaign-configuration-2026-09-15.html              # /reports/campaign-configuration-2026-09-15 (TX Home)
    commercial-campaign-configuration-2026-09-16.html   # /reports/commercial-campaign-configuration-2026-09-16 (TX Commercial)
    seo-audit-2026-09-16.html                           # /reports/seo-audit-2026-09-16
  assets/
    tx-map-home.svg                                     # county map for the Home report (243 targeted / 11 excluded)
    tx-map-commercial.svg                               # county map for the Commercial report (all 254 targeted)
  robots.txt       # asks crawlers not to index the reports
vercel.json        # Vercel build & header configuration
```

Files outside `public/` (this README, `.gitignore`, `vercel.json`) are **not** served — `outputDirectory` is pinned to `public`.

## Vercel project settings

This is a zero-build static site. When importing the repo into Vercel, use:

| Setting | Value |
| --- | --- |
| Framework Preset | **Other** |
| Build Command | *(leave empty / override off)* |
| Output Directory | `public` (already set by `vercel.json`) |
| Install Command | *(leave empty)* |
| Root Directory | `./` |
| Node.js Version | *(irrelevant — nothing is built)* |

No environment variables are required. There is no build step, no package manager, and no server-side code, so deploys are effectively instant.

## What `vercel.json` does

- `outputDirectory: "public"` — serves only the `public/` folder, so repo files are never exposed.
- `cleanUrls: true` — `public/foo.html` is served at `/foo` (and `/foo.html` redirects to `/foo`).
- `trailingSlash: false` — one canonical URL per page.
- `Cache-Control: public, max-age=0, must-revalidate` — browsers revalidate on every load, so an updated report is picked up immediately instead of being served stale from cache. Everything here is a small HTML document, so there is no long-lived asset cache to preserve.
- `X-Robots-Tag: noindex, nofollow` — belt-and-braces alongside `robots.txt`, since a `vercel.app` URL is publicly reachable by anyone who has the link.
- `Content-Security-Policy` — scoped to exactly what the report uses: inline `<style>`/`<script>`, `data:` images, and Google Fonts (`fonts.googleapis.com` / `fonts.gstatic.com`). Adding an outside script, stylesheet, iframe, or XHR call to the report means widening this policy or the browser will block it.
- `Strict-Transport-Security`, `X-Content-Type-Options`, `Referrer-Policy` — standard hardening.

## The report itself

Every page is self-contained apart from its county map: logos are embedded as base64 `data:` URIs and the only inline script is an `IntersectionObserver` that animates the bar widths. The two campaign reports load their Texas county map from `public/assets/` as a static `<img src="../assets/tx-map-*.svg">` (the SVG carries its own `<style>`), which keeps each report page small; the map only changes when the geography does. The single external dependency is the Inter webfont from Google Fonts, which degrades gracefully to a system font stack if it fails to load.

## Ad preview section

Section 02 reproduces the GOAL ad preview in HTML/CSS rather than embedding a screenshot, so it stays crisp at any zoom, reflows on phones, and prints cleanly.

The ad card's classes use a `cr` (creative) prefix — `.crwrap`, `.crcard`, `.crlogo`, `.crhead` and so on. Do not rename them to `ad*`: EasyList hides `.adwrap`, `.adcard`, `.adhead`, `.adlist` and `.adtop`, so browsers with a built-in or extension ad blocker (Helium, Brave, uBlock Origin) silently remove the whole preview.

The agency logo is embedded as a base64 `data:` URI (`<img class="crlogo" …>`, 203×142 px, cropped from the GOAL ad preview on 2026-09-16). Both campaign reports carry the same logo. Keep it as a `data:` URI rather than a separate file — the CSP allows `img-src 'self' data:` and each report stays self-contained. The Parisienne webfont import is no longer used by the ad card and can be dropped from the `<link>` tag if the reports are rebuilt.

## Adding future reports

1. Put the report in `public/reports/` with a URL-friendly, dated name, e.g. `public/reports/campaign-performance-2026-10-15.html`. `cleanUrls` serves it at `/reports/campaign-performance-2026-10-15`.
2. Make its first sidebar element the back link, and hide it in print (`.side .back{display:none;}` inside `@media print`):

   ```html
   <a class="back" href="../index.html">&larr; All Reports</a>
   ```

3. Add a card for it to the matching group in `public/index.html` (or add a new group):

   ```html
   <a class="card" href="reports/campaign-performance-2026-10-15.html"><div class="ct">Title</div><div class="cs">Scope &middot; Date</div><div class="cl">Open report &rarr;</div></a>
   ```

## Website redesign demo card

The "Website & SEO" group also has a **Website Redesign Demo** card. It links to an external site, https://kayrich-site-demo.vercel.app, deployed from the [`TomsTools11/kayrich-site-demo`](https://github.com/TomsTools11/kayrich-site-demo) repo. It is the only card that opens in a new tab (`target="_blank" rel="noopener noreferrer"`); report cards stay in the same tab so the "← All Reports" back link works. If the demo's Vercel project is renamed or gets a custom domain, update the card's `href`. No CSP change is needed, because the policy doesn't restrict link navigation.

`vercel.json` redirects the short links to the files under `reports/`: `/seo-audit`, `/seo-audit-2026-09-16`, `/campaign-configuration` (TX Home) and `/commercial-campaign-configuration` (TX Commercial).

## Access note

A Vercel deployment is reachable by anyone who has the URL. `robots.txt` and `X-Robots-Tag` keep it out of search results, but they do not restrict access. If this report should be restricted, enable Vercel's Deployment Protection (Password Protection or Vercel Authentication) in **Project Settings → Deployment Protection**.

## Local preview

```bash
npx serve public
# or
python3 -m http.server -d public 3000
```
