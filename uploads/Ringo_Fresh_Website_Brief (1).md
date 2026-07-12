# Ringo Fresh — Website Build Brief for Claude Design

**Company:** Ringo Fresh — wholesale fruit & vegetable supplier, Tanzania
**Parent/sister brands (footer only, logos to follow):** VeryBerry, Eat Well
**Reference for look & feel:** yorkshiredentalsuite.co.uk (modern, clean typography, confident whitespace, big imagery, bold short headlines — NOT the dental content, just the design language)

---

## 1. Brand & Design Direction

Pull the palette directly from the Ringo Fresh logo:

| Colour | Hex | Use |
|---|---|---|
| Navy | `#2A397D` | Primary text, headers, nav, buttons |
| Olive Green | `#8AA000` | Accent, "fresh" cues, tags, hover states |
| Red | `#CB2125` | Highlights, CTAs, price emphasis |
| Brown | `#76472C` | Secondary accent (earth/produce tone) |
| Warm off-white | `#FAF6EE` or similar | Background (same warm neutral feel as the reference site, not stark white) |

**Typography:** A modern, clean sans-serif — geometric but warm (think Inter, General Sans, or Satoshi). Big, confident headline sizes with generous line-height, similar weight contrast to the reference site (very bold headline / light body text). Avoid anything that reads as "corporate stock template."

**Layout language to borrow from the reference site:**
- Full-bleed hero image/video with a short, punchy headline and one clear CTA button
- Trust strip under the hero (e.g. "500+ tonnes supplied," "Trusted by hotels & retailers across Dar es Salaam," "Farm to warehouse in under 24 hours")
- Card grids with rounded corners and image-led tiles (used for produce categories, like their "treatments" grid)
- Numbered 3-step "how it works" section (perfect for explaining the ordering process)
- Large photography throughout — real photos of produce, warehouse, and team, not stock/illustration
- Generous whitespace, soft rounded corners, subtle shadows, mobile-first responsiveness
- Clear, sticky navigation bar with a prominent "Place an Order" / "Download Price List" button

**Tone of voice:** Confident, straightforward, trustworthy — a B2B wholesale supplier, not a boutique grocer. Emphasise reliability, freshness, quality control, and consistency of supply.

---

## 2. Recommended Site Structure — 6 Pages

| # | Page | Purpose |
|---|---|---|
| 1 | **Home** | First impression, what Ringo Fresh does, trust signals, links into every other page |
| 2 | **About Us** | Company story, mission, quality standards, facilities overview |
| 3 | **Gallery / Our Facilities** | Photos of produce, warehouse, cold storage, sorting/packing, delivery fleet |
| 4 | **Price List** | Live pricing table (Google Sheets synced), PDF download, internal Instagram export |
| 5 | **Place an Order** | Order form → Google Sheets + email + WhatsApp notification |
| 6 | **Contact Us** | Address, phone, WhatsApp, email, map, business hours |

Keep navigation to these 6 — enough to feel professional and complete without becoming a maintenance burden. (Home + About can later be trimmed to 5 if you want to fold Contact details into a Home footer instead, but a dedicated Contact page reads as more established for a wholesale B2B buyer.)

---

## 3. Page-by-Page Content Brief

### Home
- Hero: full-width photo of fresh produce or the warehouse floor, headline like *"Fresh Produce, Delivered Reliably"* or *"Tanzania's Trusted Wholesale Fruit & Vegetable Supplier"*, subtext, two buttons: **View Price List** / **Place an Order**
- Trust strip: key stats (years in business, tonnes/week, clients served, delivery radius)
- "What We Do" section: short intro + 3–4 category cards (Fruits, Vegetables, Bulk/Wholesale Orders, Delivery) each with a real photo
- "Why Ringo Fresh" section: quality control, cold-chain storage, consistent supply, competitive wholesale pricing
- Facilities teaser: 2–3 photos with a link through to the Gallery page
- CTA banner before footer: "Ready to order? Get today's prices" → Price List / Order buttons
- Footer: contact snippet, address, socials, and the VeryBerry / Eat Well logos (see Section 6)

