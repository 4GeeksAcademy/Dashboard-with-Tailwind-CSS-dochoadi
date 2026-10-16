# Context: Influencer Analytics Dashboard

This document captures **what this project must deliver** based on the original brief. It is the single reference for scope, constraints, data rules, and design decisions. The companion file [validation.md](validation.md) lists how each requirement is verified before delivery.

---

## 1. Project summary

A **single-page, static influencer analytics dashboard** built with **HTML + Tailwind CSS v4 only** (no JavaScript at all). It consolidates an influencer's affiliate results from **Instagram, TikTok, and YouTube** for three promoted products into one dark, premium-looking, fully responsive page.

The task is **not done** until every line of the final checklist (section 11 of this document, detailed in [validation.md](validation.md)) reports **PASS**.

### Problem statement
An influencer promotes affiliate products across three platforms and has no single view of results. They need one dashboard that answers:

1. How much commission revenue am I generating?
2. Which products generate the most revenue?
3. How well do my ads convert (conversions ÷ reach)?
4. Which platforms give the best return (revenue ÷ cost)?
5. What is the engagement rate by platform and by product?

---

## 2. Hard constraints (non-negotiable)

| Area | Rule |
|---|---|
| Languages | HTML and Tailwind CSS v4 only. **No JavaScript** of any kind (no React, Vue, Alpine, jQuery, vanilla JS). |
| Forbidden markup | Zero `<script>` tags, zero `<style>` tags, zero `style=""` attributes. |
| CSS files | Only `src/input.css` (source) and `dist/output.css` (generated). No other CSS files. |
| CDN | Zero CDN links. **Do not use `cdn.tailwindcss.com`** or `@tailwindcss/browser`. |
| Tailwind install | Via CLI: npm packages `tailwindcss` and `@tailwindcss/cli`. |
| `src/input.css` | Exactly one line: `@import "tailwindcss";` — no `@theme`, no custom CSS. (Exception: `@source "../index.html";` **only** if classes are missing from the build; document it in the README.) |
| No Tailwind v3 | No `tailwind.config.js`, no `@tailwind base/components/utilities` directives. |
| Colors | Default Tailwind palette + arbitrary values (e.g. `bg-[#0B0F1A]`, `bg-[conic-gradient(...)]`, `bg-[radial-gradient(...)]`). |
| No motion | No `animate-*`, `transition*`, `duration-*`, `motion-*`, keyframes, or hover movement (`translate`, `scale`). |
| Hover | May only change color, border, or background (`hover:border-white/20`, `hover:bg-white/10`). |
| Sizes | No pixel sizes. Use Tailwind spacing/sizing scale and grid/flex utilities. |

> **Note about the existing repo:** the starter template's `README.md` recommends the Tailwind browser CDN. That approach is **forbidden** here, and the README must be fully replaced (see section 9).

---

## 3. Project structure

```
.
├── index.html          # the dashboard page; links dist/output.css
├── src/
│   └── input.css       # single line: @import "tailwindcss";
├── dist/
│   └── output.css      # generated, minified, COMMITTED to git
├── package.json        # scripts: "build" (one-shot, --minify) and "dev" (--watch)
├── .gitignore          # node_modules
├── README.md           # GitHub-ready docs (section 9)
├── server.py           # starter Flask dev server (port 3000), used to preview the page
├── CONTEXT.md          # this file
└── validation.md       # delivery verification checklist
```

Starter files from the 4Geeks template (`learn.json`, `README.es.md`, `README.cn.md`, `.vscode/`) may remain. Following the template's instructions, `server.py` (Flask) is kept as the local preview server: it serves `index.html` at `/` and every other file (including `dist/output.css`) by path. It is a dev tool only; the delivered page is static HTML + CSS.

### npm scripts
```json
"scripts": {
  "build": "npx @tailwindcss/cli -i ./src/input.css -o ./dist/output.css --minify",
  "dev":   "npx @tailwindcss/cli -i ./src/input.css -o ./dist/output.css --watch",
  "start": "python3 server.py"
}
```

`index.html` links the stylesheet with `<link rel="stylesheet" href="./dist/output.css">`.

---

## 4. Layout and semantic HTML

### Page skeleton
```
<header>                         ← radial glow background div inside
  <nav> … </nav>                 ← sticky or static; active item uses accent gradient
  <h1> … </h1>                   ← exactly ONE h1 on the page
</header>
<main>
  <section aria-labelledby="kpis-title">     <h2 id="kpis-title">       1. TOP: KPIs
  <section aria-labelledby="drivers-title">  <h2 id="drivers-title">    2. MIDDLE: Drivers
  <section aria-labelledby="ops-title">      <h2 id="ops-title">        3. BOTTOM: Operational details & alerts
</main>
```

