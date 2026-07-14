# Lion Marine Redesign Mockup Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Build a static HTML/CSS/JS mockup of the redesigned Lion Marine site (5 pages: Home, Services, Motors, About, Contact) matching `docs/superpowers/specs/2026-07-13-lion-marine-redesign-design.md`, and publish it to GitHub Pages as a live demo link.

**Architecture:** Plain static HTML per page (no framework, no build step) sharing a small set of hand-authored CSS files (`css/tokens.css`, `css/base.css`, `css/components.css`, `css/pages.css`) and one JS file (`js/main.js`) for the mobile nav toggle. Header/nav and footer markup is duplicated identically across pages (5 pages, no templating engine in scope). Images already downloaded to `assets/images/`.

**Tech Stack:** HTML5, CSS3 (custom properties), vanilla JS, Google Fonts (Big Shoulders Display, IBM Plex Sans, IBM Plex Mono). No package manager, no build tool. Verification uses the chrome-devtools MCP tools already connected in this session (`navigate_page`, `evaluate_script`, `take_screenshot`) against local `file://` paths.

## Global Constraints

- Palette, type, and spacing tokens must come from the spec's Design System section verbatim: hull-green `#104F1E`, engine-black `#14181A`, plate-paper `#F3F0E8`, sand-tan `#C9BB9C`, brass `#B4863B`, chart-navy `#0E2A36`.
- Fonts: display = Big Shoulders Display, body = IBM Plex Sans, mono = IBM Plex Mono (Google Fonts CDN, no local hosting).
- Nav must always include: Home, Services, Motors, About, Contact, and the phone number `(808) 300-6894`.
- All contact details must exactly match the spec's Content Inventory (phone, email, addresses, hours) — no invented content.
- Placeholder sections (About/Bio, Testimonials, Gallery) must be visually and textually unmistakable as placeholders ("Coming soon" / dashed borders) — never fabricated customer quotes or bio copy.
- No horizontal scroll/clipping at 375px, 768px, 1440px, or 1920px viewport widths on any page.
- If the chrome-devtools MCP tools are not already loaded in a fresh session, load them first via `ToolSearch` with query `select:mcp__plugin_chrome-devtools-mcp_chrome-devtools__navigate_page,mcp__plugin_chrome-devtools-mcp_chrome-devtools__evaluate_script,mcp__plugin_chrome-devtools-mcp_chrome-devtools__take_screenshot,mcp__plugin_chrome-devtools-mcp_chrome-devtools__resize_page,mcp__plugin_chrome-devtools-mcp_chrome-devtools__new_page`.
- Project root: `C:\Users\prsja\projects\lionmarineservice`. Git remote `origin` is already set to `https://github.com/Thecavepeanut/lionmarine-ex.git`; local git identity is already configured (user.name "Jake Grajewski", user.email "prs.jake@gmail.com").

---

### Task 1: Design tokens, base styles, shared components, and nav JS

**Files:**
- Create: `css/tokens.css`
- Create: `css/base.css`
- Create: `css/components.css`
- Create: `js/main.js`

**Interfaces:**
- Produces CSS custom properties consumed by every later task: `--color-hull-green`, `--color-engine-black`, `--color-plate-paper`, `--color-sand-tan`, `--color-brass`, `--color-chart-navy`, `--color-white`, `--font-display`, `--font-body`, `--font-mono`, `--space-1`..`--space-7`, `--radius-sm`, `--radius-md`, `--container-width`, `--focus-ring`.
- Produces shared component classes consumed by every page: `.container`, `.section`, `.section__heading`, `.eyebrow`, `.font-display`, `.font-mono`, `.sr-only`, `.site-header`, `.nav`, `.nav__brand`, `.nav__logo`, `.nav__brand-text`, `.nav__links`, `.nav__links--open`, `.nav__phone`, `.nav__toggle`, `.nav__toggle-bar`, `.btn`, `.btn--primary`, `.btn--ghost`, `.plate-grid`, `.plate-card`, `.plate-card__code`, `.plate-card__title`, `.plate-card__body`, `.rivet` (`--tl`/`--tr`/`--bl`/`--br`), `.form`, `.form__row`, `.form__field`, `.site-footer`, `.site-footer__grid`, `.site-footer__col`, `.site-footer__brand`, `.site-footer__logo`, `.site-footer__name`, `.site-footer__legal`, `.stub-panel`, `.stub-panel__label`, `.stub-grid`, `.stub-box`.
- Produces `js/main.js` behavior: clicking `#navToggle` toggles `.nav__links--open` on `#navLinks` and flips `aria-expanded`.

- [ ] **Step 1: Write `css/tokens.css`**

```css
:root {
  --color-hull-green: #104F1E;
  --color-engine-black: #14181A;
  --color-plate-paper: #F3F0E8;
  --color-sand-tan: #C9BB9C;
  --color-brass: #B4863B;
  --color-chart-navy: #0E2A36;
  --color-white: #FFFFFF;

  --font-display: 'Big Shoulders Display', sans-serif;
  --font-body: 'IBM Plex Sans', sans-serif;
  --font-mono: 'IBM Plex Mono', monospace;

  --space-1: 0.5rem;
  --space-2: 1rem;
  --space-3: 1.5rem;
  --space-4: 2rem;
  --space-5: 3rem;
  --space-6: 4.5rem;
  --space-7: 7rem;

  --radius-sm: 2px;
  --radius-md: 6px;

  --container-width: 1180px;

  --focus-ring: 3px solid var(--color-brass);
}
```

- [ ] **Step 2: Write `css/base.css`**

```css
*, *::before, *::after { box-sizing: border-box; }
html { scroll-behavior: smooth; }
@media (prefers-reduced-motion: reduce) {
  html { scroll-behavior: auto; }
  *, *::before, *::after {
    animation-duration: 0.01ms !important;
    animation-iteration-count: 1 !important;
    transition-duration: 0.01ms !important;
  }
}
body {
  margin: 0;
  font-family: var(--font-body);
  color: var(--color-engine-black);
  background: var(--color-plate-paper);
  line-height: 1.55;
}
h1, h2, h3 {
  font-family: var(--font-display);
  font-weight: 700;
  line-height: 1.05;
  margin: 0 0 var(--space-2);
  letter-spacing: 0.01em;
}
h1 { font-size: clamp(2.5rem, 6vw, 5rem); }
h2 { font-size: clamp(1.75rem, 3.5vw, 2.75rem); }
h3 { font-size: 1.25rem; }
p { margin: 0 0 var(--space-2); }
a { color: inherit; }
img { max-width: 100%; display: block; }
.font-display { font-family: var(--font-display); }
.font-mono { font-family: var(--font-mono); }
.sr-only {
  position: absolute; width: 1px; height: 1px; padding: 0; margin: -1px;
  overflow: hidden; clip: rect(0,0,0,0); white-space: nowrap; border: 0;
}
.container { max-width: var(--container-width); margin: 0 auto; padding: 0 var(--space-3); }
.section { padding: var(--space-6) var(--space-3); }
.section__heading { text-align: center; margin-bottom: var(--space-5); }
a:focus-visible, button:focus-visible, input:focus-visible, textarea:focus-visible {
  outline: var(--focus-ring); outline-offset: 2px;
}
.eyebrow {
  display: block; text-transform: uppercase; letter-spacing: 0.12em;
  color: var(--color-brass); font-size: 0.85rem; margin-bottom: var(--space-2);
}
```

