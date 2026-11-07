# Swadeshi Medicine Store

One static web page that lists Indian generic medicines with the **Jan Aushadhi (PMBJP) MRP**, and turns a search for a foreign brand into the Indian generic that matches it.

- Live page: `index.html` (self-contained — HTML, CSS and JS in one file)
- Legal page: `legal.html`
- 2,026 medicines in 59 therapeutic groups, plus 90 skin, hair and oral care products in 15 groups, plus 13 protein products
- Branded market prices for 60 packs, from the Jan Aushadhi price comparison sheet, so the compare view shows real savings
- 26 brand mappings for the search

No build step, no framework, no CDN, no backend, no analytics, no cookies.

---

## Where the data comes from

Every product on the page comes from the **Jan Aushadhi medicine list (generic name wise), product list updated 13.01.2023** — the official PMBJP price list. Each row in that PDF gives a serial number, a drug code, the generic name, the unit size, the MRP and the therapeutic group.

A second, smaller set comes from **Jan Aushadhi skin care product sheets** supplied as spreadsheets (`Jan_Aushadhi_Skin_Care_Products.xlsx`, 78 rows, and `_1.xlsx`, 91 rows). The 78-row file is a strict subset of the 91-row one — every one of its drug codes appears in the 91-row file — so the page is built from the 91-row file alone and nothing is double counted.

The skin care sheet adds face wash, medicated soap, toothpaste and mouthwash categories that the medicine list does not cover, and it carries its own MRP column. Where a skin care drug code also appears in the 2023 medicine list and the two prices differ, the detail sheet shows both figures and says which source each came from, instead of picking one silently.

Two data notes on that sheet, both visible in the page rather than smoothed over:

- One row, **Nitroglycerin Controlled Release Tablets 6.4 mg**, is filed under *Moisturiser / dryness*. It is a cardiac tablet, so it is excluded from the skin care set rather than published under a skin category.
- Several unit-size strings arrive truncated in the source (`20gm Tube`, `2.5ml with`, `10ml in`). They are reproduced as supplied; nothing is completed by guesswork.

A third source is the **Jan Aushadhi price comparison sheets** photographed and supplied for this build: Analgesic, Antidiabetic, Anti-biotic/Anti-fungal, Dermatology, Supplements/Vitamins & Minerals, Eye/Ear drops, Gastrointestinal, Respiratory and Steroids & Hormones. Each row there prints the medicine, the unit size, the **PMBJP price** and the **branded price** side by side. 60 of those rows matched a listing in the medicine or skin care set, and the branded price now powers the compare view and the "you save" line.

A fourth is the **Jan Aushadhi protein products price list** (13 items, drug codes 1464 to 2247 plus 2238): protein bars, powders, Janani, women's protein, renal and diabetes care powders and a whey protein jar, each with its protein content per 100 gm and its price.

Two honest notes on the comparison sheets:

- Their PMBJP prices are **not always identical** to the 2023 product list — for example Teneligliptin 20mg reads ₹50.00 on the sheet and ₹20.00 in the list, and most other rows sit about 6–10% lower on the sheet. The page keeps the list price as the headline and shows the sheet's figure in a "two prices for the same pack" note rather than silently overwriting one with the other.
- Branded price is the market price of the equivalent brand as printed on that sheet. It is not a quote from any particular shop.

Two small updates come from **Jan Aushadhi Kendra posts** the operator sent (October 2026):

- The **5-pad regular sanitary napkin pack** (JA-8199) is shown as "under process" in the 2023 list. The Kendra post and the PMBJP promo both price it at **₹5 for 5 pads**, which matches ₹1 a pad on the 4-pad (₹4) and 10-pad (₹10) packs already listed. That price is set, and the detail sheet says where it came from.
- **Evion (Vitamin E 400)** was added to the brand mapping, so searching it lands on the Jan Aushadhi Vitamin E 400 softgel. Its owner is listed as Procter & Gamble Health, which took over Merck's consumer health brands.

