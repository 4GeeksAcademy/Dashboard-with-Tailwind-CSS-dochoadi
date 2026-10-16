# Influencer Analytics Dashboard

A static, single-page analytics dashboard that consolidates an influencer's affiliate results from Instagram, TikTok and YouTube, built with HTML and Tailwind CSS v4 and no JavaScript.

![HTML](https://img.shields.io/badge/HTML-semantic-e34f26) ![Tailwind CSS](https://img.shields.io/badge/Tailwind_CSS-v4-38bdf8) ![JavaScript](https://img.shields.io/badge/JavaScript-none-lightgrey)

---

## Problem statement

An influencer promotes three affiliate products and earns a 15% commission on every sale. The results are scattered across Instagram, TikTok and YouTube analytics plus an affiliate sales report, so there is no single place to see what earns money, what converts, and where ad spend pays off.

This dashboard puts those results on one page: the headline KPIs at the top, the drivers behind them in the middle, and the row-level detail and alerts at the bottom.

## Questions this dashboard answers

| # | Question | Where to look |
|---|---|---|
| 1 | How much commission revenue am I generating? | KPI **Commission revenue** (€9,600), Products table total |
| 2 | Which products generate the most revenue? | **Performance by product** donut, Products table, "Top products" ranking |
| 3 | How well do my ads convert (conversions ÷ reach)? | KPI **Conversion rate** (0.079%), Sales funnel, Products and Platforms tables |
| 4 | Which platforms give the best return (revenue ÷ cost)? | KPI **ROI** (2.29×), Platforms table "Best" badges, "Top platforms" ranking |
| 5 | What is the engagement rate by platform and by product? | Performance by platform cards, Platforms table, product conv. / eng. row |

## Dashboard structure

The page has a `<header>` with navigation, then three `<section>` blocks inside `<main>`:

1. **KPIs (top):** eight cards with the same layout: an uppercase label, a large value, a ▲/▼ change vs. the previous period, and a progress bar toward the target.

   | Category | KPI |
   |---|---|
   | Revenue | Commission revenue, average selling price |
   | Volume | Units sold |
   | Engagement | Engagement rate |
   | Performance | Conversion rate (conversions ÷ reach) |
   | Retention | Repeat-buyer rate *(proxy)* |
   | Satisfaction | Audience sentiment *(proxy)* |
   | Efficiency | ROI (revenue ÷ cost), with cost per conversion |

2. **Drivers (middle):**
   - **Sales funnel:** Reach → Clicks → Conversions → Sales, with stage-to-stage drop-off.
   - **Performance by product:** revenue-share donut and legend, plus conversion and engagement rates per product.
   - **Performance by platform:** reach, engagement rate, conversions and cost per conversion, with a sparkline per platform.
   - **Traffic and sales quality:** qualified-click ratio and valid-sale rate *(proxies)*.
   - **Activity and engagement by platform:** posts, reels, stories and videos; likes, comments, shares and saves.

3. **Operational details (bottom):**
   - **Products table** and **Platforms table**, with "Best" badges in the Platforms table.
   - **Campaigns table** with a status badge per campaign.
   - **Three alerts** (critical drop, spike, anomaly) in a `<ul role="status">`.
   - **Ranked lists** with four filter tabs (Top products, Top platforms, Top campaigns, Improvement opportunities), built with CSS only.

## Tech stack

- **HTML5**, with semantic landmarks, captioned tables and labels for assistive technology
- **Tailwind CSS v4**, compiled with the official CLI (`tailwindcss` + `@tailwindcss/cli`)
- **No JavaScript.** There are no `<script>` tags, inline styles, `<style>` blocks or CDNs, and no `tailwind.config.js`. The ranking tabs are hidden radio inputs styled with the `has-checked:` / `group-has-[…]:` variants.
- **Flask `server.py`** (from the original template), used only to preview the page locally

## Getting started in Codespaces

1. Open the repository in a GitHub Codespace (or any machine with Node 18+ and Python 3).
2. Install the dependencies and build the CSS:

   ```bash
   npm install          # installs tailwindcss and @tailwindcss/cli
   npm run build        # writes the minified dist/output.css
   ```

3. Start the preview server. This is the template's Flask server, which serves `index.html` on port 3000:

   ```bash
   pip3 install flask   # first time only
   npm start            # same as: python3 server.py
   ```

4. Open the **Ports** tab and click the forwarded port **3000**.
5. While editing, run the watcher in a second terminal:

   ```bash
   npm run dev          # rebuilds dist/output.css on every save (unminified)
   ```

   Run `npm run build` again before committing, so the committed CSS is minified.

| Script | Command |
|---|---|
| `npm run build` | `npx @tailwindcss/cli -i ./src/input.css -o ./dist/output.css --minify` |
| `npm run dev` | `npx @tailwindcss/cli -i ./src/input.css -o ./dist/output.css --watch` |
| `npm start` | `python3 server.py` |

`npx serve .` also works as a static server.

## Project structure

```
.
├── index.html         # the dashboard (links ./dist/output.css)
├── src/
│   └── input.css      # one line: @import "tailwindcss";
├── dist/
│   └── output.css     # generated and minified; committed so the page works without a build
├── package.json       # build / dev / start scripts
├── server.py          # Flask preview server (port 3000) from the template
├── CONTEXT.md         # project requirements and data rules
├── validation.md      # delivery checklist with verification commands
├── .gitignore         # node_modules/
└── README.md
```

Tailwind v4 finds `index.html` through automatic source detection, so `src/input.css` needs no `@source` line.

## Responsive breakpoints

The layout is mobile-first: base classes target phones, and breakpoints only add columns and size.

| Range | Breakpoint | Layout |
|---|---|---|
| Mobile (tested at 375px) | base | Single column. Wide tables scroll inside their own `overflow-x-auto` container; the page never scrolls sideways. |
| Large phone / small tablet | `sm:` ≥ 640px | 2-column KPI grid, larger values, donut beside its legend |
| Tablet (tested at 768px) | `md:` ≥ 768px | Platform cards in 3 columns, more spacing |
| Desktop (tested at 1280px) | `lg:` ≥ 1024px | 4-column KPI grid, 2-column drivers, full-width tables, alerts beside rankings |

## Visual design

**Palette** (Tailwind's default colors plus arbitrary values; no custom theme)

| Role | Value |
|---|---|
| Page background | `bg-[#0B0F1A]` |
| Cards | `bg-white/5 border border-white/10 backdrop-blur rounded-2xl shadow-lg shadow-black/30 ring-1 ring-white/10` |
| Header glow | `bg-[radial-gradient(...)]` layer in violet and cyan |
| Accent (one gradient) | `bg-gradient-to-r from-violet-500 to-cyan-400`, used for KPI values (with `bg-clip-text text-transparent`), the active nav item, funnel bars and progress bars |
| Positive / negative / warning | emerald / rose / amber, always shown with ▲, ▼, ⚠ or a text label, never color alone |
| Badges | `rounded-full px-2.5 py-0.5 text-xs font-medium` with a tint, e.g. `bg-emerald-500/15 text-emerald-300` |

**Typography**
- KPI values: `text-3xl sm:text-4xl font-bold tabular-nums`
- Labels: `text-xs uppercase tracking-wide text-slate-300`
- Body text is `text-slate-200`/`text-slate-300` or lighter, which keeps contrast at WCAG AA on the dark surfaces.
- Every money and percentage value uses `tabular-nums`, so digits line up.

**Data visuals (pure HTML and CSS)**
- **Donut chart:** a `rounded-full` div with `bg-[conic-gradient(var(--color-violet-500)_0_32.8%, …)]` and an inner circle for the total.
- **Funnel bars:** fixed width utilities (`w-full`, `w-[73%]`, `w-[49%]`, `w-[48%]`) on a log scale, so the small stages stay visible.
- **Sparklines:** inline `<svg>` with a `<polyline>` drawn with `stroke-current`, so each line takes its color from a text color class.
- **Progress bars:** a `bg-white/10` track with a width-sized gradient fill.

There is no motion: no animation or transition utilities. Hover only changes a border or background (`hover:border-white/20`, `hover:bg-white/10`). Every link, filter tab and scrollable table shows a `focus-visible:` ring.

## Assumptions

- **All data is invented sample data** for the period 16 Sep – 15 Oct 2026 ("Last 30 days"), compared with the previous 30 days.
- **Commission:** 15% of the price on every sale: Product A €50 → €7.50, Product B €120 → €18.00, Product C €80 → €12.00. Every row and total follows commission = 0.15 × price × units.

  | Product | Units | 0.15 × price × units |
  |---|---|---|
  | A | 420 | 0.15 × 50 × 420 = **€3,150** |
  | B | 185 | 0.15 × 120 × 185 = **€3,330** |
  | C | 260 | 0.15 × 80 × 260 = **€3,120** |
  | **Total** | **865** | **€9,600** (= Instagram €3,630 + TikTok €2,925 + YouTube €3,045) |

- **Ad spend per platform is assumed:** Instagram €1,650, TikTok €1,500, YouTube €1,050 (total €4,200). ROI = revenue ÷ spend, and cost per conversion = spend ÷ conversions.
- **Ad spend per product is assumed** for product ROI: A €1,600, B €1,300, C €1,300 (the same €4,200 split by share of sponsored posts).
- **Conversions vs. sales:** a conversion is an attributed order (912). A sale is a valid unit after cancellations and returns (865), with one unit per order.
- **Proxies:**
  - *Retention:* **repeat-buyer rate** = share of orders using a discount code the customer has used before.
  - *Satisfaction:* **audience sentiment** = share of positive comments on promotional content, scored 0–100 (a stand-in for customer satisfaction, CSAT).
  - *Quality:* **qualified-click ratio** = clicks that stayed 10 seconds or more on the product page.
  - *Quality:* **valid-sale rate** = orders that were not cancelled or returned.
- **Product reach** is the reach of the posts that featured each product. Each post features exactly one product.
- **Targets** behind the progress bars (e.g. €12,000 commission) are illustrative.

## Limitations and next steps

- **Static snapshot:** the numbers are written into the HTML, with no live data, date picker or drill-down.
- **Connect real sources:**
  - Instagram Graph API for reach, interactions and story/reel metrics
  - TikTok for Business / TikTok API for video reach and engagement
  - YouTube Analytics API for views, Shorts and video engagement
  - An affiliate network or store (e.g. a Shopify or Amazon Associates report) for orders, returns and commission
- **Generate the HTML** from those sources at build time (e.g. a small static-site script) so the page itself still ships without JavaScript.
- Replace the proxies with direct measures once the data exists, e.g. a post-purchase survey for satisfaction and a customer ID for repeat buyers.
- Add a print stylesheet and a light theme.

## Screenshot

> _Screenshot placeholder: add a capture of the dashboard at 1280px here, e.g. `docs/screenshot.png`._

<!-- ![Influencer Analytics Dashboard](docs/screenshot.png) -->
