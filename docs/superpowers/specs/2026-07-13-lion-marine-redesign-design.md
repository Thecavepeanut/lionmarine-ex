# Lion Marine Website Redesign — Design Spec

**Date:** 2026-07-13
**Source site:** https://www.lionmarineservice.com/ (Wix, classic/legacy editor)
**Deliverable of this spec:** an HTML/CSS mockup (not a deployed site) that serves as the exact visual/content reference for manually rebuilding the site inside the friend's existing Wix account, preserving his prepaid hosting.

## Goals

- Preserve the current brand identity (lion head mark, forest-green accent, marine/Hawaii subject matter) while giving the site a distinctive, non-templated visual identity.
- Fix concrete problems found on the live site (see Audit below).
- Produce a mockup detailed enough that rebuilding it in Wix's editor (classic or Wix Studio) is a mechanical translation job, not a design decision process.
- Stay within the current 4-page scope (Home, Services, Motors, Contact) plus stubbed placeholder sections the client explicitly wants reserved (About, Testimonials, Gallery) but has no content for yet.

## Non-goals

- Not building a deployed/hosted site — friend keeps his prepaid Wix hosting and rebuilds there (by hand, or via browser automation in a later session if he grants Wix login access).
- Not sourcing real photography for Electronics/Audio or Solar services — current Unsplash stock photos are kept as placeholders, clearly flagged for the friend to swap in real job photos later.
- Not transcribing the full Mercury price list — replaced with motor family/HP-range categories + "call or email for current pricing," per client decision (prices change often; a static list becomes stale and was already just an unreadable screenshot).

## Audit of current site (what we're fixing)

1. **Non-responsive/clipped layout.** Classic Wix editor uses a fixed ~1905px-wide absolutely-positioned canvas. At 1440px viewport width, both left and right-hand content columns are cut off entirely (confirmed via screenshot at 1440px vs. 1920px).
2. **~2000px of dead empty white space** below the fold on the homepage.
3. **Motors page is a single 963×19,079px JPEG screenshot** of a price sheet — not real text, not readable on mobile, not maintainable.
4. **Motors page "Product Gallery" section is empty** ("We don't have any products to show here right now").
5. Generic default Wix button styling (soft rounded pill buttons, default shadows) with no distinctive identity.
6. Two of four Services images (Electronics/Audio, Solar) are unrelated Unsplash stock photos, not the shop's own work.

## Content inventory (carried over from live site)

- **Business:** Lion Marine — "Outboard Sales and Repair," Honolulu, HI
- **Phone:** (808) 300-6894
- **Email:** Lionmarineservice@Gmail.com
- **Physical:** 24 Sand Island Access Rd, Honolulu, HI 96819 (Keehi Marine Center)
- **Mailing:** P.O. Box 8174, Honolulu, HI 96830
- **Hours:** Mon–Fri, 8am–5pm. After-hours emergency calls accepted, additional charge.
- **Services:** Mercury outboard sales (authorized dealer), outboard diagnostics/repair, engine repowering, marine electronics (Raymarine dealer), Fusion audio systems/marine audio, solar, parts sales, mobile service across the Hawaiian Islands.
- **Get Pre-Qualified** financing CTA (QR code + link) for new outboards/repowers.
- **Nav:** Home, Services, Motors, Contact Us.

## Design system

### Palette

| Token | Hex | Usage |
|---|---|---|
| `hull-green` | `#104F1E` | primary accent (kept from current brand) |
| `engine-black` | `#14181A` | dark sections, primary text |
| `plate-paper` | `#F3F0E8` | main light background |
| `sand-tan` | `#C9BB9C` | secondary neutral, dividers, borders |
| `brass` | `#B4863B` | sparing highlight — CTAs, hover states, spec numbers |
| `chart-navy` | `#0E2A36` | deep gradient backgrounds, footer |

### Type

- Display: **Big Shoulders Display** (condensed industrial grotesk) — headlines
- Body: **IBM Plex Sans** — body copy, nav, UI labels
- Data/utility: **IBM Plex Mono** — phone numbers, hours, HP/spec figures, prices, data-plate codes

