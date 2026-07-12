# Ringo Fresh — Backend / Google Apps Script Build Brief

**Goal:** Build a Google Apps Script Web App attached to the Ringo Fresh Google Sheet that (1) serves live product/price data to the website as JSON, and (2) receives order form submissions and triggers notifications.

**Source Sheet:** Google Sheet titled "OYSTERBAY APPLE STOCK FILE" — contains existing operational ledger tabs (do not touch these) plus two tabs built specifically for this project:
- `Website price list - oya`
- `Website price list - kko`

*(Confirm exact tab name spelling/capitalisation before building, since Apps Script references tab names literally.)*

Each of these two tabs has columns: `Product | Category | Unit | Price (TZS) | Photo Link (Google Drive) | Active (Yes/No)`

---

## Part 1 — Live Price Feed (Sheet → Website)

Build an Apps Script Web App with a GET endpoint that:
- Reads both `Website price list - oya` and `Website price list - kko` tabs
- Filters out any row where Active = "No"
- Converts each Photo Link (Google Drive share link) into a direct-display image URL — Drive share links don't render as `<img>` sources as-is, so extract the file ID from the link and build a direct-display URL format from it (e.g. `https://drive.google.com/uc?export=view&id=FILE_ID`, or an equivalent stable direct-view format)
- Returns JSON structured like:

```json
{
  "lastUpdated": "2026-07-13T09:15:00+03:00",
  "locations": {
    "oysterbay": [
      { "product": "Cripps Red 135", "category": "Apples", "unit": "box", "price": 95000, "photo": "https://..." }
    ],
    "kko": [ ... ]
  }
}
```

- `lastUpdated` should reflect either the Sheet's last-edit timestamp or the time the request was served — this powers the "Prices last updated" label on the website
- Deploy as "Execute as: Me" / "Who has access: Anyone" so the website can fetch it without requiring the visitor to log in

---

## Part 2 — Order Form (Website → Sheet + Email + WhatsApp)

Build a second endpoint (or a second action on the same Web App) that accepts POST requests from the website's order form with this payload:

```json
{
  "customerName": "",
  "businessName": "",
  "phone": "",
  "email": "",
  "deliveryAddress": "",
  "location": "oysterbay | kko",
  "items": [ { "product": "", "quantity": 0 } ],
  "deliveryDate": "",
  "notes": ""
}
```

On receiving a submission, the script should:

1. Append a new row to an `ORDERS` tab (create this tab if it doesn't exist) with all fields plus an auto-generated date and time of submission
2. Send an email notification to the Ringo Fresh team inbox (use `MailApp.sendEmail` or `GmailApp.sendEmail`) summarising the order
3. Trigger a WhatsApp notification to both business numbers:
   - +255 795 471 471
   - +255 768 414 342

   Apps Script cannot send WhatsApp natively, so route this through one of:
   - **Zapier or Make.com** (recommended starting point) — the script POSTs to a Zapier/Make webhook, which sends the WhatsApp message. No-code, fast to set up, has a free tier for low volume.
   - **Twilio WhatsApp API** — call Twilio's API directly via `UrlFetchApp`. More reliable/scalable long-term, but requires a Twilio account, WhatsApp Business API approval, and per-message cost. A good next step once order volume grows.

---

## Part 3 — Instagram Export Feed (internal, password-gated)

- Reuse the Part 1 JSON feed as the data source
- The website front-end generates the 1080×1080 (post) and 1080×1920 (story) images client-side from that feed, styled with the Ringo Fresh logo, current date, and product/price grid — no separate backend endpoint needed
- Gate this page behind a simple password on the front-end; do not link it anywhere public-facing or let it be indexed by search engines

---

## Part 4 — Customer PDF Download

- Also generated client-side (or via the same feed) from live data on each request, including logo, page title, generation date, and timestamp — no separate backend step needed

---

## Deliverables

1. The Apps Script code, deployed as a Web App, with the deployment URL
2. Documentation of the two endpoint URLs (price feed GET, order submission POST) to hand to whoever builds the front-end in Claude Design
3. Confirmation of which WhatsApp routing method was used (Zapier/Make or Twilio) and setup steps for whichever was chosen

---

## Notes
- Minimum order quantities are not finalised yet — build the Sheet structure and order validation so an optional per-product MOQ column can be added later without breaking the feed.
- Both stock locations (Oysterbay, KKO) must stay clearly separated throughout — in the JSON feed, in the Orders tab (via the `location` field), and in any WhatsApp/email notification text, so the team always knows which location an order applies to.