Those six columns are what the page shows. The source list does **not** contain manufacturer names, stock, expiry, batch numbers or product photographs, so the page leaves them blank or says "not listed". Nothing is filled in by guesswork.

Prices are the printed MRP from that list. They can change, and availability differs between Jan Aushadhi Kendras.

## Layout

```
index.html            the whole store — markup, tokens, styles and app logic inline
legal.html            privacy, terms, refunds, cookies, accessibility, security, disclosure
404.html              same styling, links back to the store
robots.txt            allows indexing, points at the sitemap
sitemap.xml           the two real pages
sources/              the Jan Aushadhi product list the data is parsed from
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
| Jan Aushadhi | Category chips, two promo cards, the "Indian generics for common brands" carousel, a suggested list, and a **Skin, hair and oral care** section |
| Medicines | The full list, 60 rows at a time with a **Show more** button, filtered by chip, therapeutic group or dosage form |
| Search | Brand-to-generic search with alternatives (below) |
| Prescriptions | Type the names a doctor wrote, one per line, and get the generic and its listed price for each |
| Categories | Therapeutic groups, skin care groups and dosage forms, each with a count; tapping one filters the list |
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

## Data shape

Each product is embedded as:

```js
{ n: name, m: manufacturer, c: category, s: pack size, d: one-line description,
  u: link, i: image id, p: product code, pr: MRP, pg: source page,
  fo: foreign owner, sr: serial number }