- [ ] **Step 3: Write `css/components.css`**

```css
/* Header / Nav */
.site-header {
  position: sticky; top: 0; z-index: 50;
  background: var(--color-plate-paper);
  border-bottom: 1px solid var(--color-sand-tan);
}
.nav {
  display: flex; align-items: center; justify-content: space-between;
  padding-top: var(--space-2); padding-bottom: var(--space-2); gap: var(--space-3);
  position: relative;
}
.nav__brand { display: flex; align-items: center; gap: var(--space-2); text-decoration: none; }
.nav__logo { width: 40px; height: 40px; object-fit: contain; }
.nav__brand-text { font-family: var(--font-display); font-size: 1.4rem; font-weight: 700; color: var(--color-engine-black); }
.nav__links { display: flex; gap: var(--space-4); list-style: none; margin: 0; padding: 0; }
.nav__links a { text-decoration: none; font-weight: 600; padding: var(--space-1) 0; border-bottom: 2px solid transparent; }
.nav__links a[aria-current="page"], .nav__links a:hover { border-bottom-color: var(--color-brass); }
.nav__phone { font-family: var(--font-mono); font-weight: 600; text-decoration: none; white-space: nowrap; }
.nav__toggle { display: none; flex-direction: column; gap: 4px; background: none; border: 0; cursor: pointer; padding: var(--space-1); }
.nav__toggle-bar { width: 24px; height: 2px; background: var(--color-engine-black); }

@media (max-width: 860px) {
  .nav__toggle { display: flex; }
  .nav__links {
    position: absolute; top: 100%; left: 0; right: 0;
    background: var(--color-plate-paper); border-bottom: 1px solid var(--color-sand-tan);
    flex-direction: column; gap: 0; max-height: 0; overflow: hidden;
  }
  .nav__links a { padding: var(--space-2) var(--space-3); width: 100%; }
  .nav__links--open { max-height: 400px; }
  .nav__phone { display: none; }
}

/* Buttons */
.btn {
  display: inline-block; font-family: var(--font-body); font-weight: 700;
  padding: var(--space-2) var(--space-4); text-decoration: none; border-radius: var(--radius-sm);
  border: 2px solid transparent; cursor: pointer; font-size: 1rem;
}
.btn--primary { background: var(--color-hull-green); color: var(--color-white); border-color: var(--color-hull-green); }
.btn--primary:hover { background: var(--color-brass); border-color: var(--color-brass); }
.btn--ghost { background: transparent; color: var(--color-white); border-color: var(--color-white); }
.btn--ghost:hover { background: var(--color-white); color: var(--color-engine-black); }

/* Data-plate card */
.plate-grid { display: grid; grid-template-columns: repeat(auto-fit, minmax(240px, 1fr)); gap: var(--space-4); }
.plate-card {
  position: relative; display: block; background: var(--color-white);
  border: 1px solid var(--color-sand-tan); border-top: 3px solid var(--color-brass);
  border-radius: var(--radius-md); padding: var(--space-4); text-decoration: none; color: inherit;
  overflow: hidden;
}
.plate-card img { border-radius: var(--radius-sm); margin-bottom: var(--space-3); aspect-ratio: 4/3; object-fit: cover; }
.plate-card__code { display: block; color: var(--color-brass); font-size: 0.75rem; letter-spacing: 0.08em; margin-bottom: var(--space-1); }
.plate-card__title { margin-bottom: var(--space-1); }
.plate-card__body { margin: 0; font-size: 0.95rem; }
.rivet { position: absolute; width: 6px; height: 6px; border-radius: 50%; background: var(--color-sand-tan); }
.rivet--tl { top: 8px; left: 8px; }
.rivet--tr { top: 8px; right: 8px; }
.rivet--bl { bottom: 8px; left: 8px; }
.rivet--br { bottom: 8px; right: 8px; }

/* Forms */
.form { display: grid; gap: var(--space-3); max-width: 640px; }
.form__row { display: grid; grid-template-columns: 1fr 1fr; gap: var(--space-3); }
.form__field { display: grid; gap: var(--space-1); }
.form__field label { font-weight: 600; font-size: 0.9rem; }
.form__field input, .form__field textarea {
  font: inherit; padding: var(--space-2); border: 1px solid var(--color-sand-tan); border-radius: var(--radius-sm);
  background: var(--color-white);
}
.form__field textarea { min-height: 120px; resize: vertical; }
.form .btn { justify-self: start; }

@media (max-width: 600px) {
  .form__row { grid-template-columns: 1fr; }
}

/* Footer */
.site-footer { background: var(--color-chart-navy); color: var(--color-plate-paper); padding: var(--space-6) 0 var(--space-3); }
.site-footer__grid { display: grid; grid-template-columns: 1.2fr 1fr 1fr 1fr; gap: var(--space-4); margin-bottom: var(--space-5); }
.site-footer h3 { font-family: var(--font-display); color: var(--color-brass); font-size: 1rem; text-transform: uppercase; letter-spacing: 0.08em; margin-bottom: var(--space-2); }
.site-footer a { text-decoration: none; }
.site-footer a:hover { color: var(--color-brass); }
.site-footer__logo { width: 48px; height: 48px; margin-bottom: var(--space-2); filter: invert(1); }
.site-footer__name { font-size: 1.5rem; margin-bottom: 0; }
.site-footer__legal { font-size: 0.8rem; opacity: 0.7; border-top: 1px solid rgba(243,240,232,0.15); padding-top: var(--space-3); }

@media (max-width: 860px) {
  .site-footer__grid { grid-template-columns: 1fr 1fr; }
}

/* Stub placeholders */
.stub-panel {
  border: 2px dashed var(--color-sand-tan); border-radius: var(--radius-md);
  padding: var(--space-5); text-align: center; color: #6b6456;
}
.stub-panel__label { color: var(--color-brass); letter-spacing: 0.1em; margin-bottom: var(--space-2); }
.stub-grid { display: grid; grid-template-columns: repeat(auto-fit, minmax(200px, 1fr)); gap: var(--space-3); }
.stub-box {
  border: 2px dashed var(--color-sand-tan); border-radius: var(--radius-md); aspect-ratio: 4/3;
  display: flex; align-items: center; justify-content: center; color: #6b6456;
  font-family: var(--font-mono); font-size: 0.85rem; text-align: center; padding: var(--space-2);
}
```

- [ ] **Step 4: Write `js/main.js`**

