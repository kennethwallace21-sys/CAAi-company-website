# Catalyst Applied AI — Company Website

Marketing site for **Catalyst Applied AI (CAAi)** — governed, auditable AI deployed
inside the customer's environment (air-gapped to cloud).

Positioning: _"AI that runs where your data lives."_ Audience: federal/DoD, healthcare,
and financial buyers who can't let data leave their perimeter.

---

## ⚠️ Current status / known blocker (read first)

| Thing | State |
|-------|-------|
| Code in this repo | ✅ Current — homepage, lead form, social cards, SEO files all committed to `main` |
| GitHub Pages build | ✅ Builds & deploys from `main` (root). Files verified on `raw.githubusercontent.com`. |
| **Live apex `catalystappliedai.com`** | ❌ **Serving an OLD, different site** ("Practical AI Solutions for Business"). DNS/Cloudflare for the apex still points at another origin — **not** this GitHub Pages site. |

**To make this site go live, the apex DNS must point at GitHub Pages** (and any
conflicting Cloudflare Pages project / Worker route / proxied origin removed). Until
then, visitors see the old site. See **Deployment** below for the exact records.

Other open items: see [TODO](#todo--backlog).

---

## Pages in this repo

| File | Purpose | Where it deploys |
|------|---------|------------------|
| `index.html` | Main homepage (governed-AI pitch, 5 offerings, architecture, compliance, lead modal) | `catalystappliedai.com` (this repo → GitHub Pages) |
| `clerk.html` | CLERK — government records intelligence product page | `gov-products.catalystappliedai.com` (**served by `caai-ops`, not this repo** — see below) |
| `custom-models.html` | Custom Models product page | `custom-models.catalystappliedai.com` (**served by `caai-ops`**) |
| `command-center.html` | Command Center demo (React + Babel, in-browser) | served by `caai-ops` |
| `scooters-enterprise-deck.html` | Scooter's Coffee sales deck | static |
| `contact-form.js` | Shared contact modal used by the **subdomain** pages (posts to `/api/contact`) | served by `caai-ops` |
| `assets/` | Logos (`caai-mark.svg`, wordmarks), NVIDIA badge, **`og-cover.png/.svg`** (social share image) | |
| `robots.txt`, `sitemap.xml` | SEO | apex |
| `CNAME` | GitHub Pages custom domain (`catalystappliedai.com`) | apex |
| `.nojekyll` | Tells Pages to skip Jekyll and serve files as-is | apex |

> **Important:** `clerk.html`, `custom-models.html`, `command-center.html`, and
> `contact-form.js` in this repo are **migrated copies**. The live product subdomains
> are served by a separate FastAPI app — editing them here does **not** update those
> subdomains. See [Architecture](#architecture-where-each-page-actually-runs).

---

## Architecture — where each page actually runs

There are **two** deployment targets:

1. **Apex marketing site** (`catalystappliedai.com`) → **GitHub Pages**, from THIS repo
   (`kennethwallace21-sys/CAAi-company-website`, branch `main`, path `/`).
   Pushing to `main` is the deploy.

2. **Product subdomains** (`gov-products.*` → CLERK, `custom-models.*`) → a **FastAPI
   app** at `catalyst/ui/app.py` in the **`caai-ops`** repo
   (`kennethwallace21-sys/caai-ops`). It routes the homepage by `Host` header and
   mounts `ui/static/` at `/`. The product `.html` files live in `caai-ops/ui/static/`.
   _As of last check these subdomains returned HTTP 500/520 (app down/erroring)._

**Why the cross-page asset URLs are absolute:** social-card and favicon `<meta>` tags on
the subdomain pages point at `https://catalystappliedai.com/assets/...`. Because the apex
(this repo's Pages) serves those shared assets, they resolve from any subdomain with **no
per-subdomain copies needed** — once the apex DNS is fixed.

---

## Deployment

### Apex (this repo → GitHub Pages)

- Source: `main` branch, root. Configured via repo **Settings → Pages**.
- `CNAME` = `catalystappliedai.com`; `.nojekyll` present.
- **Required DNS at the registrar / Cloudflare** to point the apex at GitHub Pages:
  - `A` records → `185.199.108.153`, `185.199.109.153`, `185.199.110.153`, `185.199.111.153`
  - `AAAA` records → `2606:50c0:8000::153`, `2606:50c0:8001::153`, `2606:50c0:8002::153`, `2606:50c0:8003::153`
  - `www` → `CNAME` → `kennethwallace21-sys.github.io`
  - If using Cloudflare proxy (orange cloud), set SSL/TLS mode to **Full** and ensure no
    Cloudflare **Pages project / Worker route** is intercepting the apex. After changes,
    **purge the Cloudflare cache** (it aggressively caches — `CF-Cache-Status: HIT`).
- Verify the real origin any time with:
  `curl -s https://raw.githubusercontent.com/kennethwallace21-sys/CAAi-company-website/main/index.html | grep '<title>'`

### Subdomains (caai-ops)

Mirror any product-page edits into `caai-ops/ui/static/` and redeploy that app.

---

## Lead capture

The homepage demo form (modal in `index.html`) submits **client-side** to
[Web3Forms](https://web3forms.com) and emails submissions to the sales inbox.

- Access key lives in `index.html` as `WEB3FORMS_KEY` (Web3Forms keys are public/
  client-side by design).
- Fallback: if the key is left as the `REPLACE_...` placeholder, the form opens a
  pre-filled `mailto:` instead of failing.
- Note: Web3Forms' free plan **rejects server-side POSTs** (e.g. `curl`) — only real
  browser submissions work. Test from the live page, not the CLI.
- The subdomain pages use a different path: `contact-form.js` → `POST /api/contact`
  (handled by the `caai-ops` FastAPI backend).

---

## Updating the social share image

`assets/og-cover.png` (1200×630) is the link-preview image for LinkedIn/Slack/X/iMessage.
Edit the source `assets/og-cover.svg`, then rasterize with Node:

```bash
npm i @resvg/resvg-js
node -e 'const fs=require("fs"),{Resvg}=require("@resvg/resvg-js");const b="assets/";const png=new Resvg(fs.readFileSync(b+"og-cover.svg","utf8"),{fitTo:{mode:"width",value:1200},font:{loadSystemFonts:true}}).render().asPng();fs.writeFileSync(b+"og-cover.png",png);'
```

Validate previews at <https://www.opengraph.xyz>.

---

## Local development

No build step for the marketing site — it's static HTML/CSS/JS.

```bash
python -m http.server 8000   # then open http://localhost:8000
```

`command-center.html` is the exception: it loads React + Babel from a CDN and compiles
JSX in the browser (slow; see TODO).

---

## TODO / backlog

- [ ] **Point apex DNS at GitHub Pages** so this site actually goes live (see Deployment). _Highest priority._
- [ ] Investigate the **500/520** on `gov-products` / `custom-models` subdomains (caai-ops app).
- [ ] Mirror the social-card + favicon `<meta>` additions into `caai-ops/ui/static/` so the subdomain pages get them too.
- [ ] `command-center.html`: replace in-browser Babel with a small build step (Vite) for performance.
- [ ] Consider per-page OG images for the product subdomains.

---

_Last substantial update: 2026-06. Lead capture (Web3Forms), Open Graph / Twitter cards +
custom share image, favicon, JSON-LD, robots/sitemap, and Pages hardening (CNAME,
.nojekyll) added to the apex site._
