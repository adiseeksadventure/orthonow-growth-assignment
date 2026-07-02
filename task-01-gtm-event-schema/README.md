# Task 01 — GTM Event Schema for OrthoNow

Full event-tracking spec to implement in Google Tag Manager before any paid
campaigns go live. Every event below is a **custom `dataLayer` event** unless the
Trigger Type says otherwise — that distinction is the whole point (see the
booking-funnel section).

## Naming conventions

- Events and parameters use `snake_case` (GA4 requirement).
- Every event carries `clinic_location` where a clinic context exists, so any
  event can be broken down by clinic in GA4.
- GA4 key events (conversions) are marked ✅.

---

## 1. Complete event schema

| # | Event Name | Trigger Type (GTM) | Key Parameters (≥3) | Feeds into (GA4 report / audience) |
|---|------------|--------------------|---------------------|------------------------------------|
| 1 | `booking_start` | Custom Event — `booking_start` (fires when step-1 UI renders) | `clinic_location`, `specialty`, `entry_source` (organic/ads/direct) | Funnel Exploration (entry step); audience: *Started booking, didn't finish* |
| 2 | `booking_step_complete` | Custom Event — `booking_step_complete` (front-end pushes at each step) | `step_number` (1–3), `step_name`, `clinic_location`, `specialty` | Funnel Exploration (step-by-step drop-off) |
| 3 | `booking_confirmed` ✅ | Custom Event — `booking_confirmed` | `clinic_location`, `specialty`, `preferred_date`, `booking_id`, `value` | Key event → GA4 Conversions report; **imported to Google Ads** |
| 4 | `call_now_click` ✅ | Click — *Click Element* matches `a[href^="tel:"]` (or `.data-event=call_now`) | `clinic_location`, `page_type` (home/clinic/landing), `phone_number` | Key event; audience: *High-intent callers* |
| 5 | `whatsapp_click` | Click — *Click Element* matches `a[href*="wa.me"]` | `clinic_location`, `page_type`, `link_url` | Engagement report; audience: *WhatsApp enquirers* |
| 6 | `patient_guide_download` ✅ | Custom Event — `patient_guide_download` (fires after the gate form validates) | `guide_name`, `clinic_location`, `lead_source`, `value` | Key event (lead); audience: *Guide leads → remarketing* |
| 7 | `clinic_page_view` | Custom Event — `clinic_page_view` (or History Change + Page Path regex `^/clinics/`) | `clinic_location`, `clinic_city`, `page_path` | Landing-page / location report; audience: *Viewed clinic X* (per-clinic remarketing) |
| 8 | `blog_scroll` | Scroll Depth — vertical, thresholds 25/50/75/90% | `percent_scrolled`, `article_title`, `article_category` | Content engagement; audience: *Engaged readers (≥75%)* |
| 9 | `consultation_form_submitted` ✅ | Custom Event — pushed by the landing page (see Task 02) | `form_id`, `lead_source`, `clinic_preference`, `campaign` | Key event; the LP campaign's optimisation signal |

> **Why `call_now_click` is a Click trigger but the booking steps are Custom
> Events:** GTM can natively "see" a click on a `tel:` link or a `wa.me` link in
> the DOM. It **cannot** see a user moving between steps of a JS-driven form —
> there is no page load and no unique element click that reliably means "step 2
> is done." That state lives inside the front-end app, so the front-end dev must
> **push it to the dataLayer explicitly**. That is the next section.

---

## 2. Booking funnel — step-level drop-off tracking

The 3-step form is a single-page component: no page reloads, no URL changes.
GTM has zero visibility into it by default. To measure drop-off we instrument
**one push per step transition**, fired by the front-end developer.

### The dataLayer pushes (actual JSON)

**On form open (funnel entry):**
```json
{
  "event": "booking_start",
  "clinic_location": "Indiranagar",
  "specialty": "Knee & Joint",
  "entry_source": "google_ads"
}
```

**Step 1 complete — location + specialty selected:**
```json
{
  "event": "booking_step_complete",
  "step_number": 1,
  "step_name": "location_specialty_selected",
  "clinic_location": "Indiranagar",
  "specialty": "Knee & Joint"
}
```