```js
const navToggle = document.getElementById('navToggle');
const navLinks = document.getElementById('navLinks');

if (navToggle && navLinks) {
  navToggle.addEventListener('click', () => {
    const isOpen = navLinks.classList.toggle('nav__links--open');
    navToggle.setAttribute('aria-expanded', String(isOpen));
  });
}
```

- [ ] **Step 5: Verify token/class names are all present**

Run (Bash):
```
grep -c "color-hull-green\|color-brass\|color-chart-navy" "C:\Users\prsja\projects\lionmarineservice\css\tokens.css"
grep -c "plate-card\|site-footer\|nav__toggle" "C:\Users\prsja\projects\lionmarineservice\css\components.css"
```
Expected: both commands print a number >= 3 (no errors, matches found).

- [ ] **Step 6: Commit**

```bash
git add css/tokens.css css/base.css css/components.css js/main.js
git commit -m "Add design tokens, base styles, and shared nav/footer/card/form components"
```

---

### Task 2: Home page

**Files:**
- Create: `css/pages.css` (hero, trust-bar, info-strip, cta-banner rules only in this task)
- Create: `index.html`

**Interfaces:**
- Consumes: all classes/JS from Task 1.
- Produces: `.hero`, `.hero__img`, `.hero__overlay`, `.hero__content`, `.hero__headline`, `.hero__sub`, `.hero__actions`, `.trust-bar`, `.trust-bar__item`, `.info-strip`, `.info-strip__col`, `.cta-banner`, `.cta-banner__grid` — consumed by Task 4 (Motors reuses `.cta-banner`).

- [ ] **Step 1: Write hero/trust-bar/info-strip/cta-banner rules into `css/pages.css`**

```css
/* Hero */
.hero { position: relative; min-height: 640px; display: flex; align-items: flex-end; color: var(--color-white); overflow: hidden; }
.hero__img { position: absolute; inset: 0; width: 100%; height: 100%; object-fit: cover; }
.hero__overlay { position: absolute; inset: 0; background: linear-gradient(180deg, rgba(14,24,17,0.35) 0%, rgba(14,24,17,0.85) 100%); }
.hero__content { position: relative; padding-bottom: var(--space-6); }
.hero__headline { color: var(--color-white); margin-bottom: var(--space-2); }
.hero__sub { font-size: 1.15rem; max-width: 42ch; margin-bottom: var(--space-4); }
.hero__actions { display: flex; gap: var(--space-3); flex-wrap: wrap; }

/* Trust bar */
.trust-bar {
  display: flex; justify-content: center; gap: var(--space-5); flex-wrap: wrap;
  padding: var(--space-4) var(--space-3); border-bottom: 1px solid var(--color-sand-tan);
  font-size: 0.8rem; letter-spacing: 0.06em; color: var(--color-hull-green); margin: 0;
}

/* Info strip */
.info-strip { display: grid; grid-template-columns: 1fr 1fr; gap: var(--space-5); padding: var(--space-6) var(--space-3); }
@media (max-width: 760px) { .info-strip { grid-template-columns: 1fr; } }

/* CTA banner */
.cta-banner { background: var(--color-hull-green); color: var(--color-white); padding: var(--space-6) var(--space-3); text-align: center; }
.cta-banner h2 { color: var(--color-white); }
.cta-banner__grid { max-width: 640px; margin: 0 auto; }
.cta-banner .btn--primary { background: var(--color-brass); border-color: var(--color-brass); }
.cta-banner .btn--primary:hover { background: var(--color-white); color: var(--color-engine-black); border-color: var(--color-white); }
```

- [ ] **Step 2: Write `index.html`**

```html
<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Lion Marine | Outboard Sales &amp; Repair — Honolulu, HI</title>
<meta name="description" content="Mercury outboard dealer, engine repowering, diagnostics, marine electronics and audio, and solar installs. Mobile service across the Hawaiian Islands.">
<link rel="icon" href="assets/images/logo-lion.png">
<link rel="preconnect" href="https://fonts.googleapis.com">
<link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>
<link href="https://fonts.googleapis.com/css2?family=Big+Shoulders+Display:wght@700;900&family=IBM+Plex+Mono:wght@500;600&family=IBM+Plex+Sans:wght@400;500;600;700&display=swap" rel="stylesheet">
<link rel="stylesheet" href="css/tokens.css">
<link rel="stylesheet" href="css/base.css">
<link rel="stylesheet" href="css/components.css">
<link rel="stylesheet" href="css/pages.css">
</head>
<body>
<header class="site-header">
  <div class="nav container">
    <a class="nav__brand" href="index.html">
      <img src="assets/images/logo-lion.png" alt="Lion Marine" class="nav__logo">
      <span class="nav__brand-text">Lion Marine</span>
    </a>
    <button class="nav__toggle" id="navToggle" aria-expanded="false" aria-controls="navLinks">
      <span class="nav__toggle-bar"></span><span class="nav__toggle-bar"></span><span class="nav__toggle-bar"></span>
      <span class="sr-only">Menu</span>
    </button>
    <nav class="nav__links" id="navLinks">
      <a href="index.html" aria-current="page">Home</a>
      <a href="services.html">Services</a>
      <a href="motors.html">Motors</a>
      <a href="about.html">About</a>
      <a href="contact.html">Contact</a>
    </nav>
    <a class="nav__phone" href="tel:+18083006894">(808) 300-6894</a>
  </div>
</header>

<main>
  <section class="hero">
    <img src="assets/images/hero-boat-2.jpg" alt="Twin Yamaha outboards on a Hoku 27 center console at the dock" class="hero__img">
    <div class="hero__overlay"></div>
    <div class="hero__content container">
      <span class="eyebrow font-mono">HONOLULU, HI &mdash; KEEHI MARINE CENTER</span>
      <h1 class="hero__headline">Outboard Sales &amp; Repair</h1>
      <p class="hero__sub">Mercury dealer. Mobile service across the Hawaiian Islands.</p>
      <div class="hero__actions">
        <a href="tel:+18083006894" class="btn btn--primary">Call (808) 300-6894</a>
        <a href="services.html" class="btn btn--ghost">Get a Quote</a>
      </div>
    </div>
  </section>

  <p class="trust-bar">
    <span class="trust-bar__item font-mono">MERCURY OUTBOARD DEALER</span>
    <span class="trust-bar__item font-mono">RAYMARINE DEALER</span>
    <span class="trust-bar__item font-mono">FUSION AUDIO DEALER</span>
  </p>

  <section class="info-strip container">
    <div class="info-strip__col">
      <h2>Based in Honolulu</h2>
      <p>24 Sand Island Access Rd, Honolulu, HI 96819 (Keehi Marine Center). Offering mobile service to all the Hawaiian Islands.</p>
    </div>
    <div class="info-strip__col">
      <h2>Hours</h2>
      <p class="font-mono">Mon&ndash;Fri &nbsp; 8AM&ndash;5PM</p>
      <p>For emergencies call (808) 300-6894 and leave a detailed message. After-hours emergency calls are charged an emergency service fee.</p>
    </div>
  </section>

  <section class="section container">
    <h2 class="section__heading">Services</h2>
    <div class="plate-grid">
      <a class="plate-card" href="services.html">
        <span class="rivet rivet--tl"></span><span class="rivet rivet--tr"></span><span class="rivet rivet--bl"></span><span class="rivet rivet--br"></span>
        <span class="plate-card__code">SVC&mdash;01 REPOWER</span>
        <h3 class="plate-card__title">Repowering</h3>
        <p class="plate-card__body">Full engine repower for your existing hull.</p>
      </a>
      <a class="plate-card" href="services.html">
        <span class="rivet rivet--tl"></span><span class="rivet rivet--tr"></span><span class="rivet rivet--bl"></span><span class="rivet rivet--br"></span>
        <span class="plate-card__code">SVC&mdash;02 SERVICE</span>
        <h3 class="plate-card__title">Outboard Engine Service</h3>
        <p class="plate-card__body">Diagnostics, repair, and parts for Mercury outboards.</p>
      </a>
      <a class="plate-card" href="services.html">
        <span class="rivet rivet--tl"></span><span class="rivet rivet--tr"></span><span class="rivet rivet--bl"></span><span class="rivet rivet--br"></span>
        <span class="plate-card__code">SVC&mdash;03 ELECTRONICS</span>
        <h3 class="plate-card__title">Electronics &amp; Audio</h3>
        <p class="plate-card__body">Raymarine electronics and Fusion marine audio installs.</p>
      </a>
      <a class="plate-card" href="services.html">
        <span class="rivet rivet--tl"></span><span class="rivet rivet--tr"></span><span class="rivet rivet--bl"></span><span class="rivet rivet--br"></span>
        <span class="plate-card__code">SVC&mdash;04 SOLAR</span>
        <h3 class="plate-card__title">Solar</h3>
        <p class="plate-card__body">Marine solar systems for auxiliary power.</p>
      </a>
    </div>
  </section>

  <section class="cta-banner">
    <div class="container cta-banner__grid">
      <h2>Get Pre-Qualified Today</h2>
      <p>For new outboards and repowers &mdash; check financing options before you buy.</p>
      <a href="contact.html" class="btn btn--primary">Contact Us</a>
    </div>
  </section>
</main>

<footer class="site-footer">
  <div class="container site-footer__grid">
    <div class="site-footer__brand">
      <img src="assets/images/logo-lion.png" alt="" class="site-footer__logo">
      <p class="font-display site-footer__name">Lion Marine</p>
      <p>Outboard Sales and Repair</p>
    </div>
    <div class="site-footer__col">
      <h3>Hours</h3>
      <p class="font-mono">Mon&ndash;Fri, 8AM&ndash;5PM</p>
      <p>After-hours emergency calls accepted (additional charge).</p>
    </div>
    <div class="site-footer__col">
      <h3>Contact</h3>
      <p><a href="tel:+18083006894" class="font-mono">(808) 300-6894</a></p>
      <p><a href="mailto:Lionmarineservice@gmail.com">Lionmarineservice@gmail.com</a></p>
    </div>
    <div class="site-footer__col">
      <h3>Location</h3>
      <p>24 Sand Island Access Rd<br>Honolulu, HI 96819<br>(Keehi Marine Center)</p>
      <p>P.O. Box 8174<br>Honolulu, HI 96830</p>
    </div>
  </div>
  <p class="site-footer__legal container">&copy; 2026 Lion Marine. Mobile service available throughout the Hawaiian Islands.</p>
</footer>

<script src="js/main.js"></script>
</body>
</html>
```

