# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this repo is

A single-page static demo site that exercises the Shopify Buy Button SDK (`buy-button-js`) against a Simon & Schuster dev storefront, wired to the SNS/Glassboxx application for purchasability + pre-order gating. There is **no build tooling here** — only `index.html`, `buybutton.min.js`, and `README.md`. The SDK source lives in a sibling repo `../buy-button-js/`; this site is where changes get smoke-tested in a real browser.

## Running the demo

```bash
open index.html                # simplest
python3 -m http.server 8080    # or serve locally
```

When iterating on SDK source, rebuild in the sibling repo and copy the artifact in:

```bash
cd ../buy-button-js && yarn run src:watch
cp tmp/buybutton.dev.js ../sns-demo-site/buybutton.min.js
```

Note: `index.html` currently loads the **CDN** build (`sdks.shopifycdn.com/buy-button/latest/...`), not the local `./buybutton.min.js`. To test a local build, swap the `sdkUrl` assignment near the top of the `<script>` block in `index.html`.

## Architecture — what actually happens when the page loads

Everything lives inline in `index.html`. Three external systems are wired together:

1. **Shopify Storefront API** (`simon-and-schuster-dev.myshopify.com`, API version `2025-01`) — product listing, cart mutations, cart attribute updates. Accessed with the hardcoded `token` (Storefront Access Token) directly via `fetch` for most custom logic, and via the SDK for the buy-button UI.
2. **SNS application** (`simon-dev.glassboxx.com`) — `GET /api/public/product-purchasable` returns `{ isPurchasable, isPreOrder }` per product. Authenticated with `snsAppToken`. This gate determines whether a product renders a buy button at all, and whether the button says "Buy From Us" vs. "Pre Order".
3. **Buy Button SDK** (`ShopifyBuy.UI`) — initialized with `snsAppUrl` + `snsAppToken` so its internal cart code can call the SNS app too. `useCustomGraphQL: true` in the cart options tells the SDK to use custom cart mutations that include the `requiresShipping` field (the whole reason this demo exists — see README).

### Page flow

1. Capture UTM params from URL → `localStorage.SANDS_UTM`.
2. `renderProductListing()` paginates products via Storefront GraphQL, builds a card grid, then for each product calls the SNS purchasability endpoint **one-by-one** (sequential `await` inside the loop) and only mounts a Buy Button component for purchasable products.
3. On checkout click (`cart.onCheckout` override), the current page URL is written to the cart as a `shopping_url` attribute via `cartAttributesUpdate` before opening `cart.model.webUrl`. This is how the SNS app knows where the buyer came from.

### BXGY engine (inline, ~lines 121–404)

The "Buy X Get Y" free-gift logic is implemented **outside the SDK**, directly against the Storefront API. `BXGYEngine`:

- Reads cart lines with `cart(id:).lines` query (not the SDK's cached model — always fresh).
- Counts qualifying quantities (excluding gift variants), computes `want` for each rule, and reconciles via `cartLinesAdd` / `cartLinesUpdate` / `cartLinesRemove`. Gift lines are tagged with attribute `_bxgy_gift=true`.
- Tracks `_lastGiftState` to detect **manual removal** — once a user removes a gift, it is not re-added even if the trigger is still met.
- After mutating the cart, calls `_refreshSDK()` to re-fetch via `client.cart.fetch` (or `client.checkout.fetch` as fallback) and `cart.render()` so the SDK UI matches reality.
- Triggered from the SDK's `cart.events.afterRender` hook, **debounced 800ms** to let the SDK's own model update first and avoid reconcile loops. `busy` flag prevents overlapping runs.
- Rules are configured in the `bxgyRules` array near the top — edit product/variant IDs there to change promos.

### Keeping BXGY and the SDK in sync

This is the subtle part. The SDK owns cart rendering and its own view of the cart; BXGY mutates the cart out-of-band. The debounce + `busy` flag + `_refreshSDK` + `_lastGiftState` are all there to keep the two views consistent without thrashing. When editing BXGY or the cart `events` handlers, preserve that discipline — removing the debounce or the busy guard will cause reconcile loops.

## Secrets

`index.html` contains hardcoded Shopify Storefront token and SNS app token for this **dev** store. Do not swap in production credentials — this file is served as-is to the browser.
