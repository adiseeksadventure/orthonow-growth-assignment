# OrthoNow — Namoza Developer Assignment

Submission for the Namoza *Developer – Position 1 (Client Web + Martech)*
assignment. Client scenario: **OrthoNow**, a chain of 9 orthopaedic clinics
across Bengaluru, Hyderabad, and Chennai.

## Deliverables

| Task | What it is | Where |
| --- | --- | --- |
| **01 — GTM Event Schema** | Full event-tracking spec: schema table, booking-funnel `dataLayer` JSON + drop-off tracking, and the one conversion to import into Google Ads. | [`task-01-gtm-event-schema/`](task-01-gtm-event-schema/README.md) |
| **02 — Landing Page** | Single self-contained HTML file for the *Book a Consultation* campaign. Fires the `consultation_form_submitted` dataLayer push on submit; shows a thank-you state with no reload. **Lighthouse mobile: 100/100.** | [`task-02-landing-page/`](task-02-landing-page/) |
| **03 — Integration Design** | Written architecture for landing page → HubSpot → Karix WhatsApp → Google Ads, incl. the phone-dedup problem, biggest failure point + fallback, and the 2-min SLA. | [`task-03-integration-design/`](task-03-integration-design/README.md) |

## Task 02 quick start

Open the file directly in a browser — no server, no build step:

```
open task-02-landing-page/index.html
```

**To see the tracking fire:** open DevTools → Console, type `dataLayer` and press
Enter, submit the form, then re-inspect `dataLayer` — the last entry is the
`consultation_form_submitted` push. It fires **only on a valid submit**, never on
page load. Try `?clinic=indiranagar` in the URL to see `clinic_preference` flow
through from the ad's landing variant.

Core Web Vitals evidence and how to capture the official PageSpeed Insights
screenshot: [`task-02-landing-page/PAGESPEED.md`](task-02-landing-page/PAGESPEED.md).