- Each `<section>` has an `<h2>` and `aria-labelledby` pointing to its id.
- Cards use `<h3>`. Heading order must be logical (h1 → h2 → h3, no skips).
- Tables: `<caption>`, `<th scope="col">`, each wrapped in `<div class="overflow-x-auto">`.
- The **page** must never scroll horizontally at **375px** (only table wrappers may scroll).
- All interactive elements (nav links, filter tab labels) show a visible `focus-visible:` ring. For `sr-only` radios, the ring must appear on the visible label (e.g. `has-focus-visible:ring-2` or `peer-focus-visible:`).

### Responsive (mobile-first)
Base classes target phones; overrides via at least **three breakpoints**, e.g.:

| Breakpoint | Width | Typical layout |
|---|---|---|
| base | < 640px (test at 375px) | 1 column, stacked cards |
| `sm:` | ≥ 640px | 2 KPI columns, larger display values |
| `md:` | ≥ 768px | 2–3 columns, drivers side by side |
| `lg:` | ≥ 1024px (test at 1280px) | 4 KPI columns, multi-column drivers/ops |

---

## 5. Business model and data rules

### Products and commission (15% per sale)

| Product | Price | Commission per sale (15%) |
|---|---|---|
| Product A | €50 | €7.50 |
| Product B | €120 | €18.00 |
| Product C | €80 | €12.00 |

### Platforms
Instagram, TikTok, YouTube — each with an **assumed ad spend** (labelled as an assumption in the README) so that ROI and cost per conversion can be computed.

### Period
A fixed period, e.g. **"Last 30 days"**, with a **"vs. previous period"** delta on every KPI.

### Formulas (must hold everywhere)

| Metric | Formula |
|---|---|
| Commission revenue (row) | `0.15 × price × units sold` |
| Total commission | Σ product commission = Σ platform revenue = KPI value |
| Units sold (total) | Σ product units = Σ platform sales |
| Average selling price | `Σ(price × units) ÷ Σ units` |
| Engagement rate | `interactions ÷ reach` (interactions = likes + comments + shares + saves) |
| Conversion rate | `conversions ÷ reach` |
| CTR | `clicks ÷ reach` (or ÷ impressions; state which) |
| ROI | `revenue ÷ cost` (cost = assumed ad spend) |
| Cost per conversion | `cost ÷ conversions` |
| Funnel drop-off | `1 − (next stage ÷ current stage)` |
| Revenue share (donut) | `product commission ÷ total commission`; shares sum to 100%; conic-gradient stops match the shares |
| Delta vs prev. | `(current − previous) ÷ previous` |

**Consistency rule:** every number that appears more than once (KPI cards, funnel, donut, tables, ranked lists, alerts, README) must be the same value. At least **two rows must be verified by hand** before delivery.

---

## 6. Dashboard content

### 6.1 TOP block — KPIs (target 8 cards; minimum 3)
At least one KPI per category (proxies labelled in README):

| Category | Candidate KPI |
|---|---|
| Volume | Units sold / total impressions |
| Revenue | Commission revenue (€) / average selling price (€) |
| Engagement | Engagement rate (%) / total interactions |
| Retention | Repeat-buyer rate or audience-retention **proxy** (%) |
| Performance | Conversion rate (conversions ÷ reach) / CTR |
| Satisfaction | Audience sentiment score or CSAT **proxy** |
| Efficiency | ROI (revenue ÷ cost) / cost per conversion (€) |

**Every card has identical structure:**
1. Uppercase label (`text-xs uppercase tracking-wide text-slate-300`)
2. Large value (`text-3xl sm:text-4xl font-bold tabular-nums`, hero values with accent gradient text)
3. Delta with arrow glyph **▲/▼** and text, e.g. `▲ +12% vs prev.` — never color alone
4. Small progress bar built with Tailwind width utilities (e.g. `w-3/4`, `w-[62%]`)

### 6.2 MIDDLE block — Drivers (target 5 groups; minimum 3)
1. **Sales funnel:** Reach → Clicks → Conversions → Sales; bars sized with width utilities; stage-to-stage drop-off %.
2. **Performance by platform:** reach, engagement rate, conversions for Instagram, TikTok, YouTube.
3. **Performance by product:** commission share and conversion rate per product + **donut chart** (`rounded-full` div with `bg-[conic-gradient(...)]`) and a legend beside it.
4. **Quality:** qualified-click ratio and valid-sale rate (proxies labelled).
5. **Activity & engagement by platform:** posts, reels, stories, videos; likes, comments, shares, saves.

**Sparklines:** inline `<svg>` with `<polyline>` and `stroke-current` (allowed — it is HTML, not JS).