### About Us
- Company story and mission
- What "wholesale" means for the customer (hotels, restaurants, retailers, institutions)
- Quality & sourcing standards
- Short facilities overview (with a link to the full Gallery)
- Team / leadership photo if available

### Gallery / Our Facilities
- Filterable photo grid: **Fruits | Vegetables | Warehouse & Cold Storage | Sorting & Packing | Delivery Fleet**
- All images should be lightbox-enabled (click to enlarge)
- Build this to pull from a Google Drive folder (see Section 4) so new photos can be added without touching code

### Price List
This is the most functionally important page — see Section 4 for the technical build notes. On the page itself:
- Auto-updating pricing table, grouped by category (Fruits / Vegetables), each row: product photo, name, unit (kg/crate/box), price
- Search/filter bar
- "Prices last updated: [auto date/time]" shown clearly under the page title
- **"Download Price List (PDF)"** button — customer-facing, includes logo, date, and timestamp
- A **hidden/internal-only** section (e.g. accessible via a private URL like `/pricelist/export` or unlocked with a simple passcode) offering:
  - **Download as Instagram Story image** (1080×1920)
  - **Download as Instagram Post image** (1080×1080)
  - Both auto-generated from the same live pricing + photos, so they never go out of date

### Place an Order
- Simple, clean order form (not a shopping cart — a request form)
- Fields:
  - Customer name
  - Business/company name
  - **Phone number (required)** — used for WhatsApp confirmation
  - Email address
  - Delivery address
  - Product selection: multi-select list pulling from the same live product/price data (so pricing is always accurate), with a quantity field per item
  - Preferred delivery date
  - Notes/special requests
- On submit: order is written to a Google Sheet, an email notification goes to Ringo Fresh's inbox, and a WhatsApp notification is triggered (see Section 4)
- Confirmation screen: "Thank you — your order request has been received. We'll confirm by WhatsApp/call shortly."

### Contact Us
- Address (large, clear — this is a priority per your brief), embedded map
- Phone / WhatsApp click-to-chat buttons for both numbers: +255 795 471 471 and +255 768 414 342
- Email
- Business hours
- Delivery coverage note: "We deliver across Tanzania"
- Short contact form as a backup to the order form (for general enquiries, not orders)

---

## 4. Data & Integration Architecture (read this before briefing Claude Design)

Claude Design builds the front-end beautifully, but three of your requirements are **live data / automation features**, which need a lightweight backend. The cleanest, lowest-cost way to do this (no server needed) is a **Google Apps Script Web App** bound to your Google Sheet. Here's the recommended architecture:

### a) Live pricing sync (Google Sheets → Website)
1. Maintain one Google Sheet as the master price list: columns for Product, Category, Unit, Price, Photo (Google Drive link), Active (Yes/No).
2. Publish the Sheet via **Apps Script Web App** (deployed as "Anyone can view") that returns the sheet as JSON — this is more reliable than "Publish to web as CSV" and updates instantly.
3. The website's Price List page fetches that JSON on page load (and can poll every few minutes) to render the table — always current.
4. Deleting a row (or setting Active = "No") in the Sheet removes it from the JSON feed, which automatically removes it from the site — no manual step needed.

