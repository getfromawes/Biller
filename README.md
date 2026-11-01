# Shawear

Offline-first billing software for hardware shops. It runs in any modern browser, installs on a phone or PC as an app, and keeps working when the shop's internet goes down. All data is stored on the device.

## Run it

```bash
npm install        # only needed for the tests
npm start          # serves the app at http://localhost:8080
```

There is no build step. Any static file server works (`python3 -m http.server`, `npx serve`, nginx). To use it on a phone, open the server's address on the same Wi-Fi, then choose "Install app" or "Add to Home screen". Service workers need `localhost` or HTTPS, so use HTTPS when you host it for real.

On the first run, a new shop sees only two buttons on the home screen: **Add party** and **Add bill**. Open Settings to enter the shop name and address, and to load sample hardware items and customers so you can try a bill straight away. Once the first bill is saved, the home screen fills with sales, dues and low stock.

## What is in the base

| Area | What it does |
| --- | --- |
| Billing | Type to search thousands of items, Enter adds the top match, arrow keys pick another. Edit quantity, unit, rate and discount per line. Retail, contractor and wholesale price levels. Cash, UPI, card or credit (udhaar), with part payment. Warns on low stock and when a credit limit is crossed. The total is the sum of the lines, rounded to the rupee, with amount in words (Indian system). There is no GST or tax anywhere in the app. |
| Units | Stock is kept in a base unit (for example kg). Each item can also be sold in up to two other units with a conversion (box of 5 kg, coil of 90 m). Stock is reduced in base units whatever unit is billed. |
| Items | Name, brand, code, three prices, cost, stock, minimum stock. Stock-in with optional supplier bill. |
| Parties | Customers and suppliers with a running ledger, opening balance, credit limit, receive and pay payments, and a WhatsApp reminder for pending balances. |
| Invoices | Every bill is one fixed A4 portrait page (long bills continue on more A4 pages), identical in the preview, on paper, in the WhatsApp picture and in the PDF. **Share on WhatsApp** sends the bill as a picture (the phone's share sheet opens; on a PC the picture is saved and the customer's WhatsApp chat opens so you can attach it). **PDF** shares or saves a PDF. Customer name is bold. 80mm thermal print is still available. Cancel puts stock back and reverses the balance. |
| Offline | Installable PWA with a service worker. Backup and restore as a JSON file. |

## Code map

```
index.html            page shell
manifest.webmanifest  install metadata
sw.js                 offline cache
src/billing.js        pure billing maths (no DOM, no storage)
src/invoice-view.js   lays out and draws the A4 bill (canvas); one source for preview, print, picture and PDF
src/pdf.js            tiny PDF writer (one A4 page per picture)
src/db.js             IndexedDB data layer; invoice save, cancel, payments and stock run in single transactions
src/app.js            screens and routing
src/styles.css        styles and print layouts
tests/                billing, db, invoice layout, pdf, and a UI smoke test driven in jsdom
```

Run all tests with `npm test`.

## Conventions worth knowing

- A party's `balance` is positive when they owe the shop and negative when the shop owes them. The ledger is the sum of debits minus credits, and it always matches the stored balance.
- Bills are drawn from the saved invoice, so old invoices also use the current layout.
- Invoice lines store a copy of the item name, rate and amount at the time of sale, so editing or deleting an item never changes an old bill.
- Prices are stored per base unit; a line in another unit uses price x conversion factor by default and can be edited.

## Updating a deployed copy

Data is stored per web address. Redeploy to the **same** site (on Netlify: Site, Deploys, drag the folder there). A brand new drop creates a new address with empty data. Open the app online once, then close and reopen it so the update loads.

## Before you rely on it

- **No tax.** Shawear prints plain invoices and does not calculate, show or file GST. Bills saved earlier with GST keep their final amounts but no longer show the tax split.
- **Data lives in the browser.** Clearing site data or losing the device loses the data. Use Settings, Download backup regularly until cloud sync exists.

## Suggested next steps

1. Pilot with 3 to 5 shops and watch how they bill. Fix whatever slows them down at the counter first.
2. Purchases module: purchase bills with many lines, supplier bill numbers, purchase returns.
3. Quotations and estimates that convert to invoices; sales returns and credit notes.
4. Cloud sync and backup: a small server (Postgres plus an API) with per-shop accounts. IndexedDB stays the source of truth on the device and syncs when online.
5. Barcode scanning and a barcode label printer; direct thermal printing (ESC/POS over Bluetooth or USB).
6. Roles (owner, cashier), shop-wise reports (day book, stock valuation, party-wise outstanding), WhatsApp Business API for automatic reminders.
7. Pricing and licensing: a free pilot, then a yearly plan per shop.