### 6.3 BOTTOM block — Operational details
- **Products table:** product, price, commission per sale, units sold, commission €, conversion rate, ROI.
- **Platforms table:** platform, reach, engagement rate, conversions, revenue, ROI; **badge** marking the best platform per metric.
- **Campaigns table:** campaign name, dates, products promoted, results, performance status (badge).
- **Alerts (exactly 3)** in `<ul role="status">`:
  1. Sharp conversion **drop**
  2. Conversion **spike**
  3. Performance **anomaly**
  Severity = color + text label (e.g. `▼ Critical`, `▲ Spike`, `⚠ Warning`).
- **Ranked lists with filter tabs:** "Top products", "Top platforms", "Top campaigns", "Improvement opportunities". Built with visually hidden radio inputs (`sr-only`) and `has-checked:` / `peer-checked:` to reveal the matching list. **No JavaScript.**

---

## 7. Visual design (premium, static, dark)

| Element | Classes / rule |
|---|---|
| Page background | `bg-[#0B0F1A]` |
| Cards | `bg-white/5 border border-white/10 backdrop-blur rounded-2xl shadow-lg shadow-black/30 ring-1 ring-white/10` |
| Header glow | background div with `bg-[radial-gradient(...)]` |
| Accent (one only) | violet → cyan: `bg-gradient-to-r from-violet-500 to-cyan-400`; on hero values add `bg-clip-text text-transparent`. Used for main numbers, active nav item, primary chart. |
| Semantic colors | emerald = positive, rose = negative, amber = warning — always paired with ▲/▼ or a text label |
| Display values | `text-3xl sm:text-4xl font-bold tabular-nums` |
| Labels | `text-xs uppercase tracking-wide text-slate-300` |
| Numbers | `tabular-nums` on **every** money and percentage value |
| Badges | `rounded-full px-2.5 py-0.5 text-xs font-medium` + tint, e.g. `bg-emerald-500/15 text-emerald-300` |
| Hover | only `hover:border-white/20` or `hover:bg-white/10` — no movement |
| Contrast | WCAG AA. Body text `text-slate-300` or lighter; avoid `text-slate-400` for small text on translucent surfaces. |

---

## 8. Build and run workflow

```bash
npm install
npm run build          # generates minified dist/output.css
npm run dev            # watch mode while developing (writes unminified CSS; run build again before committing)
pip3 install flask     # once, as the template README instructs
npm start              # = python3 server.py → http://localhost:3000 (open the forwarded port 3000)
# alternative: npx serve .
```

Check the page at **375px, 768px, 1280px**: no horizontal page scroll at 375px, no overlapping text. Fix every FAIL and re-run the checklist.

---

## 9. README.md requirements (replace the starter README)

Sections in this exact order:
1. Title and one-line description
2. Problem statement (consolidate Instagram, TikTok, YouTube results)
3. The five questions the dashboard answers
4. Dashboard structure: KPIs, drivers, operational details
5. Tech stack: HTML + Tailwind CSS v4 (no JavaScript)
6. Getting started in Codespaces: `npm install`, `npm run build`, `npm run dev`, open the forwarded port
7. Project structure
8. Responsive breakpoints (mobile, tablet, desktop)
9. Visual design: palette, typography, data visuals (conic donut, funnel bars, SVG sparklines)
10. Assumptions: invented sample data; assumed ad spend per platform; each retention/satisfaction/quality proxy named
11. Limitations and next steps: real APIs (Instagram Graph, TikTok, YouTube Analytics) + affiliate/sales source
12. Screenshot placeholder

If `@source "../index.html";` was added to `src/input.css`, note it in the README.

---

## 10. Definition of done

Delivery is complete only when:
- All files in section 3 exist, `dist/output.css` is generated and committed.
- Every item in the final checklist below is **PASS**, with evidence recorded in [validation.md](validation.md).

## 11. Final checklist (report PASS / FAIL for each)

- [ ] Semantic HTML: header/nav/main/section landmarks, one h1, logical h2/h3 order
- [ ] Tailwind only: `src/input.css` has only the @import line; zero `<style>`, zero `style=""`, zero CDN, zero v3 patterns
- [ ] Three clearly separated blocks in order: KPIs, drivers, operational details
- [ ] KPIs cover all 7 categories (volume, revenue, engagement, retention, performance, satisfaction, efficiency)
- [ ] Drivers cover funnel, platform, quality, product, activity, and engagement
- [ ] Operational block has products table, platforms table, campaigns table, alerts, and filtered ranked lists
- [ ] Reusable components: consistent card and table patterns
- [ ] Responsive: works at 375px, 768px, and 1280px, mobile-first, no horizontal scroll at 375px
- [ ] Zero animate-, transition, duration-, motion-, or keyframe usage in `index.html` and `src/input.css`
- [ ] Sample data internally consistent: commission = 15% × price × units; spot-checked by hand
- [ ] Contrast meets WCAG AA for body text and labels
- [ ] README.md complete, including the visual design section and the proxy assumptions
