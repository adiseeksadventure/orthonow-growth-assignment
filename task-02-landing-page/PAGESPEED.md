# Task 02 — Core Web Vitals

**Requirement:** 90+ on PageSpeed Insights (Mobile).

## Verified result — Lighthouse Mobile (this repo)

Audited locally with Lighthouse 12 (mobile form factor, `--form-factor=mobile`,
simulated throttling) against `index.html`:

| Metric | Result |
| --- | --- |
| **Performance score** | **100 / 100** |
| Largest Contentful Paint (LCP) | 0.9 s |
| First Contentful Paint (FCP) | 0.8 s |
| Total Blocking Time (TBT) | 0 ms |
| Cumulative Layout Shift (CLS) | 0 |
| Speed Index | 0.8 s |

The full report is committed alongside this file: **`lighthouse-mobile-report.html`**
(open it in a browser). Lighthouse is the same engine that powers PageSpeed
Insights, so the lab scores line up.

## Why it scores this high

- **Zero network requests beyond the HTML itself** — no web fonts (system font
  stack), no images (inline SVG + CSS), no framework, no analytics library on
  first paint. Nothing to download, parse, or block.
- **Single self-contained file** — CSS is inlined in `<head>`, JS is a small
  inline block at the end of `<body>`.
- **CLS = 0** — fixed layout, no late-loading media pushing content around.
- **TBT = 0 ms** — only a few KB of vanilla JS, no long tasks.

## Capturing the official PageSpeed Insights screenshot

PageSpeed Insights only audits a **public URL**, so this needs a one-time deploy:

1. Enable GitHub Pages on this repo (Settings → Pages → deploy from branch),
   or drop `index.html` on Netlify/Vercel/Cloudflare Pages.
2. Open <https://pagespeed.web.dev/> and run the deployed URL.
3. Screenshot the **Mobile** tab and save it here as `pagespeed-mobile.png`.

> Screenshot placeholder: add `pagespeed-mobile.png` after deploying.
