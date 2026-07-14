# Lion Marine — redesign mockup

Static HTML/CSS/JS demo of a redesigned website for Lion Marine (outboard sales & repair, Honolulu, HI). It is a visual spec for rebuilding the friend's live Wix site (https://www.lionmarineservice.com/) inside his existing Wix account — not a production deployment. Live demo: https://thecavepeanut.github.io/lionmarine-ex/ (GitHub Pages, `master` branch root).

## Structure

- `index.html`, `services.html`, `motors.html`, `contact.html`, `about.html` — one file per page, no build step, no framework.
- `css/tokens.css` — design tokens only (colors, fonts, spacing). `css/base.css` — reset, typography, utilities. `css/components.css` — shared nav/footer/buttons/cards/forms. `css/pages.css` — page-specific sections.
- `js/main.js` — mobile nav toggle only.
- `assets/images/` — photos pulled from the live Wix site. `svc-electronics.jpeg` and `svc-solar.jpeg` are stock placeholders to be replaced with real job photos.
- `docs/superpowers/specs/` and `docs/superpowers/plans/` — design spec and implementation plan.

## Conventions (do not break)

- Header/nav/footer markup is intentionally duplicated verbatim across all 5 pages (accepted tradeoff; no templating). When editing nav or footer, apply the identical change to **all five** HTML files. Only `aria-current="page"` differs — it sits on the active page's link.
- All colors/fonts/spacing come from custom properties in `css/tokens.css`. Never hardcode hex values or font names elsewhere. Brand green is seagrass `#3E6E56` (`--color-hull-green`; name kept for stability).
- Contact facts are fixed and must match everywhere: (808) 300-6894 (`tel:+18083006894`), Lionmarineservice@gmail.com, 24 Sand Island Access Rd, Honolulu, HI 96819 (Keehi Marine Center), P.O. Box 8174 Honolulu HI 96830, Mon–Fri 8AM–5PM.
- No prices on the Motors page — every motor family says "call or email for current pricing" (client decision; prices go stale).
- About page sections (story, reviews, gallery) are deliberate "coming soon" stubs. Never fill them with invented bios, quotes, or photos.
- Design intent: fun/modern but readable for 55+ — big type, high contrast, chunky buttons, no interactions that need explaining.

## Verifying changes

Use the chrome-devtools MCP tools against `file:///C:/Users/prsja/projects/lionmarineservice/<page>.html`. Check `document.documentElement.scrollWidth` at 375/768/1440/1920 widths (must not exceed viewport). **Use the `emulate` tool's `viewport` param for narrow widths — `resize_page` has a ~500px floor on this machine.** Full-page screenshots of `contact.html` may show a duplicated sticky header; that's a capture artifact, not a bug.

## Deploying

`git push origin master` — GitHub Pages serves the branch root directly; changes go live in ~1 minute. Repo: https://github.com/Thecavepeanut/lionmarine-ex (public).