```

`m`, `u`, `i` and `fo` are empty strings for every product, because the Jan Aushadhi lists carry none of them. `pr` is empty where the source says "Under Process" or prints a dash. `pg` is the page of the source PDF the row came from, so any price can be checked by hand.

Skin care products live in a second array, `SKIN`, with the same shape plus two extra keys: `src: "skin"` and, where the drug code also exists in the 2023 list, `off: { pr, pg, n }` holding the official price and page for the dual-source note. Their serial numbers start at 10001 so saved items from the two sets never collide.

## The source PDF

The list this page is built from ships with it, at `sources/Jan-Aushadhi-Medicine-list-2023.pdf`, and the **You → Sources** block links to it with a `download` attribute, so anyone can check a price against the original page.

That file is the 13.01.2023 edition. The current PMBJP list lives behind `janaushadhi.gov.in`'s product API, which is served on port 8443 — a port this build environment's network policy blocks — and the site itself exposes no PDF path on 443. So the newer edition could not be pulled automatically. To update: drop the new PDF into `sources/`, run the parser, and the prices, counts and category counts all follow. Nothing else in the page is hard-coded to the 2023 numbers.

## Search: brand names and everyday words

Two different searches share one box.

**Brand names** — `Crocin`, `Combiflam`, `Evion`, `Thyronorm` — hit the mapping table and come back with Indian alternatives, the therapeutic category, who owns the brand and the Jan Aushadhi price.

**Everyday words** — `fever`, `acidity`, `sugar`, `BP`, `dandruff`, `deworming`, `eye`, `thyroid`, `protein`, and their Hinglish forms (`bukhar`, `dard`, `khansi`, `kabz`, `ulti`) — hit a lay-term table that maps the word to a filter over the listings. Searching `fever` returns 54 listings with printed MRPs, `acidity` returns 31, `dandruff` 13. **Show more** turns the result into a normal filtered list.

A product search matches on name, pack size and therapeutic group, so `sanitary napkin 5` finds the five-pad pack, and `metformin 500` finds the strengths and combinations.

The lay-term result carries the line "This is not medical advice. Follow your doctor's prescription." A symptom is not a diagnosis, and this page only prices what is listed.

## Pictures

Every listing carries a drawing, generated in the page from the listing itself — never a photograph, never a downloaded image.

- The silhouette follows the dosage form: a blister strip for tablets and capsules, a bottle for syrups and suspensions, a tube for creams, a vial for injections, a dropper for eye and ear drops, an inhaler, a sachet, or a box for devices.
- For tablets and capsules the number of dots follows the pack size, so a 10's pack and a 15's pack do not look the same.
- The colour comes from the therapeutic group, so a category reads consistently across the page.
- The detail sheet shows the same drawing larger, with the strength printed on it.

Why not photographs: the Jan Aushadhi product list and the skin care sheet carry no images, no image URLs and no manufacturer artwork. There is no licensed source of pack shots in this build, and the page is forbidden from loading anything from a CDN. So a real photo cannot be shown without inventing one or hotlinking someone else's. The drawing is labelled as a drawing in the detail sheet, and the old coloured letter tile still exists as the fallback if the SVG cannot render.

All of it is about 40 lines of JavaScript that return an SVG string — no image files, no build step, and the page stays one file.

## Languages

The interface runs in **English** and **हिन्दी**. The switch lives in **You → Language**, is remembered in `localStorage`, and applies to every view, sheet, button, empty state and `aria-label`.

How it works: there is one copy of the markup. `HI` is a dictionary of exact interface strings, `HI_PATTERNS` handles the ones with numbers in them (`12 products`, `See all 2026`, `3 Indian options found`), and a tree walker translates the rendered DOM after each render. Switching back to English restores the original text from a `WeakMap` snapshot, so nothing is lost.

What deliberately stays English: **medicine names, therapeutic groups and manufacturer names**. Those are printed identifiers from the source lists — translating "Paracetamol Tablets IP 500 mg" into Hindi would make the page harder to match against a pack, a prescription or a pharmacist's shelf.

To add a third language, copy the `HI` object, translate the values, and add the code to the switcher; nothing else in the page needs to change.

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
- Real `<button>` and `<a>` elements, visible focus rings, `aria-label` on icon-only buttons, 44px tap targets.
- A **Skip to content** link as the first tab stop.
- Dialogs carry `aria-modal`, trap Tab inside the sheet, close on Escape, and **return focus to the control that opened them**.
- Search results and list sizes are announced through a polite `role="status"` live region, so a screen-reader user hears "49 results for paracetamol" without moving focus.
- Matched text is highlighted with `<mark>`, and the search field has a labelled clear button.
- `prefers-reduced-motion` is respected, and a print stylesheet strips the navigation so a price list prints cleanly.
- With JavaScript off, a `<noscript>` block explains the page and links to the source PDF and the legal page.
- No emoji used as icons, no pill-shaped primary buttons, no purple gradients, no stock photography. A missing image becomes a coloured letter tile derived from the product name.

## What this page cannot do

- **It cannot read a prescription photo.** There is no OCR and no upload. Reading an image only works in the mobile app; the web page says so where a user would expect it.
- **It cannot check stock.** No pharmacy inventory is connected.
- **It cannot take payments.** No cart, no checkout, no prices other than the published MRP.
- **It gives no medical advice.** Substitution and dosage are for a doctor or pharmacist. Every view carries "Information only. Always follow your doctor's prescription."
- **It can only compare where the sheet prints a branded price.** For the 60 packs that have one, the compare sheet shows Jan Aushadhi price, branded price and the saving. For every other pack it says the branded price is not printed and shows no comparison, rather than inventing a number.
- **It cannot show manufacturer or expiry.** Not in the source list.

## Search behaviour worth knowing

- Typing is debounced by 140 ms; every keystroke re-renders from the in-memory arrays, so there is no network call at any point.
- A product match searches name, pack size and therapeutic group, so `sanitary napkin 5`, `10's` or `ophthalmic` all work.
- Brand and lay-term matches take precedence over plain text matching, and both show a banner before the rows so the reader knows why they are seeing those results.

## Running it

Open `index.html` in a browser. That is the whole setup. For a local server:

```
python3 -m http.server 8000
```

## Deploying

GitHub Pages serves the repository root directly; `.nojekyll` stops Jekyll from touching the files. Settings → Pages → Source: *Deploy from a branch* → `main` / `/ (root)`.

## Licence

MIT. The underlying price data belongs to the Pharmaceuticals & Medical Devices Bureau of India (PMBJP) and is reproduced from its public list. Brand names are trademarks of their respective owners and are used only to point to the generic equivalent.
