# Validation: Influencer Analytics Dashboard

How to verify that the project meets every requirement in the brief (summarised in [CONTEXT.md](CONTEXT.md)). Run every check, record **PASS / FAIL** and evidence, fix every FAIL, then re-run the whole list.

**Legend:** 🤖 = automated command (run from the repo root) · 👁 = manual check in the browser or by reading the code.

A command check **passes** when its output matches the "Expected" column exactly.

---

## V1. Project structure and build

| # | Check | How | Expected |
|---|---|---|---|
| 1.1 | Required files exist | 🤖 `ls index.html src/input.css dist/output.css package.json .gitignore README.md` | All 6 listed, no error |
| 1.2 | Tailwind v4 CLI installed | 🤖 `npm ls tailwindcss @tailwindcss/cli` | Both listed, `tailwindcss@4.x` |
| 1.3 | npm scripts | 🤖 `node -p "JSON.stringify(require('./package.json').scripts,null,1)"` | `build` uses `--minify`; `dev` uses `--watch`; both use `-i ./src/input.css -o ./dist/output.css` |
| 1.4 | Build succeeds | 🤖 `npm install && npm run build` | Exits 0, `dist/output.css` updated |
| 1.5 | Output is minified (re-run `npm run build` after `npm run dev`, which writes unminified CSS) | 🤖 `wc -l dist/output.css` | 1–2 lines |
| 1.6 | `index.html` links output | 🤖 `grep -c 'href="\./dist/output.css"\|href="dist/output.css"' index.html` | `1` |
| 1.7 | `.gitignore` ignores node_modules | 🤖 `grep -x 'node_modules/\?' .gitignore` | Match found |
| 1.8 | `dist/output.css` committed | 🤖 `git ls-files dist/output.css` | Path printed |
| 1.9 | `node_modules` not committed | 🤖 `git ls-files node_modules \| head -1` | Empty |
| 1.10 | Classes are actually generated | 🤖 `for c in 'bg-\[#0B0F1A\]' 'conic-gradient' 'radial-gradient' 'has-checked' 'tabular-nums' 'backdrop-blur' 'from-violet-500' 'to-cyan-400' 'overflow-x-auto' 'sr-only'; do grep -q "$c" dist/output.css && echo "OK $c" \|\| echo "MISSING $c"; done` | All `OK` (if any `MISSING`, add `@source "../index.html";` and note it in README) |

---

## V2. Tailwind-only / forbidden patterns

| # | Check | How | Expected |
|---|---|---|---|
| 2.1 | `input.css` is one line | 🤖 `cat src/input.css` | `@import "tailwindcss";` only (plus `@source "../index.html";` **only** if justified by 1.10 and documented) |
| 2.2 | No `@theme`/custom CSS | 🤖 `grep -cE '@theme\|@layer\|@apply\|@tailwind\|\{' src/input.css` | `0` |
| 2.3 | No `<script>` | 🤖 `grep -ci '<script' index.html` | `0` |
| 2.4 | No `<style>` | 🤖 `grep -ci '<style' index.html` | `0` |
| 2.5 | No inline styles | 🤖 `grep -ciE '\sstyle=' index.html` | `0` |
| 2.6 | No CDN / external CSS | 🤖 `grep -ciE 'cdn\.\|jsdelivr\|unpkg\|tailwindcss\.com\|@tailwindcss/browser\|googleapis' index.html` | `0` |
| 2.7 | No v3 config | 🤖 `ls tailwind.config.* postcss.config.* 2>/dev/null` | Empty |
| 2.8 | No v3 directives | 🤖 `grep -rn '@tailwind' src/ index.html` | Empty |
| 2.9 | No other CSS files | 🤖 `git ls-files '*.css'` | Only `src/input.css` and `dist/output.css` |
| 2.10 | No JS event handlers | 🤖 `grep -ciE '\son[a-z]+=' index.html` | `0` |
| 2.11 | No JS files | 🤖 `git ls-files '*.js' '*.mjs' '*.jsx' '*.ts'` | Empty |

---

## V3. No animation or motion

