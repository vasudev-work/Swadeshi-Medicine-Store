# Swadeshi Medicine Store

One static web page that lists Indian generic medicines with the **Jan Aushadhi (PMBJP) MRP**, and turns a search for a foreign brand into the Indian generic that matches it.

- Live page: `index.html` (self-contained — HTML, CSS and JS in one file)
- Legal page: `legal.html`
- 2,026 products, 59 therapeutic groups, 25 brand mappings

No build step, no framework, no CDN, no backend, no analytics, no cookies.

---

## Where the data comes from

Every product on the page comes from the **Jan Aushadhi medicine list (generic name wise), product list updated 13.01.2023** — the official PMBJP price list. Each row in that PDF gives a serial number, a drug code, the generic name, the unit size, the MRP and the therapeutic group.

Those six columns are what the page shows. The source list does **not** contain manufacturer names, stock, expiry, batch numbers or product photographs, so the page leaves them blank or says "not listed". Nothing is filled in by guesswork.

Prices are the printed MRP from that list. They can change, and availability differs between Jan Aushadhi Kendras.

## Layout

```
index.html            the whole store — markup, tokens, styles and app logic inline
legal.html            privacy, terms, refunds, cookies, accessibility, security, disclosure
.nojekyll             keeps GitHub Pages from running Jekyll over the files
LICENSE               MIT
README.md             this file
```

Inside `index.html`:

```
<style>               design tokens + all layout, mobile first, laptop at 900px
<body>                top bar, side nav / bottom bar, the five views, the sheets
<script> const PRODUCTS = [...]   the product list, embedded as JSON
<script>              app logic (render, search, sheets, mailto, localStorage)
```

## Views

Five destinations, in the bottom bar on a phone and the left sidebar at 900px and up.

| View | What it does |
|---|---|
| Jan Aushadhi | Category chips, two promo cards, the "Indian generics for common brands" carousel, and a suggested list |
| Medicines | The full list, 60 rows at a time with a **Show more** button, filtered by chip, therapeutic group or dosage form |
| Search | Brand-to-generic search with alternatives (below) |
| Prescriptions | Type the names a doctor wrote, one per line, and get the generic and its listed price for each |
| You | Saved list, theme and accent, store state, operator card, contribute links, about, Make in India bar |

## How the search works

The search is the point of the page, so it never answers a foreign brand with "no results".

1. A mapping table sits in the file. Each entry looks like:

   ```js
   { brand: 'Crocin', keys: ['crocin', 'crosin', 'crocin advance'],
     owner: 'GSK plc', country: 'United Kingdom',
     category: 'Analgesic/Antipyretic/Anti-Inflammatory',
     primary: 1431, others: [1432, 1427] }
   ```

   `primary` and `others` are product serial numbers from the embedded list, so an alternative is always a real listing with a real price.

2. **Matching rules**
   - An exact key match wins outright: score `1000 + key length`.
   - Contains / starts-with only counts when the typed text is at least **4 characters**; the score is then the key length.
   - If several entries match, the highest score wins.

3. **What a matched search shows**
   - `N Indian options found`
   - A green banner: **TRY AN INDIAN GENERIC** — what you typed, the Indian generic to try, its pack and price, the therapeutic category, and who owns the brand.
   - **Indian alternatives** — rows for the mapped products.
   - **Other matches** — every other listing whose name contains what you typed.

4. **Known brand, no mapping** — brands on the foreign-brand list but without a mapping entry still get the banner, with their category and a plain statement that the mapping is not added yet. The page says that instead of pretending.

5. **A generic name** — no banner, just matches from the list, so `metformin` returns 54 listings.

6. All of it runs on the device against the embedded JSON. No query is sent anywhere.

## Detail sheet

Any card or row opens a sheet with the name, a **Made in India** badge, manufacturer (or "not listed in the source list"), pack size, category, dosage form, MRP, product code, a one-line description and the source page of the PDF.

When the item is reached through a foreign-owned brand, an amber box appears: **Foreign-owned brand**, who owns it and the country. The product stays listed — the box is information, not a removal.

Buttons: **Jan Aushadhi price**, **Compare prices**, **Save**, **Report this listing**, **Close**.

## Save, report, submit

- **Save** toggles a star and writes the serial number to `localStorage` under `swadeshi.v1`. The **You** view lists saved items with a **Clear saved** button. Nothing leaves the device.
- **Report this listing** opens a sheet with five reasons as radio buttons plus a details box, and builds a `mailto:` link:

  ```
  Subject: Listing report: <product name>

  Listing report from Swadeshi Medicine Store

  Product: <name>
  Manufacturer: <name>
  Type: REPORT
  Reason: <reason>
  Details: <note>

  --- machine readable ---
  {"user_id":"…","product_id":"…","product_name":"…","manufacturer":"…",
   "report_type":"REPORT","reason":"…","description":"…","timestamp":"…"}
  ```

- **Add a medicine** collects name, manufacturer, pack size, category, link and why it should be listed, and sends `Subject: Listing submission for Swadeshi Medicine Store` with the same key/value body shape.

Both are plain email. There is no form endpoint, no server and no account.

## Themes and accessibility

- Dark by default, plus Light and Black; accent in green, saffron or indigo. All three are CSS variables on `<html>`, applied live and remembered in `localStorage`.
- Real `<button>` and `<a>` elements, visible focus rings, `aria-label` on icon-only buttons, 44px tap targets, dialogs with `aria-modal` and Escape to close, `prefers-reduced-motion` respected.
- No emoji used as icons, no pill-shaped primary buttons, no purple gradients, no stock photography. A missing image becomes a coloured letter tile derived from the product name.

## What this page cannot do

- **It cannot read a prescription photo.** There is no OCR and no upload. Reading an image only works in the mobile app; the web page says so where a user would expect it.
- **It cannot check stock.** No pharmacy inventory is connected.
- **It cannot take payments.** No cart, no checkout, no prices other than the published MRP.
- **It gives no medical advice.** Substitution and dosage are for a doctor or pharmacist. Every view carries "Information only. Always follow your doctor's prescription."
- **It cannot show a brand's retail price.** The compare sheet shows the Jan Aushadhi MRP and says the brand MRP is not listed rather than inventing a number.
- **It cannot show manufacturer or expiry.** Not in the source list.

## Running it

Open `index.html` in a browser. That is the whole setup. For a local server:

```
python3 -m http.server 8000
```

## Deploying

GitHub Pages serves the repository root directly; `.nojekyll` stops Jekyll from touching the files. Settings → Pages → Source: *Deploy from a branch* → `main` / `/ (root)`.

## Licence

MIT. The underlying price data belongs to the Pharmaceuticals & Medical Devices Bureau of India (PMBJP) and is reproduced from its public list. Brand names are trademarks of their respective owners and are used only to point to the generic equivalent.