All three are open-source Google Fonts, freely usable when rebuilt in Wix (Wix supports custom font upload or Google Fonts via Velo/embed).

### Signature element — "data plate" card

A recurring card motif modeled on the stamped metal ID plate riveted to every outboard motor: beveled panel, subtle corner rivet dots, hairline brass border, a small monospace "spec code" label (e.g. `SVC—01 REPOWER`) above the title. Used for: service categories, motor family categories, and (once populated) testimonials. Not used elsewhere, to keep it a signature rather than a wallpaper pattern.

### Components

- Beveled buttons (not rounded pills) — flat industrial panel look, brass accent on hover/focus.
- Full-bleed section photography (hero, section breaks) instead of the current small centered/clipped images.
- Sticky top nav bar with phone number always visible (current site repeats phone/email in the header on every page — keep that pattern, it works).

## Page-by-page layout

### Home
1. Full-bleed hero: real outboard/boat photo (from downloaded assets) with dark gradient overlay, condensed display headline ("Outboard Sales & Repair — Honolulu, HI"), phone CTA + "Get a Quote" button.
2. Trust bar: Mercury / Raymarine / Fusion dealer badges, small data-plate style.
3. Mobile service area + hours/location strip.
4. Services teaser grid (data-plate cards, links to Services page).
5. Get Pre-Qualified CTA block (financing QR + link).
6. Footer with hours, address, contact, nav.

### Services
- Same 4 categories as today (Repowering, Outboard Engine Services, Electronics & Audio, Solar), each a data-plate card with photo, title, 1–2 line description.
- "Get a Free Quote" form retained (First/Last name, Email*, Phone, Engine Serial #, Service Requesting).

### Motors
- Intro: "2026 Mercury Outboards" heading.
- Motor family cards by HP range (e.g. FourStroke portable 2.5–20HP, FourStroke mid-range 25–115HP, Pro XS/V6-V8 performance, SeaPro commercial) with representative photo and HP range shown gauge/dial-style.
- Prominent "Call (808) 300-6894 or email for current pricing" CTA on every card — no static price list.
- Get Pre-Qualified block retained.

### Contact
- Address (physical + mailing), phone, email, hours — styled as a dockside sign board panel.
- Contact form (First/Last name, Email*, Message) retained.
- Google Map embed retained.

### Placeholder sections (stubbed only, clearly marked)
- **About/Bio** — short "Coming soon" stub with a labeled placeholder box, not fake content.
- **Testimonials** — stub with placeholder card(s) explicitly marked "Reviews coming soon."
- **Gallery** — stub grid explicitly marked "More project photos coming soon."

These are visually designed (so the friend can see where/how they'll slot in) but contain no invented customer quotes, bio text, or fake photos — only clearly-labeled placeholders.

## Assets

Downloaded from live site into `assets/images/`: `bg-water.jpg`, `logo-lion.png`, `hero-boat-1.jpg`, `hero-boat-2.jpg`, `svc-repowering.jpeg`, `svc-outboard.jpeg`, `svc-electronics.jpeg` (stock), `svc-solar.jpeg` (stock).

## Handoff / rebuild-in-Wix plan

The mockup is built as static HTML/CSS (no framework) so it's easy to view directly in a browser and use section-by-section as a visual spec while manually rebuilding in the Wix Editor (or Wix Studio, if the friend later chooses to migrate for better responsive tooling — out of scope for this pass). Colors, fonts, and spacing are called out as CSS custom properties at the top of the stylesheet so they translate directly into Wix's design panel values.

## Testing / QA checklist for the mockup

- Renders correctly at 375px (mobile), 768px (tablet), 1440px and 1920px (desktop) with no clipped/cut-off content.
- All copy carried over accurately from the audit above (phone, email, addresses, hours).
- Placeholder sections clearly marked as placeholders, not mistakable for real content.
- Keyboard focus visible on all interactive elements; reduced-motion respected for any transitions.
