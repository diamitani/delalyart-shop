# Delaly Art — gallery portfolio + fine-art print shop

A gallery-grade portfolio and print store for **Delaly Art** ([@delalyart.shop](https://instagram.com/delalyart.shop)).
The site is organized into **exhibits** (curated collections); the first exhibit is *Beach* —
27 Lake Michigan shoreline studies, each reimagined through the hand of a master of art history.

**How it works:** a pure-Node build script reads `data/catalog.json` + `data/config.json`
and emits a fully static site into `dist/`. The cart is vanilla JS + localStorage with a
slide-over drawer on every page. Checkout is a Vercel serverless function that creates a
Stripe Checkout Session — prices are **recomputed server-side**, never trusted from the client.

## Project structure

```
delalyart-shop/
├── data/
│   ├── catalog.json      # exhibits[] + artworks[] — the whole gallery lives here
│   └── config.json       # site name, artist, email, instagram, print sizes + prices
├── public/art/           # original artwork JPEGs (+ thumbs/ generated at build prep)
├── src/assets/css/       # style.css — the gallery design system
├── src/assets/js/        # shop.js — cart drawer, buy box, checkout redirect
├── scripts/
│   ├── build.js          # static site generator (pure Node, zero deps)
│   ├── templates.js      # HTML layout + card templates
│   ├── check-links.js    # dead-link checker over dist/
│   └── serve.js          # tiny local static server for previews
├── api/
│   ├── checkout.js       # Vercel serverless fn: POST /api/checkout -> Stripe session
│   └── _lib.js           # shared validation + line-item builder (also used by tests)
├── test/
│   └── checkout.test.js  # node:test suite with a mocked Stripe client
├── dist/                 # build output (generated; this is what gets deployed)
├── package.json          # scripts + the `stripe` dependency (for the API fn)
├── vercel.json           # Vercel: build command, output dir, function file includes
└── .env.example          # env vars needed for live checkout
```

## Quick start

```bash
cd delalyart-shop
node scripts/build.js        # generate dist/
node scripts/serve.js        # preview at http://localhost:4173
```

Full verification (build + dead-link check + tests):

```bash
npm run verify
```

## Adding a new exhibit (3 steps)

Future exhibits (new styles, new subjects) slot straight in — no code changes:

1. **Drop the images** into `public/art/` (and matching thumbnails into `public/art/thumbs/` —
   generate with e.g. `python3 -c` + PIL, max 900px, quality 82).
2. **Add one exhibit object** to `exhibits[]` in `data/catalog.json`:
   `{ "id": "city", "title": "City", "subtitle": "...", "description": "...", "coverImage": "city-01.jpg" }`.
3. **Add artwork objects** to `artworks[]` with `"exhibitId": "city"` — each needs
   `id` (unique, URL-safe), `title` (evocative gallery title), `style` ("After …"),
   `image` (filename in `public/art/`), and `blurb` (1–2 sentences).

Then `node scripts/build.js` — the exhibits index, exhibit page, and all artwork pages
(including "more from this exhibit" rails) generate automatically.

## Setting real prices

**Prices in `data/config.json` are PLACEHOLDERS** (`priceNote` says so explicitly).
Edit `printSizes[].price` (whole dollars) and rebuild — the storefront, cart subtotals,
and the server-side checkout all read from this one file, so a single edit updates everything.

## Configuring Stripe + deploying on Vercel

1. Create a Stripe account and grab a **secret key** (`sk_live_…`; use `sk_test_…` while testing).
2. In the Vercel project settings → Environment Variables, add:
   - `STRIPE_SECRET_KEY` = your Stripe secret key (Production + Preview as appropriate)
   - `SITE_URL` = `https://delalyart.shop` (your real domain; falls back to the Vercel URL automatically)
3. Deploy: Vercel detects `vercel.json` — build command `node scripts/build.js`, output `dist/`.
   The `stripe` npm package is installed automatically from `package.json` for the API function.
4. Test checkout end-to-end with a test key and Stripe's test card `4242 4242 4242 4242`
   before switching to live keys.
5. Point your domain at the Vercel project and update `siteUrl` in `data/config.json`.

The checkout function uses **inline `price_data`**, so no products need to be pre-created
in the Stripe dashboard. Shipping address collection is enabled; success/cancel pages
live at `/checkout/success/` and `/checkout/cancel/`.

## Design

Warm off-white (`#faf8f3`) gallery walls, ink-black type, Cormorant Garamond display serif
+ Inter body via Google Fonts. Fully responsive (desktop grid → single column on mobile),
keyboard-dismissable cart drawer (Esc), lazy-loaded imagery, semantic HTML.