- [ ] **Step 3: Verify in browser**

Use `mcp__plugin_chrome-devtools-mcp_chrome-devtools__navigate_page` with `url: "file:///C:/Users/prsja/projects/lionmarineservice/index.html"`, then `evaluate_script`:

```js
() => ({
  headline: document.querySelector('.hero__headline').textContent.trim(),
  phoneHref: document.querySelector('.nav__phone').getAttribute('href'),
  cardCount: document.querySelectorAll('.plate-card').length,
  scrollWidth: document.documentElement.scrollWidth,
})
```
Expected: `headline` is `"Outboard Sales & Repair"`, `phoneHref` is `"tel:+18083006894"`, `cardCount` is `4`.

Then resize to 375×800 and re-run the same script. Expected: `scrollWidth <= 376` (no horizontal overflow). Repeat at 1920×1080; expected `scrollWidth <= 1921`.

Take a full-page screenshot (`take_screenshot`, `fullPage: true`) at 1440×900 and read it back to confirm the hero photo, headline, service cards, and footer render as designed (no clipped content, no giant empty gap).

- [ ] **Step 4: Commit**

```bash
git add css/pages.css index.html
git commit -m "Add home page: hero, trust bar, info strip, services teaser, CTA banner"
```

---

### Task 3: Services page

**Files:**
- Create: `services.html`

**Interfaces:**
- Consumes: `.plate-grid`, `.plate-card`, `.form`, `.form__row`, `.form__field`, `.btn` from Task 1; header/footer markup pattern from Task 2.

- [ ] **Step 1: Write `services.html`**