Applies to `index.html` and `src/input.css` (`dist/output.css` is excluded: Tailwind's theme layer always emits transition variables).

| # | Check | How | Expected |
|---|---|---|---|
| 3.1 | Forbidden utilities | 🤖 `grep -noE 'animate-\|transition\|duration-\|motion-\|keyframes\|ease-\|delay-' index.html src/input.css` | Empty |
| 3.2 | No hover movement | 🤖 `grep -noE '(hover\|focus\|group-hover):(-?translate\|scale\|rotate\|skew)' index.html` | Empty |
| 3.3 | Hover only changes color/border/bg | 🤖 `grep -oE 'hover:[a-z0-9/\[\]#.-]+' index.html \| sort -u` | Only `hover:bg-*`, `hover:border-*`, `hover:text-*` |

---

## V4. Semantic HTML and accessibility structure

| # | Check | How | Expected |
|---|---|---|---|
| 4.1 | Landmarks | 🤖 `for t in header nav main; do echo "$t: $(grep -c "<$t[ >]" index.html)"; done` | `header ≥1`, `nav ≥1`, `main: 1` |
| 4.2 | Exactly one `<h1>` | 🤖 `grep -o '<h1[ >]' index.html \| wc -l` | `1` |
| 4.3 | Three main sections | 🤖 `grep -o '<section[ >]' index.html \| wc -l` | `3` (more only if nested inside, never as extra top-level blocks) |
| 4.4 | Sections labelled | 🤖 `grep -oE '<section[^>]*aria-labelledby="[^"]+"' index.html \| wc -l` | Equals section count |
| 4.5 | `aria-labelledby` targets an `<h2>` | 🤖 `for id in $(grep -oE 'aria-labelledby="[^"]+"' index.html \| cut -d'"' -f2); do grep -qE "<h2[^>]*id=\"$id\"" index.html && echo "OK $id" \|\| echo "FAIL $id"; done` | All `OK` |
| 4.6 | Heading order | 🤖 `grep -oE '<h[1-6]' index.html \| tr -d '<' \| uniq -c` | Starts with `h1`; no jump from h1→h3 or h2→h4 |
| 4.7 | Cards use `<h3>` | 👁 every card title is an `<h3>` | Yes |
| 4.8 | Tables have captions | 🤖 `echo "tables=$(grep -c '<table' index.html) captions=$(grep -c '<caption' index.html)"` | Equal, ≥ 3 |
| 4.9 | Column headers scoped | 🤖 `grep -oE '<th[ >][^>]*' index.html \| grep -vc 'scope='` | `0` |
| 4.10 | Tables wrapped in scroll div | 👁 each `<table>` is the child of a `<div class="… overflow-x-auto …">` | Yes for all 3 |
| 4.11 | Alerts list | 🤖 `grep -c '<ul[^>]*role="status"' index.html` | `1`, containing exactly 3 `<li>` |
| 4.12 | Focus rings on interactive elements | 🤖 `grep -oE '<(a\|button\|label)[^>]*>' index.html \| grep -vc 'focus-visible:'` | `0` (radio tab labels may use `has-focus-visible:` / `peer-focus-visible:`) |
| 4.13 | Keyboard test | 👁 Tab through the page | Every nav link and filter tab shows a visible ring; arrow keys switch filter tabs |
| 4.14 | No pixel sizes | 🤖 `grep -noE '[0-9.]+px' index.html` | Empty (ignore SVG `viewBox` numbers, which have no unit) |
| 4.15 | `lang` and viewport | 🤖 `grep -cE '<html[^>]*lang=\|name="viewport"' index.html` | `2` |

---

## V5. Required content

### KPIs (TOP block)
| # | Check | Expected |
|---|---|---|
| 5.1 | 👁 8 KPI cards (minimum 3) | 8 |
| 5.2 | 👁 Category coverage | ✅ Volume ✅ Revenue ✅ Engagement ✅ Retention (proxy) ✅ Performance ✅ Satisfaction (proxy) ✅ Efficiency |
| 5.3 | 👁 Identical card structure | uppercase label → large value → ▲/▼ delta with text ("+12% vs prev.") → width-based progress bar |
| 5.4 | 🤖 Arrows present: `grep -oE '▲\|▼' index.html \| wc -l` | ≥ 8 |
| 5.5 | 👁 Fixed period shown | e.g. "Last 30 days" visible; every KPI has "vs prev." |

### Drivers (MIDDLE block)
| # | Check | Expected |
|---|---|---|
| 5.6 | 👁 Funnel | Reach → Clicks → Conversions → Sales, bars sized by width utilities, drop-off % between stages |
| 5.7 | 👁 Platform performance | Instagram, TikTok, YouTube × reach, engagement rate, conversions |
| 5.8 | 👁 Product performance + donut | Commission share + conversion rate per product; `rounded-full` div with `bg-[conic-gradient(...)]`; legend beside it |
| 5.9 | 👁 Quality | Qualified-click ratio + valid-sale rate (labelled proxy if applicable) |
| 5.10 | 👁 Activity & engagement | Posts, reels, stories, videos; likes, comments, shares, saves per platform |
| 5.11 | 🤖 Sparklines: `grep -c '<polyline' index.html`; `grep -c 'stroke-current' index.html` | Both ≥ 1 |

### Operational details (BOTTOM block)
| # | Check | Expected |
|---|---|---|
| 5.12 | 👁 Products table columns | product, price, commission per sale, units sold, commission €, conversion rate, ROI |
| 5.13 | 👁 Platforms table columns | platform, reach, engagement rate, conversions, revenue, ROI + "Best" badge per metric |
| 5.14 | 👁 Campaigns table columns | campaign name, dates, products promoted, results, performance status badge |
| 5.15 | 👁 Alerts | exactly 3: conversion drop, conversion spike, performance anomaly; severity = color **+** text (e.g. "▼ Critical") |
| 5.16 | 🤖 Filter radios hidden: `grep -oE '<input[^>]*type="radio"[^>]*>' index.html \| grep -c 'sr-only'` | `4` |
| 5.17 | 🤖 CSS-only filter: `grep -cE 'has-checked:\|peer-checked:\|has-\[' index.html` | ≥ 1 |
| 5.18 | 👁 Filter works | Clicking each tab ("Top products", "Top platforms", "Top campaigns", "Improvement opportunities") shows only its list; one tab checked by default |

---

## V6. Data consistency

Constants: commission rate **15%**; A €50 → €7.50, B €120 → €18.00, C €80 → €12.00.

| # | Check | Expected |
|---|---|---|
| 6.1 | Every product row | `commission € = 0.15 × price × units` |
| 6.2 | Commission per sale column | €7.50 / €18.00 / €12.00 |
| 6.3 | Totals | Σ product commission = Σ platform revenue = Commission KPI |
| 6.4 | Units | Σ product units = Units KPI = final funnel stage (Sales) |
| 6.5 | Funnel | Stages non-increasing; drop-off % = `1 − next ÷ current` |
| 6.6 | Platforms | Σ platform conversions = funnel Conversions; ROI = revenue ÷ assumed spend; cost/conversion = spend ÷ conversions |
| 6.7 | Rates | Conversion rate = conversions ÷ reach; engagement rate = interactions ÷ reach |
| 6.8 | Donut | Shares sum to 100% and match conic-gradient stops and legend |
| 6.9 | ASP | `Σ(price × units) ÷ Σ units` |
| 6.10 | Cross-references | Ranked lists, alerts, campaigns and README quote the same numbers as the tables |

### Hand spot-check (fill in — at least two rows)

| Row | Price | Units | 15% × price × units | Value shown on page | Result |
|---|---|---|---|---|---|
| Product A | €50 | … | 0.15 × 50 × … = € … | € … | PASS / FAIL |
| Product B | €120 | … | 0.15 × 120 × … = € … | € … | PASS / FAIL |
| Product C | €80 | … | 0.15 × 80 × … = € … | € … | PASS / FAIL |
| **Total** | | Σ … | Σ = € … | KPI € … | PASS / FAIL |

---

## V7. Visual design

| # | Check | How | Expected |
|---|---|---|---|
| 7.1 | Page background | 🤖 `grep -c 'bg-\[#0B0F1A\]' index.html` | ≥ 1 (on `<body>`) |
| 7.2 | Card recipe | 🤖 `grep -c 'bg-white/5 border border-white/10 backdrop-blur rounded-2xl shadow-lg shadow-black/30 ring-1 ring-white/10' index.html` | ≥ number of cards (same pattern everywhere) |
| 7.3 | Header glow | 🤖 `grep -c 'bg-\[radial-gradient' index.html` | ≥ 1, inside `<header>` |
| 7.4 | Accent gradient | 🤖 `grep -c 'from-violet-500 to-cyan-400' index.html` | ≥ 3 (hero values, active nav, primary chart) |
| 7.5 | Gradient text | 🤖 `grep -c 'bg-clip-text text-transparent' index.html` | ≥ 1 |
| 7.6 | Display typography | 🤖 `grep -c 'text-3xl sm:text-4xl font-bold tabular-nums' index.html` | ≥ number of KPI cards |
| 7.7 | Label typography | 🤖 `grep -c 'text-xs uppercase tracking-wide text-slate-300' index.html` | ≥ number of KPI cards |
| 7.8 | `tabular-nums` on numbers | 👁 every €/% value sits in an element (or table) with `tabular-nums` | Yes |
| 7.9 | Badges | 🤖 `grep -c 'rounded-full px-2.5 py-0.5 text-xs font-medium' index.html` | ≥ 1 per badge |
| 7.10 | Semantic colors paired | 👁 every emerald/rose/amber element also shows ▲, ▼, ⚠ or a text label | Yes |

---

## V8. Contrast (WCAG AA)

| # | Check | How | Expected |
|---|---|---|---|
| 8.1 | No weak small text on translucent cards | 🤖 `grep -noE 'text-(slate\|gray\|zinc)-(400\|500\|600\|700)' index.html` | Empty, or each hit justified (large text or decorative only) |
| 8.2 | Body/labels | 👁 body text `text-slate-300` or lighter; labels `text-slate-300` | Yes |
| 8.3 | Measured ratios | 👁 DevTools → Inspect → contrast (or Lighthouse Accessibility) | ≥ 4.5:1 normal text, ≥ 3:1 large text |
| 8.4 | Lighthouse | 👁 Lighthouse Accessibility audit | No contrast failures; score ≥ 95 |

---

## V9. Responsive and visual QA

Serve as the template README instructs: `pip3 install flask && python3 server.py` (or `npm start`) → open forwarded port **3000** → DevTools device toolbar. `npx serve .` also works.

| # | Check | Expected |
|---|---|---|
| 9.1 | 🤖 Breakpoints used: `for b in sm md lg xl; do echo "$b: $(grep -o "$b:" index.html \| wc -l)"; done` | At least 3 breakpoints with > 0 |
| 9.2 | 👁 Mobile-first | Base classes = phone layout; `md:`/`lg:` only add columns/size |
| 9.3 | 👁 375px | No horizontal page scroll (`document.documentElement.scrollWidth === 375` in console), no overlapping/clipped text, tables scroll inside their wrapper |
| 9.4 | 👁 768px | Tablet layout, no overlap |
| 9.5 | 👁 1280px | Desktop multi-column layout, no overlap |
| 9.6 | 👁 Blocks order | KPIs → Drivers → Operational details, clearly separated |
| 9.7 | 👁 No console errors / 404s | Network tab: `output.css` loads with 200 |
| 9.8 | 🤖 Server smoke test (server running) | `curl -s -o /dev/null -w '%{http_code}' localhost:3000/dist/output.css` → `200` |

---

## V10. README.md

| # | Section (in this order) | Present |
|---|---|---|
| 10.1 | Title + one-line description | ☐ |
| 10.2 | Problem statement (Instagram, TikTok, YouTube consolidation) | ☐ |
| 10.3 | The five questions | ☐ |
| 10.4 | Dashboard structure (KPIs, drivers, operational details) | ☐ |
| 10.5 | Tech stack: HTML + Tailwind CSS v4, no JavaScript | ☐ |
| 10.6 | Getting started in Codespaces (`npm install`, `npm run build`, `npm run dev`, forwarded port) | ☐ |
| 10.7 | Project structure | ☐ |
| 10.8 | Responsive breakpoints (mobile, tablet, desktop) | ☐ |
| 10.9 | Visual design: palette, typography, conic donut, funnel bars, SVG sparklines | ☐ |
| 10.10 | Assumptions: invented data, assumed ad spend per platform, **each proxy named** | ☐ |
| 10.11 | Limitations & next steps: Instagram Graph, TikTok, YouTube Analytics APIs + affiliate/sales source | ☐ |
| 10.12 | Screenshot placeholder | ☐ |
| 10.13 | `@source` note (only if used) | ☐ / N/A |
| 10.14 | Starter template text (CDN instructions) removed | ☐ |

🤖 Order check: `grep -n '^## ' README.md`

---

## V11. Quick automated run

Copy-paste from the repo root; every line should print `PASS`.

```bash
chk(){ if eval "$2"; then echo "PASS  $1"; else echo "FAIL  $1"; fi; }
chk "input.css single import"   '[ "$(grep -v "^\s*$" src/input.css)" = "@import \"tailwindcss\";" ] || grep -qx "@source \"../index.html\";" src/input.css'
chk "no <script>"               '! grep -qi "<script" index.html'
chk "no <style>"                '! grep -qi "<style" index.html'
chk "no style=\"\""             '! grep -qiE "\sstyle=" index.html'
chk "no CDN"                    '! grep -qiE "cdn\.|jsdelivr|unpkg|tailwindcss\.com" index.html'
chk "no v3 config"              '! ls tailwind.config.* >/dev/null 2>&1'
chk "no @tailwind"              '! grep -rq "@tailwind" src index.html'
chk "no motion"                 '! grep -qE "animate-|transition|duration-|motion-|keyframes" index.html src/input.css'
chk "no hover movement"         '! grep -qE "hover:(-?translate|scale|rotate)" index.html'
chk "no px sizes"               '! grep -qE "[0-9]px" index.html'
chk "one h1"                    '[ "$(grep -o "<h1[ >]" index.html | wc -l)" -eq 1 ]'
chk "header/nav/main"           'grep -q "<header" index.html && grep -q "<nav" index.html && grep -q "<main" index.html'
chk "3 sections labelled"       '[ "$(grep -oE "<section[^>]*aria-labelledby" index.html | wc -l)" -ge 3 ]'
chk "captions = tables"         '[ "$(grep -c "<caption" index.html)" -eq "$(grep -c "<table" index.html)" ]'
chk "th scoped"                 '[ "$(grep -oE "<th[ >][^>]*" index.html | grep -vc scope=)" -eq 0 ]'
chk "role=status alerts"        'grep -qE "<ul[^>]*role=\"status\"" index.html'
chk "conic donut"               'grep -q "bg-\[conic-gradient" index.html'
chk "radial glow"               'grep -q "bg-\[radial-gradient" index.html'
chk "sparklines"                'grep -q "<polyline" index.html && grep -q "stroke-current" index.html'
chk "css-only filter"           'grep -qE "has-checked:|peer-checked:" index.html'
chk "output.css linked"         'grep -q "dist/output.css" index.html'
chk "output.css committed"      '[ -n "$(git ls-files dist/output.css)" ]'
```

---

## V12. Final checklist (delivery report)

Fill in after all sections above pass. **Any FAIL blocks delivery.**

| # | Requirement | Evidence (sections) | Result |
|---|---|---|---|
| 1 | Semantic HTML: header/nav/main/section landmarks, one h1, logical h2/h3 order | V4 | PASS / FAIL |
| 2 | Tailwind only: `src/input.css` has only the @import line; zero `<style>`, zero `style=""`, zero CDN, zero v3 patterns | V1, V2 | PASS / FAIL |
| 3 | Three clearly separated blocks in order: KPIs, drivers, operational details | 4.3, 9.6 | PASS / FAIL |
| 4 | KPIs cover all 7 categories | 5.1–5.5 | PASS / FAIL |
| 5 | Drivers cover funnel, platform, quality, product, activity, and engagement | 5.6–5.11 | PASS / FAIL |
| 6 | Operational block has products, platforms, campaigns tables, alerts, filtered ranked lists | 5.12–5.18 | PASS / FAIL |
| 7 | Reusable components: consistent card and table patterns | 7.2, 4.8–4.10 | PASS / FAIL |
| 8 | Responsive at 375 / 768 / 1280px, mobile-first, no horizontal scroll at 375px | V9 | PASS / FAIL |
| 9 | Zero animate-, transition, duration-, motion-, keyframe usage in `index.html` and `src/input.css` | V3 | PASS / FAIL |
| 10 | Data consistent: commission = 15% × price × units; spot-checked by hand | V6 | PASS / FAIL |
| 11 | Contrast meets WCAG AA for body text and labels | V8 | PASS / FAIL |
| 12 | README.md complete incl. visual design section and proxy assumptions | V10 | PASS / FAIL |

**Validated by:** ____________ **Date:** ____________ **Commit:** ____________
