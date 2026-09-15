# KayRich Insurance Reporting

Static hosting for the KayRich Insurance Agency campaign configuration report, deployed on Vercel.

**Live report:** the root URL of the deployment (`/`).

## Repository layout

```
public/            # everything in here is deployed and publicly reachable
  index.html       # the campaign configuration report (served at /)
  robots.txt       # asks crawlers not to index the report
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

`public/index.html` is fully self-contained: all imagery is embedded as base64 `data:` URIs and the only inline script is an `IntersectionObserver` that animates the bar widths. The single external dependency is the Inter webfont from Google Fonts, which degrades gracefully to a system font stack if it fails to load.

## Adding future reports

Drop the new file into `public/` with a URL-friendly, dated name — spaces and punctuation in filenames make for ugly, easily-broken links:

```
public/campaign-configuration-2026-10-15.html   ->   /campaign-configuration-2026-10-15
```

`cleanUrls` gives it an extensionless URL automatically. To make a new report the default landing page, replace `public/index.html` with it and keep the previous one under its dated name.

## Access note

A Vercel deployment is reachable by anyone who has the URL. `robots.txt` and `X-Robots-Tag` keep it out of search results, but they do not restrict access. If this report should be restricted, enable Vercel's Deployment Protection (Password Protection or Vercel Authentication) in **Project Settings → Deployment Protection**.

## Local preview

```bash
npx serve public
# or
python3 -m http.server -d public 3000
```