```html
<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Services | Lion Marine</title>
<meta name="description" content="Repowering, outboard engine diagnostics and repair, marine electronics and audio installation, and solar for boats in Honolulu, HI.">
<link rel="icon" href="assets/images/logo-lion.png">
<link rel="preconnect" href="https://fonts.googleapis.com">
<link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>
<link href="https://fonts.googleapis.com/css2?family=Big+Shoulders+Display:wght@700;900&family=IBM+Plex+Mono:wght@500;600&family=IBM+Plex+Sans:wght@400;500;600;700&display=swap" rel="stylesheet">
<link rel="stylesheet" href="css/tokens.css">
<link rel="stylesheet" href="css/base.css">
<link rel="stylesheet" href="css/components.css">
<link rel="stylesheet" href="css/pages.css">
</head>
<body>
<header class="site-header">
  <div class="nav container">
    <a class="nav__brand" href="index.html">
      <img src="assets/images/logo-lion.png" alt="Lion Marine" class="nav__logo">
      <span class="nav__brand-text">Lion Marine</span>
    </a>
    <button class="nav__toggle" id="navToggle" aria-expanded="false" aria-controls="navLinks">
      <span class="nav__toggle-bar"></span><span class="nav__toggle-bar"></span><span class="nav__toggle-bar"></span>
      <span class="sr-only">Menu</span>
    </button>
    <nav class="nav__links" id="navLinks">
      <a href="index.html">Home</a>
      <a href="services.html" aria-current="page">Services</a>
      <a href="motors.html">Motors</a>
      <a href="about.html">About</a>
      <a href="contact.html">Contact</a>
    </nav>
    <a class="nav__phone" href="tel:+18083006894">(808) 300-6894</a>
  </div>
</header>

<main>
  <section class="section container">
    <h1 class="section__heading">We&rsquo;re here to assist having the best boating experience</h1>
    <div class="plate-grid">
      <div class="plate-card">
        <span class="rivet rivet--tl"></span><span class="rivet rivet--tr"></span><span class="rivet rivet--bl"></span><span class="rivet rivet--br"></span>
        <img src="assets/images/svc-repowering.jpeg" alt="Boat on a trailer during an outboard repower">
        <span class="plate-card__code">SVC&mdash;01 REPOWER</span>
        <h3 class="plate-card__title">Repowering</h3>
        <p class="plate-card__body">Full engine repower for your existing hull.</p>
      </div>
      <div class="plate-card">
        <span class="rivet rivet--tl"></span><span class="rivet rivet--tr"></span><span class="rivet rivet--bl"></span><span class="rivet rivet--br"></span>
        <img src="assets/images/svc-outboard.jpeg" alt="Technician servicing an outboard propeller">
        <span class="plate-card__code">SVC&mdash;02 SERVICE</span>
        <h3 class="plate-card__title">Outboard Engine Services</h3>
        <p class="plate-card__body">Diagnostics, repair, and parts for Mercury outboards.</p>
      </div>
      <div class="plate-card">
        <span class="rivet rivet--tl"></span><span class="rivet rivet--tr"></span><span class="rivet rivet--bl"></span><span class="rivet rivet--br"></span>
        <img src="assets/images/svc-electronics.jpeg" alt="Marine electronics">
        <span class="plate-card__code">SVC&mdash;03 ELECTRONICS</span>
        <h3 class="plate-card__title">Electronics and Audio Installations</h3>
        <p class="plate-card__body">Raymarine electronics and Fusion marine audio installs.</p>
      </div>
      <div class="plate-card">
        <span class="rivet rivet--tl"></span><span class="rivet rivet--tr"></span><span class="rivet rivet--bl"></span><span class="rivet rivet--br"></span>
        <img src="assets/images/svc-solar.jpeg" alt="Marine solar panel installation">
        <span class="plate-card__code">SVC&mdash;04 SOLAR</span>
        <h3 class="plate-card__title">Solar</h3>
        <p class="plate-card__body">Marine solar systems for auxiliary power.</p>
      </div>
    </div>
  </section>

  <section class="section container">
    <h2 class="section__heading">Get a Free Quote</h2>
    <p style="text-align:center; max-width: 640px; margin: 0 auto var(--space-4);">Please feel free to contact us with any questions and we will respond promptly.</p>
    <form class="form" action="#" method="post">
      <div class="form__row">
        <div class="form__field"><label for="svc-first">First Name</label><input id="svc-first" name="firstName" type="text"></div>
        <div class="form__field"><label for="svc-last">Last Name</label><input id="svc-last" name="lastName" type="text"></div>
      </div>
      <div class="form__field"><label for="svc-email">Email *</label><input id="svc-email" name="email" type="email" required></div>
      <div class="form__field"><label for="svc-phone">Phone #</label><input id="svc-phone" name="phone" type="tel"></div>
      <div class="form__field"><label for="svc-serial">Engine Serial #</label><input id="svc-serial" name="engineSerial" type="text"></div>
      <div class="form__field"><label for="svc-message">Service Requesting</label><textarea id="svc-message" name="serviceRequesting" placeholder="Please describe the service you would like performed or size of engine you would like to purchase"></textarea></div>
      <button type="submit" class="btn btn--primary">Send</button>
    </form>
  </section>
</main>

<footer class="site-footer">
  <div class="container site-footer__grid">
    <div class="site-footer__brand">
      <img src="assets/images/logo-lion.png" alt="" class="site-footer__logo">
      <p class="font-display site-footer__name">Lion Marine</p>
      <p>Outboard Sales and Repair</p>
    </div>
    <div class="site-footer__col">
      <h3>Hours</h3>
      <p class="font-mono">Mon&ndash;Fri, 8AM&ndash;5PM</p>
      <p>After-hours emergency calls accepted (additional charge).</p>
    </div>
    <div class="site-footer__col">
      <h3>Contact</h3>
      <p><a href="tel:+18083006894" class="font-mono">(808) 300-6894</a></p>
      <p><a href="mailto:Lionmarineservice@gmail.com">Lionmarineservice@gmail.com</a></p>
    </div>
    <div class="site-footer__col">
      <h3>Location</h3>
      <p>24 Sand Island Access Rd<br>Honolulu, HI 96819<br>(Keehi Marine Center)</p>
      <p>P.O. Box 8174<br>Honolulu, HI 96830</p>
    </div>
  </div>
  <p class="site-footer__legal container">&copy; 2026 Lion Marine. Mobile service available throughout the Hawaiian Islands.</p>
</footer>

<script src="js/main.js"></script>
</body>
</html>
```

- [ ] **Step 2: Verify in browser**

Navigate to `file:///C:/Users/prsja/projects/lionmarineservice/services.html`, then `evaluate_script`:

```js
() => ({
  cardCount: document.querySelectorAll('.plate-card').length,
  emailRequired: document.getElementById('svc-email').required,
  scrollWidth: document.documentElement.scrollWidth,
})
```
Expected: `cardCount` is `4`, `emailRequired` is `true`. Resize to 375×800 and confirm `scrollWidth <= 376`.

- [ ] **Step 3: Commit**

```bash
git add services.html
git commit -m "Add services page with data-plate cards and free quote form"
```

---

### Task 4: Motors page

**Files:**
- Create: `motors.html`
- Modify: `css/pages.css` (append motor-dial rules)

**Interfaces:**
- Consumes: `.cta-banner` from Task 2, `.plate-grid`/`.plate-card` from Task 1.
- Produces: `.motor-dial`, `.motor-dial__range`, `.motor-dial__unit`, `.motor-note`.

- [ ] **Step 1: Append motor-dial rules to `css/pages.css`**

```css
/* Motor dial */
.motor-dial {
  width: 96px; height: 96px; border-radius: 50%; border: 3px solid var(--color-brass);
  display: flex; flex-direction: column; align-items: center; justify-content: center;
  margin-bottom: var(--space-3); font-family: var(--font-mono);
}
.motor-dial__range { font-weight: 700; font-size: 1rem; }
.motor-dial__unit { font-size: 0.65rem; letter-spacing: 0.08em; }
.motor-note { font-size: 0.85rem; color: var(--color-hull-green); font-weight: 600; margin-top: var(--space-2); }
```

- [ ] **Step 2: Write `motors.html`**