**Step 2 complete — name / phone / preferred date entered:**
```json
{
  "event": "booking_step_complete",
  "step_number": 2,
  "step_name": "patient_details_entered",
  "clinic_location": "Indiranagar",
  "specialty": "Knee & Joint",
  "preferred_date": "2026-07-08"
}
```

**Step 3 complete — booking confirmed (also the conversion):**
```json
{
  "event": "booking_confirmed",
  "step_number": 3,
  "step_name": "booking_confirmed",
  "clinic_location": "Indiranagar",
  "specialty": "Knee & Joint",
  "preferred_date": "2026-07-08",
  "booking_id": "BLR-2026-004871",
  "value": 500
}
```

> Note: step 3 is emitted as its own event name (`booking_confirmed`) **and**
> carries `step_number: 3`, so it can act as the final funnel step *and* be
> registered as a clean GA4 key event without extra trigger logic.

### GTM triggers

| Trigger | Type | Fires on |
|---------|------|----------|
| `CE - booking_start` | Custom Event | `event equals booking_start` |
| `CE - booking_step_1` | Custom Event | `event equals booking_step_complete` **AND** `step_number equals 1` |
| `CE - booking_step_2` | Custom Event | `event equals booking_step_complete` **AND** `step_number equals 2` |
| `CE - booking_confirmed` | Custom Event | `event equals booking_confirmed` |

Each trigger fires one **GA4 Event tag** that maps the dataLayer variables
(`step_number`, `step_name`, `clinic_location`, `specialty`, …) to event
parameters via Data Layer Variables. Register `step_number`, `step_name`,
`clinic_location`, and `specialty` as **custom dimensions** in GA4 so they are
usable in Explorations.

### Surfacing drop-off in GA4 Funnel Exploration

Build an **open funnel** in Explore with these ordered steps:

1. `booking_start`
2. `booking_step_complete` where `step_number = 1`
3. `booking_step_complete` where `step_number = 2`
4. `booking_confirmed`

GA4 renders **completion vs. abandonment between each step**. Add
`clinic_location` (or `specialty`) as the **breakdown dimension** to see, e.g.,
that Whitefield leaks 40% of users at step 2 (the phone-number step) while
Koramangala leaks at step 1 — which points marketing and product at the exact
step and clinic to fix.

### Who writes the push?

**The front-end developer writes the dataLayer push; I (martech/dev) spec it.**
GTM cannot instrument multi-step form state on its own. My brief to the dev team
for **step 2** would be:

> "In the step-2 `next` handler, *after* the name/phone/date fields pass
> validation and *before* you transition the UI to step 3, call
> `window.dataLayer.push({...})` with exactly this shape (event
> `booking_step_complete`, `step_number: 2`, `step_name:
> 'patient_details_entered'`, plus `clinic_location`, `specialty`,
> `preferred_date` from component state). Fire it once per successful advance —
> not on validation errors, not on back-navigation. Keys and casing must match
> this spec exactly, because GTM triggers match on literal values."

---

## 3. The one conversion to import into Google Ads

**Import `booking_confirmed`.**

- It is the **actual business outcome** — a booked appointment, bottom of the
  funnel and closest to revenue. Google Ads' Smart Bidding optimises toward the
  action you feed it, so feeding it the completed booking teaches the algorithm
  to chase patients who *book*, not merely those who start.
- **Why not the alternatives:** `booking_start` and `booking_step_complete` are
  too top-of-funnel — optimising to them buys cheap starts that never convert.
  `call_now_click` is high-intent but **unverified** (a click isn't a booked or
  attended appointment, and it double-counts misdials), so it's better as a
  secondary/observation conversion. `booking_confirmed` carries a `value`, which
  also unlocks value-based bidding (tROAS) later.
- For the landing-page campaign specifically, `consultation_form_submitted`
  (Task 02) is imported as *that* campaign's conversion — a form lead is its real
  outcome — but for the site's core booking flow, `booking_confirmed` is the one.
