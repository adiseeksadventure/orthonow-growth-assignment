## Task 03 – Integration Design

When the consultation form is submitted, I would keep the submission on the client side but trigger two actions simultaneously. First, I'd push a consultation_form_submitted event to the dataLayer so that Google Tag Manager can immediately fire the Google Ads conversion. Second, I'd send the form data to a serverless endpoint (such as a Cloudflare Worker or Vercel Function), which acts as the backend for the integration.

Instead of directly calling HubSpot and Karix from this endpoint, I'd first store the lead in a queue (Cloudflare Queues or AWS SQS) and return a success response to the user immediately. This ensures the user sees the thank-you message without waiting for external services.

A queue consumer would then process the lead. For HubSpot, I'd use the CRM API directly instead of the native form embed, Zapier, or Make. The direct API gives me better control over custom logic, lower latency, and avoids relying on third-party automation platforms for something that's part of the core lead flow.

One important issue is HubSpot's default deduplication. It only deduplicates using email, but this form only collects a phone number. Before creating a contact, I'd normalize the phone number to E.164 format and search HubSpot for an existing contact with that phone number. If one exists, I'd update it; otherwise I'd create a new contact.

There's an edge case where two different people may share the same phone number, which is fairly common in healthcare. In that case, I'd treat the phone number as the primary identifier, keep the latest name as the active value, store the previous name in a custom name_history property, and flag the contact for manual review instead of overwriting information silently.

The biggest risk is losing a lead if HubSpot or Karix is unavailable during submission. Persisting the lead in a queue before making any downstream API calls solves this problem. Failed requests can be retried with exponential backoff, and after repeated failures moved to a dead-letter queue for manual investigation.

To meet the WhatsApp two-minute SLA, I'd monitor the time between lead_created_at and whatsapp_sent_at. Alerts would be triggered if this exceeds two minutes or if Karix returns an error. I'd also log delivery webhooks and run a scheduled synthetic test lead every 15 minutes so integration issues are detected before they impact real patients.