```html
<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Motors | Lion Marine</title>
<meta name="description" content="Mercury outboard motor families sold by Lion Marine in Honolulu, HI. Call or email for current pricing.">
<link rel="icon" href="assets/images/logo-lion.png">
<link rel="preconnect" href="https://fonts.googleapis.com">
<link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>
<link href="https://fonts.googleapis.com/css2?family=Big+Shoulders+Display:wght@700;900&family=IBM+Plex+Mono:wght@500;600&family=IBM+Plex+Sans:wght@400;500;600;700&display=swap" rel="stylesheet">
<link rel="stylesheet" href="css/tokens.css">
<link rel="stylesheet" href="css/base.css">
<link rel="stylesheet" href="css/components.css">
<link rel="stylesheet" href="css/pages.css">
</head>
<body>
<header class="site-header">
  <div class="nav container">
    <a class="nav__brand" href="index.html">
      <img src="assets/images/logo-lion.png" alt="Lion Marine" class="nav__logo">
      <span class="nav__brand-text">Lion Marine</span>
    </a>
    <button class="nav__toggle" id="navToggle" aria-expanded="false" aria-controls="navLinks">
      <span class="nav__toggle-bar"></span><span class="nav__toggle-bar"></span><span class="nav__toggle-bar"></span>
      <span class="sr-only">Menu</span>
    </button>
    <nav class="nav__links" id="navLinks">
      <a href="index.html">Home</a>
      <a href="services.html">Services</a>
      <a href="motors.html" aria-current="page">Motors</a>
      <a href="about.html">About</a>
      <a href="contact.html">Contact</a>
    </nav>
    <a class="nav__phone" href="tel:+18083006894">(808) 300-6894</a>
  </div>
</header>

<main>
  <section class="section container">
    <h1 class="section__heading">2026 Mercury Outboards</h1>
    <div class="plate-grid">
      <div class="plate-card">
        <span class="rivet rivet--tl"></span><span class="rivet rivet--tr"></span><span class="rivet rivet--bl"></span><span class="rivet rivet--br"></span>
        <div class="motor-dial"><span class="motor-dial__range">2.5&ndash;20</span><span class="motor-dial__unit">HP</span></div>
        <h3 class="plate-card__title">Portable FourStroke</h3>
        <p class="plate-card__body">Lightweight portables for tenders, small skiffs, and kayaks.</p>
        <p class="motor-note">Call or email for current pricing</p>
      </div>
      <div class="plate-card">
        <span class="rivet rivet--tl"></span><span class="rivet rivet--tr"></span><span class="rivet rivet--bl"></span><span class="rivet rivet--br"></span>
        <div class="motor-dial"><span class="motor-dial__range">25&ndash;115</span><span class="motor-dial__unit">HP</span></div>
        <h3 class="plate-card__title">Mid-Range FourStroke</h3>
        <p class="plate-card__body">The everyday workhorse range for center consoles and runabouts.</p>
        <p class="motor-note">Call or email for current pricing</p>
      </div>
      <div class="plate-card">
        <span class="rivet rivet--tl"></span><span class="rivet rivet--tr"></span><span class="rivet rivet--bl"></span><span class="rivet rivet--br"></span>
        <div class="motor-dial"><span class="motor-dial__range">150&ndash;300</span><span class="motor-dial__unit">HP</span></div>
        <h3 class="plate-card__title">Pro XS / V6-V8 Performance</h3>
        <p class="plate-card__body">High-performance power for offshore and tournament boats.</p>
        <p class="motor-note">Call or email for current pricing</p>
      </div>
      <div class="plate-card">
        <span class="rivet rivet--tl"></span><span class="rivet rivet--tr"></span><span class="rivet rivet--bl"></span><span class="rivet rivet--br"></span>
        <div class="motor-dial"><span class="motor-dial__range">40&ndash;115</span><span class="motor-dial__unit">HP</span></div>
        <h3 class="plate-card__title">SeaPro Commercial</h3>
        <p class="plate-card__body">Built for daily commercial and charter use.</p>
        <p class="motor-note">Call or email for current pricing</p>
      </div>
    </div>
  </section>

  <section class="cta-banner">
    <div class="container cta-banner__grid">
      <h2>Get Pre-Qualified Today</h2>
      <p>For new outboards and repowers &mdash; check financing options before you buy.</p>
      <a href="tel:+18083006894" class="btn btn--primary">Call (808) 300-6894</a>
    </div>
  </section>
</main>

<footer class="site-footer">
  <div class="container site-footer__grid">
    <div class="site-footer__brand">
      <img src="assets/images/logo-lion.png" alt="" class="site-footer__logo">
      <p class="font-display site-footer__name">Lion Marine</p>
      <p>Outboard Sales and Repair</p>
    </div>
    <div class="site-footer__col">
      <h3>Hours</h3>
      <p class="font-mono">Mon&ndash;Fri, 8AM&ndash;5PM</p>
      <p>After-hours emergency calls accepted (additional charge).</p>
    </div>
    <div class="site-footer__col">
      <h3>Contact</h3>
      <p><a href="tel:+18083006894" class="font-mono">(808) 300-6894</a></p>
      <p><a href="mailto:Lionmarineservice@gmail.com">Lionmarineservice@gmail.com</a></p>
    </div>
    <div class="site-footer__col">
      <h3>Location</h3>
      <p>24 Sand Island Access Rd<br>Honolulu, HI 96819<br>(Keehi Marine Center)</p>
      <p>P.O. Box 8174<br>Honolulu, HI 96830</p>
    </div>
  </div>
  <p class="site-footer__legal container">&copy; 2026 Lion Marine. Mobile service available throughout the Hawaiian Islands.</p>
</footer>

<script src="js/main.js"></script>
</body>
</html>
```

- [ ] **Step 3: Verify in browser**

Navigate to `file:///C:/Users/prsja/projects/lionmarineservice/motors.html`, `evaluate_script`:

```js
() => ({
  dialCount: document.querySelectorAll('.motor-dial').length,
  hasCallCta: !!document.querySelector('.cta-banner a[href="tel:+18083006894"]'),
  scrollWidth: document.documentElement.scrollWidth,
})
```
Expected: `dialCount` is `4`, `hasCallCta` is `true`. Resize to 1920×1080 and confirm `scrollWidth <= 1921`.

- [ ] **Step 4: Commit**

```bash
git add motors.html css/pages.css
git commit -m "Add motors page with HP-range category cards and call-for-pricing CTA"
```

---

### Task 5: Contact page

**Files:**
- Create: `contact.html`
- Modify: `css/pages.css` (append contact-grid rules)

**Interfaces:**
- Consumes: `.form`, `.form__field` from Task 1.
- Produces: `.contact-grid`, `.contact-panel`, `.map-embed`.

- [ ] **Step 1: Append contact rules to `css/pages.css`**

```css
/* Contact */
.contact-grid { display: grid; grid-template-columns: 1fr 1fr; gap: var(--space-5); padding: var(--space-6) var(--space-3); }
@media (max-width: 860px) { .contact-grid { grid-template-columns: 1fr; } }
.contact-panel { background: var(--color-chart-navy); color: var(--color-plate-paper); border-radius: var(--radius-md); padding: var(--space-4); }
.contact-panel h2 { color: var(--color-brass); font-size: 1.1rem; text-transform: uppercase; letter-spacing: 0.08em; }
.contact-panel section + section { margin-top: var(--space-4); }
.map-embed { width: 100%; border: 0; border-radius: var(--radius-md); min-height: 360px; }
```

- [ ] **Step 2: Write `contact.html`**

