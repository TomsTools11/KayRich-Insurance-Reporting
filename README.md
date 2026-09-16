# KayRich Insurance Reporting

Static reporting hub for KayRich Insurance Agency, deployed on Vercel.

**Live site:** the root URL (`/`) is the report hub, with one card per report. Each report has an "← All Reports" button at the top of its sidebar that returns to the hub (hidden when printed).

## Repository layout

```
public/            # everything in here is deployed and publicly reachable
  index.html       # report hub (served at /), one card per report
  reports/
    campaign-configuration-2026-09-15.html   # /reports/campaign-configuration-2026-09-15
    seo-audit-2026-09-16.html                # /reports/seo-audit-2026-09-16
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

Every page is fully self-contained: all imagery is embedded as base64 `data:` URIs and the only inline script is an `IntersectionObserver` that animates the bar widths. The single external dependency is the Inter webfont from Google Fonts, which degrades gracefully to a system font stack if it fails to load.

## Ad preview section

Section 02 reproduces the GOAL ad preview in HTML/CSS rather than embedding a screenshot, so it stays crisp at any zoom, reflows on phones, and prints cleanly.

The agency wordmark is rendered in CSS using the Parisienne webfont, because the logo files are still outstanding (item 5 in Items to confirm). When the real logo arrives, replace the whole `<div class="adlogo">` block in `public/reports/campaign-configuration-2026-09-15.html` with one line:

```html
<img class="adlogo" src="data:image/png;base64,PASTE" alt="KayRich Insurance Agency">
```

The surrounding `.adcard` grid already sizes that column at 232px, so nothing else needs to change. An inline comment in the markup says the same thing. Keep the logo as a `data:` URI rather than a separate file — the CSP allows `img-src 'self' data:` and the report stays self-contained.

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

`vercel.json` redirects the older links to the files under `reports/`: `/seo-audit`, `/seo-audit-2026-09-16` and `/campaign-configuration`.

## Access note

A Vercel deployment is reachable by anyone who has the URL. `robots.txt` and `X-Robots-Tag` keep it out of search results, but they do not restrict access. If this report should be restricted, enable Vercel's Deployment Protection (Password Protection or Vercel Authentication) in **Project Settings → Deployment Protection**.

## Local preview

```bash
npx serve public
# or
python3 -m http.server -d public 3000
```