### b) Product photos (Google Drive → Website)
1. Store all product photos in one Google Drive folder, named to match the product (e.g. `Tomatoes.jpg`).
2. Set sharing to "Anyone with the link can view."
3. Reference each photo in the Sheet using its Drive **file ID** (not the normal share link — share links don't render as images directly; the Apps Script feed should convert the ID into a direct-display URL format automatically).

### c) PDF download (customer-facing)
- Generated client-side (or via the same Apps Script) from the live JSON feed each time it's requested — so it's always current.
- Must include: Ringo Fresh logo, page title, generation date and time stamp, grouped product table.

### d) Instagram JPEG export (internal use only)
- Two fixed-size templates (1080×1080 post, 1080×1920 story) auto-populated from the same live pricing/photo feed.
- Keep this behind a non-indexed URL or simple password so customers don't stumble on it — it's a staff tool, not a public page.

### e) Order form (Website → Google Sheet + Email + WhatsApp)
1. Order form submits to an Apps Script Web App endpoint (separate from the pricing feed, or the same one with a different action).
2. The script:
   - Appends a new row to an "Orders" Google Sheet, auto-filling **date and time** of submission alongside the customer's details.
   - Sends an email notification to your team's inbox using Apps Script's built-in mail function — instant, free, no third party needed.
   - Triggers a **WhatsApp notification** to both business numbers (+255 795 471 471 and +255 768 414 342). Note: Apps Script cannot send WhatsApp messages natively — this step needs one of:
     - **WhatsApp Business API via a provider like Twilio** (most reliable, small per-message cost), triggered by the Apps Script when a new row is added, or
     - An automation tool like **Zapier or Make.com** watching the Google Sheet for new rows and pushing a WhatsApp message to both numbers — no code needed, quick to set up, has a free/low-cost tier.
   - Recommend starting with Zapier/Make for speed, and moving to Twilio direct-integration later if order volume grows.

**Summary for whoever builds this:** Claude Design should build the front-end to call a Google Apps Script Web App URL for (a) reading live prices/photos and (b) submitting orders. The Apps Script itself (a short script pasted into the Google Sheet's Extensions → Apps Script) is the missing piece that makes the "automatic" parts actually automatic — this can be built alongside the website using Claude Code, or by whoever manages your Google Workspace.

---

## 5. Footer & Brand Family
- Standard footer: navigation links, contact snippet, address, social icons, WhatsApp click-to-chat
- Below the main footer, a smaller "Part of the [Parent Company] family" strip with logo placeholders for **VeryBerry** and **Eat Well** (leave clearly marked placeholder boxes since logos are coming later — sized consistently, greyscale-safe so they can be dropped in without breaking layout)

---

## 6. Prompt Summary — What to Paste Into Claude Design

> Design a modern, clean, professional website for **Ringo Fresh**, a wholesale fruit and vegetable supplier based in Tanzania. Use the attached logo and its colour palette (navy #2A397D, olive green #8AA000, red #CB2125, brown #76472C) on a warm off-white background. Follow a design language similar to yorkshiredentalsuite.co.uk — bold short headlines, big real photography, rounded card grids, generous whitespace, a numbered "how it works" section, and a confident modern sans-serif typeface.
>
> Build 6 pages: **Home, About Us, Gallery/Our Facilities, Price List, Place an Order, Contact Us.**
>
> The Price List page must pull live data (product name, category, unit, price, photo) from a JSON feed, show a "last updated" timestamp, offer a PDF download (with logo, date, timestamp) for customers, and a hidden internal section to export the current price list as a 1080×1080 Instagram post image and a 1080×1920 Instagram story image.
>
> The Place an Order page is a form (not a cart) capturing name, business name, phone number (required), email, delivery address, product selection with quantities pulled from the live price list, delivery date, and notes — submitting to a backend endpoint.
>
> Footer includes contact details, address, social/WhatsApp links, and a placeholder strip for two partner brand logos (VeryBerry, Eat Well).

---

## Notes / Confirmed Decisions
- **Minimum order quantities:** not decided yet — to be shared later. Build the order form so MOQ rules can be added per-product without a redesign (e.g. a quantity validation step that reads an optional "MOQ" column from the Sheet, defaulting to none until populated).
- **Instagram export tool:** sits behind a **password**, not just an unlisted URL. Add a simple password gate (single shared staff password is fine — no need for full user accounts) in front of the `/pricelist/export` page.
- **WhatsApp numbers:** two numbers should be supported —
  - +255 795 471 471
  - +255 768 414 342

  Use both for the click-to-chat buttons (offer as two options, or a single button defaulting to the first number) and for order notifications (send the new-order alert to both numbers).
- **Delivery area:** Tanzania-wide ("We deliver across Tanzania"). Display this plainly on the Contact page and Home trust strip rather than listing specific regions/cities.