```html
<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Contact | Lion Marine</title>
<meta name="description" content="Contact Lion Marine in Honolulu, HI: address, phone, email, hours, and a map to Keehi Marine Center.">
<link rel="icon" href="assets/images/logo-lion.png">
<link rel="preconnect" href="https://fonts.googleapis.com">
<link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>
<link href="https://fonts.googleapis.com/css2?family=Big+Shoulders+Display:wght@700;900&family=IBM+Plex+Mono:wght@500;600&family=IBM+Plex+Sans:wght@400;500;600;700&display=swap" rel="stylesheet">
<link rel="stylesheet" href="css/tokens.css">
<link rel="stylesheet" href="css/base.css">
<link rel="stylesheet" href="css/components.css">
<link rel="stylesheet" href="css/pages.css">
</head>
<body>
<header class="site-header">
  <div class="nav container">
    <a class="nav__brand" href="index.html">
      <img src="assets/images/logo-lion.png" alt="Lion Marine" class="nav__logo">
      <span class="nav__brand-text">Lion Marine</span>
    </a>
    <button class="nav__toggle" id="navToggle" aria-expanded="false" aria-controls="navLinks">
      <span class="nav__toggle-bar"></span><span class="nav__toggle-bar"></span><span class="nav__toggle-bar"></span>
      <span class="sr-only">Menu</span>
    </button>
    <nav class="nav__links" id="navLinks">
      <a href="index.html">Home</a>
      <a href="services.html">Services</a>
      <a href="motors.html">Motors</a>
      <a href="about.html">About</a>
      <a href="contact.html" aria-current="page">Contact</a>
    </nav>
    <a class="nav__phone" href="tel:+18083006894">(808) 300-6894</a>
  </div>
</header>

<main>
  <h1 class="section__heading" style="padding-top: var(--space-6);">Contact Us</h1>
  <div class="contact-grid container">
    <div class="contact-panel">
      <section>
        <h2>Address</h2>
        <p>24 Sand Island Access Rd.<br>Honolulu, HI 96819</p>
        <p>Mailing Address:<br>PO Box 8174<br>Honolulu, HI 96830</p>
      </section>
      <section>
        <h2>Contact</h2>
        <p class="font-mono"><a href="tel:+18083006894">(808) 300-6894</a></p>
        <p><a href="mailto:LionMarineService@gmail.com">LionMarineService@gmail.com</a></p>
      </section>
      <section>
        <h2>Opening Hours</h2>
        <p class="font-mono">Mon &ndash; Fri<br>8:00am &ndash; 5:00pm</p>
      </section>
    </div>
    <form class="form" action="#" method="post">
      <div class="form__row">
        <div class="form__field"><label for="ct-first">First Name</label><input id="ct-first" name="firstName" type="text"></div>
        <div class="form__field"><label for="ct-last">Last Name</label><input id="ct-last" name="lastName" type="text"></div>
      </div>
      <div class="form__field"><label for="ct-email">Email *</label><input id="ct-email" name="email" type="email" required></div>
      <div class="form__field"><label for="ct-message">Message</label><textarea id="ct-message" name="message"></textarea></div>
      <button type="submit" class="btn btn--primary">Send</button>
    </form>
  </div>
  <div class="container" style="padding-bottom: var(--space-6);">
    <iframe class="map-embed" title="Lion Marine location map" loading="lazy" src="https://www.google.com/maps?q=24+Sand+Island+Access+Rd,+Honolulu,+HI+96819&output=embed"></iframe>
  </div>
</main>

<footer class="site-footer">
  <div class="container site-footer__grid">
    <div class="site-footer__brand">
      <img src="assets/images/logo-lion.png" alt="" class="site-footer__logo">
      <p class="font-display site-footer__name">Lion Marine</p>
      <p>Outboard Sales and Repair</p>
    </div>
    <div class="site-footer__col">
      <h3>Hours</h3>
      <p class="font-mono">Mon&ndash;Fri, 8AM&ndash;5PM</p>
      <p>After-hours emergency calls accepted (additional charge).</p>
    </div>
    <div class="site-footer__col">
      <h3>Contact</h3>
      <p><a href="tel:+18083006894" class="font-mono">(808) 300-6894</a></p>
      <p><a href="mailto:Lionmarineservice@gmail.com">Lionmarineservice@gmail.com</a></p>
    </div>
    <div class="site-footer__col">
      <h3>Location</h3>
      <p>24 Sand Island Access Rd<br>Honolulu, HI 96819<br>(Keehi Marine Center)</p>
      <p>P.O. Box 8174<br>Honolulu, HI 96830</p>
    </div>
  </div>
  <p class="site-footer__legal container">&copy; 2026 Lion Marine. Mobile service available throughout the Hawaiian Islands.</p>
</footer>

<script src="js/main.js"></script>
</body>
</html>
```

- [ ] **Step 3: Verify in browser**

Navigate to `file:///C:/Users/prsja/projects/lionmarineservice/contact.html`, `evaluate_script`:

```js
() => ({
  mapSrc: document.querySelector('.map-embed').getAttribute('src'),
  emailRequired: document.getElementById('ct-email').required,
  scrollWidth: document.documentElement.scrollWidth,
})
```
Expected: `mapSrc` contains `"Sand+Island"`, `emailRequired` is `true`. Resize to 375×800 and confirm `scrollWidth <= 376`.

- [ ] **Step 4: Commit**

```bash
git add contact.html css/pages.css
git commit -m "Add contact page with info panel, form, and map embed"
```

---

### Task 6: About page (placeholder stubs)

**Files:**
- Create: `about.html`

**Interfaces:**
- Consumes: `.stub-panel`, `.stub-panel__label`, `.stub-grid`, `.stub-box` from Task 1.

- [ ] **Step 1: Write `about.html`**

