# Lion Marine on Wix: Site Port + Motor Parts & Inventory System — Design

**Date:** 2026-07-19
**Status:** Approved pending final user review

## Overview

Two connected efforts, built in the friend's existing Wix account:

1. **Port** the static redesign mockup (this repo, live at https://thecavepeanut.github.io/lionmarine-ex/) into a Wix site.
2. **Build** a motor parts & inventory system: serial-number lookup → motor → categorized parts list, stock tracking driven by QuickBooks Online invoices, supplier (Mercury) price/availability checks, engines-for-sale listings, and an estimate chatbot.

First pass covers **Mercury only**, but every schema and module is brand-agnostic: adding Yamaha/Honda later means new rows and a new supplier adapter, never new tables.

## Established facts and constraints

- Lion Marine is an **authorized Mercury dealer**; dealer API access (MercNET) is being requested. No public Mercury developer program exists.
- Billing is **QuickBooks Online**, and invoices are **itemized with SKU'd Product/Service items** (part numbers). This makes automatic SKU matching viable.
- **Mercury MAP policy:** no engine prices anywhere on the public site. Exact quotes go out **by email only** (private channel). The chatbot speaks only in **rounded estimates**.
- Part prices: not shown publicly for now. Mercury's dealer price (his cost) is **admin-only**.
- Wix account contains 4 sites: the live premium site (lionmarineservice.com, classic Editor), an old free site, and two fresh empty drafts — **"Lion Marine Studio"** (Studio, site id `46a2e985-ea90-457a-8a76-316aacb30260`) and "Lion Marine New" (classic Editor).
- Wix access works two ways: the **Wix MCP server** (configured with an API key in this project) and direct **Wix REST API** calls with the same credentials.
- Mockup conventions carry over: seagrass green `#3E6E56` design tokens, fixed contact facts, about-page stubs stay stubs, 55+-readable design.

## Architecture decision

**Approach A — everything in Wix, built on the "Lion Marine Studio" draft** — chosen over building in the classic Editor (B) or an external backend (C).

- Studio gives real responsive layout (closest to the mockup's CSS) and **Wix CLI local development**: backend code as real files in a git repo.
- **Portability requirement ("open door to C"):** all business logic is written as pure, Wix-free modules; Wix-specific code is confined to adapters. If the project outgrows Wix, only the adapter layer is rewritten (target: e.g. Cloudflare Workers + external DB).
- **Pre-flight check (first task):** confirm his premium plan — or a current-generation plan — can attach to a Studio site and carry the lionmarineservice.com domain. If Studio plans are a blocker, fall back to Approach B; the inventory architecture is identical either way.

## Data model (Wix CMS collections)

### Catalog — what exists in the world

| Collection | Purpose | Key fields |
|---|---|---|
| **Motors** | One row per motor model | brand, family (FourStroke, Pro XS…), horsepower, modelCode, yearRange, description |
| **SerialRanges** | Serial → motor decode table | brand, serialStart, serialEnd, year, →Motor |
| **Parts** | One row per part number | partNumber (SKU, must match QuickBooks exactly), name, →PartCategory, retailPrice, supersededBy, image, **supplierPrice, supplierAvailability, supplierShipDays, lastSupplierCheck** |
| **PartCategories** | Browse/search structure | name (Fuel System, Ignition, Cooling, Lower Unit…), sortOrder |
| **Fitment** | Motor↔Part join | →Motor, →Part, qtyRequired, diagramSection. Largest table: 400–700 rows per motor |

### Operations — what's physically in the shop

| Collection | Purpose | Key fields |
|---|---|---|
| **Inventory** | Current stock per part | →Part, qtyOnHand, reorderPoint, binLocation |
| **InventoryTransactions** | Append-only audit log; every stock change writes here | →Part, qtyDelta, reason (`invoice` \| `received` \| `dropped-in-water` \| `damaged` \| `used-unbilled` \| `correction`), qbInvoiceId, timestamp, enteredBy |
| **IncomingOrders** | Mercury orders in flight | orderNumber, lines [{part, qty}], orderDate, expectedArrival, status (`ordered`→`shipped`→`received`) |
| **EnginesForSale** | Physical engines on the floor | serial, →Motor, condition, photos, status (available/pending/sold), **hidden basePrice** (input to the pricing formula) |
| **QuoteRequests** | Chatbot leads + emailed quotes | contact, →EngineForSale, computedQuote, estimateShownInChat, status |

**Visibility rule:** public pages read only Motors, Parts, PartCategories, Fitment, Inventory's derived stock state, IncomingOrders' arrival dates, and EnginesForSale minus price. Everything else is backend/admin-only.

## Serial lookup

`resolveSerial(brand, serial)` — pure function: normalize (uppercase, strip spaces/dashes) → find containing SerialRanges row → return Motor + parts grouped by category + per-part stock state. Unrecognized serials get a friendly "call us" path and are **logged as misses** so popular unknown motors get prioritized for data entry.

## Mercury data acquisition — three lanes, one import format

1. **Dealer API** (preferred; pending): feeds the supplier adapter; if it exposes catalogs, bulk-populates Motors/Parts/Fitment.
2. **Structured import**: CSV/Excel exports from the dealer portal or DMS, uploaded via admin page. Most likely near-term path — ask the Mercury rep about exports alongside API access.
3. **Scraper fallback**: a local script (runs on the user's machine, **never inside Wix**), one engine at a time, respectful rate limits, outputs the same CSV format as lane 2. Defensible (authorized dealer, own service catalog) but brittle; fallback only.

All three lanes converge on one import format (motor + categorized part list + fitment quantities); collections never know the source.

**Pass-one scope:** one engine family fully loaded end-to-end (pick the motor he services most — ask him).

## Supplier price & availability

Backend module `supplierCatalog.getAvailability(partNumber) → { price, availability, shipDays }`.

- **Stub mode** until API credentials land: returns cached values on the Part, manually updatable.
- **Freshness:** cached with ~24h TTL, refreshed when an out-of-stock part is viewed; plus a nightly scheduled job refreshing every part below its reorder point.
- **Visibility:** ship timeframe is public ("typically ships from Mercury in 5–7 days"); Mercury's dealer price is **admin-only** (it's his cost, not retail). May refine later.
- Planned second method `getOrders()` for order tracking — see Incoming orders.

## Inventory flows

All stock changes go through one function: `recordTransaction(partNumber, qtyDelta, reason, context)` — appends to InventoryTransactions and updates Inventory atomically. Callers:

1. **QuickBooks webhook (main lane).** Intuit developer app → webhook on invoice create/update → Velo http-function endpoint → verify HMAC signature → fetch full invoice from QBO API (webhook carries only the ID) → match line SKUs to Parts → decrement, stamping the invoice ID. **Unmatched SKUs go to an admin review list** — never silently dropped. Invoice updates/voids reverse cleanly because the log records exactly what each invoice decremented. QBO OAuth tokens live in Wix Secrets Manager, refreshed by a scheduled job.
2. **"Oh shit" button.** Admin page: search part → quantity + reason (`dropped-in-water`, `damaged`, `used-unbilled`, `correction`) → done. Two taps on a phone.
3. **Receiving.** Same admin page, positive quantities, reason `received`. Completing an IncomingOrder posts its lines automatically.
4. **Initial count.** Bulk entry screen or CSV import for first-time shelf load.

### Incoming orders

- **Invoice-photo upload:** snap the Mercury invoice/packing slip → backend sends image to Claude API → extracted line items shown **for human confirmation** (never auto-committed) → confirmed lines create/complete an IncomingOrder and post `received` transactions.
- **API order tracking** (if MercNET exposes it): `getOrders()` auto-populates IncomingOrders with status and ETA. Unknown until credentials arrive; the photo route covers the same ground either way.
- Unlocks the third public stock state: **"On order — arriving ~date."**

## Public site

- **Port:** the five mockup pages rebuilt in Studio from this repo as visual spec; Studio design tokens configured from `css/tokens.css`; photos reused from the live site's media library.
- **Parts lookup page:** serial entry (or browse by family) → motor → parts by category, each showing stock state: **In stock** / **On order, arriving ~date** / **Call to order** (+ Mercury ship estimate when known). No public part prices.
- **Engines for sale page:** cards (photos, specs, condition), no prices, "Ask about this engine" opens the chatbot.
- **Admin area:** on-site pages behind Wix member login restricted to the owner; phone-first layout (used on the shop floor). Contains: review list, oh-shit button, receiving + photo upload, initial count, supplier price view, quote log.

## Chatbot & quotes

Chat widget (site-wide; engine-aware when opened from an engine card) → Velo backend route → Claude API.

- Answers from collection data (motors, parts, stock, shop info).
- **Pricing flow:** his pricing formula is implemented as code (`lib/pricing`), runs against the hidden basePrice → exact quote. The **rounding to an estimate happens in our code before the bot ever sees a number** — the model cannot leak what it was never given. Chat shows the estimate; exact quote is offered **by email** (collect address → send via Wix email API → log in QuoteRequests).
- Bot is instructed to hand off to a phone call when unsure. Every conversation that reaches pricing becomes a logged lead.

## Code structure (the A→C escape hatch)

Wix CLI project in a git repo:

```
lib/        Pure portable logic — NO Wix imports allowed.
            serial resolution, SKU matching, transaction rules,
            pricing/estimate math, invoice parsing, bot prompts
adapters/   Wix-specific glue: wix-data, Secrets Manager, email,
            + external clients: QuickBooks, Mercury, Claude
backend/    Thin route handlers: webhook endpoint, chat endpoint,
            scheduled jobs — wire adapters into lib
```

Migration to Approach C = rewrite `adapters/` only.

## Error handling & testing

- Webhook failures and unmatched SKUs land in the admin review list; nothing fails silently.
- Every external call (QBO, Mercury, Claude) fails soft with a logged alert; the site degrades to cached/manual data.
- `lib/` gets unit tests runnable locally (no Wix dependency by construction).
- InventoryTransactions is append-only; qtyOnHand is always reconcilable against the log.

## Open items (tracked, not blockers)

1. Confirm premium-plan / domain compatibility with Studio sites (fallback: Approach B, classic Editor).
2. Mercury dealer API request in progress — ask rep about **catalog exports** too (lane 2).
3. Ask him which engine family to load first (pass-one scope).
4. Intuit developer app setup on his QBO account (needs his login for OAuth consent).
5. His pricing formula — needs to be written down so it can become `lib/pricing`.
6. Later refinements parked: public part prices/markup, automated receiving from API order confirmations, email-forwarding lane for order confirmations, additional brands.