```html
<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>About | Lion Marine</title>
<meta name="description" content="About Lion Marine, an outboard sales and repair shop in Honolulu, HI.">
<link rel="icon" href="assets/images/logo-lion.png">
<link rel="preconnect" href="https://fonts.googleapis.com">
<link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>
<link href="https://fonts.googleapis.com/css2?family=Big+Shoulders+Display:wght@700;900&family=IBM+Plex+Mono:wght@500;600&family=IBM+Plex+Sans:wght@400;500;600;700&display=swap" rel="stylesheet">
<link rel="stylesheet" href="css/tokens.css">
<link rel="stylesheet" href="css/base.css">
<link rel="stylesheet" href="css/components.css">
<link rel="stylesheet" href="css/pages.css">
</head>
<body>
<header class="site-header">
  <div class="nav container">
    <a class="nav__brand" href="index.html">
      <img src="assets/images/logo-lion.png" alt="Lion Marine" class="nav__logo">
      <span class="nav__brand-text">Lion Marine</span>
    </a>
    <button class="nav__toggle" id="navToggle" aria-expanded="false" aria-controls="navLinks">
      <span class="nav__toggle-bar"></span><span class="nav__toggle-bar"></span><span class="nav__toggle-bar"></span>
      <span class="sr-only">Menu</span>
    </button>
    <nav class="nav__links" id="navLinks">
      <a href="index.html">Home</a>
      <a href="services.html">Services</a>
      <a href="motors.html">Motors</a>
      <a href="about.html" aria-current="page">About</a>
      <a href="contact.html">Contact</a>
    </nav>
    <a class="nav__phone" href="tel:+18083006894">(808) 300-6894</a>
  </div>
</header>

<main>
  <section class="section container">
    <span class="stub-panel__label font-mono" style="display:block; text-align:center;">COMING SOON</span>
    <div class="stub-panel">
      <h2>Our Story</h2>
      <p>This section is reserved for Lion Marine&rsquo;s story &mdash; background, experience, and what makes the shop different. Add your own copy here.</p>
    </div>
  </section>

  <section class="section container">
    <h2 class="section__heading">Customer Reviews</h2>
    <div class="stub-panel">
      <span class="stub-panel__label font-mono">REVIEWS COMING SOON</span>
      <p>Customer reviews will appear here once added.</p>
    </div>
  </section>

  <section class="section container">
    <h2 class="section__heading">Project Gallery</h2>
    <div class="stub-grid">
      <div class="stub-box">Photo coming soon</div>
      <div class="stub-box">Photo coming soon</div>
      <div class="stub-box">Photo coming soon</div>
    </div>
  </section>
</main>

<footer class="site-footer">
  <div class="container site-footer__grid">
    <div class="site-footer__brand">
      <img src="assets/images/logo-lion.png" alt="" class="site-footer__logo">
      <p class="font-display site-footer__name">Lion Marine</p>
      <p>Outboard Sales and Repair</p>
    </div>
    <div class="site-footer__col">
      <h3>Hours</h3>
      <p class="font-mono">Mon&ndash;Fri, 8AM&ndash;5PM</p>
      <p>After-hours emergency calls accepted (additional charge).</p>
    </div>
    <div class="site-footer__col">
      <h3>Contact</h3>
      <p><a href="tel:+18083006894" class="font-mono">(808) 300-6894</a></p>
      <p><a href="mailto:Lionmarineservice@gmail.com">Lionmarineservice@gmail.com</a></p>
    </div>
    <div class="site-footer__col">
      <h3>Location</h3>
      <p>24 Sand Island Access Rd<br>Honolulu, HI 96819<br>(Keehi Marine Center)</p>
      <p>P.O. Box 8174<br>Honolulu, HI 96830</p>
    </div>
  </div>
  <p class="site-footer__legal container">&copy; 2026 Lion Marine. Mobile service available throughout the Hawaiian Islands.</p>
</footer>

<script src="js/main.js"></script>
</body>
</html>
```

- [ ] **Step 2: Verify in browser**

Navigate to `file:///C:/Users/prsja/projects/lionmarineservice/about.html`, `evaluate_script`:

```js
() => ({
  stubLabels: Array.from(document.querySelectorAll('.stub-panel__label')).map(e => e.textContent.trim()),
  stubBoxCount: document.querySelectorAll('.stub-box').length,
})
```
Expected: `stubLabels` includes `"COMING SOON"` and `"REVIEWS COMING SOON"`; `stubBoxCount` is `3`.

- [ ] **Step 3: Commit**

```bash
git add about.html
git commit -m "Add about page with clearly-labeled bio/testimonials/gallery placeholders"
```

---

### Task 7: Cross-page responsive QA

**Files:** none created; verification only.

- [ ] **Step 1: Check every page for horizontal overflow at 4 breakpoints**

For each of `index.html`, `services.html`, `motors.html`, `contact.html`, `about.html`:
1. `navigate_page` to `file:///C:/Users/prsja/projects/lionmarineservice/<page>`
2. For each width in `[375, 768, 1440, 1920]`: `resize_page` to that width (height 900), then `evaluate_script`:
```js
() => document.documentElement.scrollWidth
```
Expected: return value `<= width + 1` for every page/width combination. If any page overflows, find the offending element with:
```js
(w) => Array.from(document.querySelectorAll('*')).filter(el => el.scrollWidth > w).map(el => el.className).slice(0, 5)
```
(pass the failing width as `args`) and fix the CSS rule causing it before re-testing.

- [ ] **Step 2: Check nav consistency across all 5 pages**

On each page, `evaluate_script`:
```js
() => Array.from(document.querySelectorAll('.nav__links a')).map(a => a.textContent.trim())
```
Expected: exactly `["Home", "Services", "Motors", "About", "Contact"]` on every page, in that order.

- [ ] **Step 3: Take final full-page screenshots for visual sign-off**

For each page, `take_screenshot` with `fullPage: true` at 1440×900, saved to the session scratchpad, then read each one back to confirm: no clipped/cut-off content, no large dead whitespace gaps, data-plate cards render with visible rivets/brass top border, hero photo displays correctly on Home.

- [ ] **Step 4: Commit any fixes found during QA**

```bash
git add -A
git commit -m "Fix responsive issues found during cross-page QA"
```
(Skip this commit if QA found no issues to fix.)

---

### Task 8: Publish to GitHub Pages

**Files:** none created; deployment only.

- [ ] **Step 1: Confirm remote and current branch**

Run: `git -C "C:\Users\prsja\projects\lionmarineservice" remote -v && git -C "C:\Users\prsja\projects\lionmarineservice" branch`
Expected: `origin` points to `https://github.com/Thecavepeanut/lionmarine-ex.git`; note the current branch name (likely `master`).

- [ ] **Step 2: Push to the remote**

Run: `git -C "C:\Users\prsja\projects\lionmarineservice" push -u origin master`
Expected: push succeeds (may require the user to authenticate via `gh auth login` or a credential prompt if not already authenticated).

- [ ] **Step 3: Make the repo public (pre-authorized by the user) if it isn't already**

Run: `gh repo view Thecavepeanut/lionmarine-ex --json visibility`
If `visibility` is not `"PUBLIC"`, run: `gh repo edit Thecavepeanut/lionmarine-ex --visibility public --accept-visibility-change-consequences`

- [ ] **Step 4: Enable GitHub Pages on the `master` branch root**

Run: `gh api repos/Thecavepeanut/lionmarine-ex/pages -X POST -f "source[branch]=master" -f "source[path]=/"`
Expected: JSON response with a `"html_url"` field like `https://thecavepeanut.github.io/lionmarine-ex/`. If it returns a 409 (already enabled), run `gh api repos/Thecavepeanut/lionmarine-ex/pages` (GET) instead to confirm the existing config.

- [ ] **Step 5: Verify the live URL serves the site**

Run: `curl -s -o /dev/null -w "%{http_code}\n" https://thecavepeanut.github.io/lionmarine-ex/`
Expected: `200` (GitHub Pages builds can take 1-2 minutes after first enabling; if it returns `404`, wait ~60 seconds and retry rather than re-running the enable step).

- [ ] **Step 6: Report the live URL to the user**

No commit needed for this task (deployment only, no file changes).
