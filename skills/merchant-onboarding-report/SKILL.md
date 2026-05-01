---
name: merchant-onboarding-report
description: "Generates a branded HTML best practice report for a Shopify Plus merchant — explains their CS team, covers their full platform stack with per-product health scores, delivers a Shopify Payments deep dive (Shop Pay, SPI, multi-currency, LPMs, wallets), gap cards with business benefits and case studies, and a Resources section. Output is a self-contained HTML quick-site styled with the merchant's design.md."
---

# Merchant Onboarding Report

Produces a branded, merchant-facing HTML report. Tone: calm, considered, practical — no jargon, no urgency. Structured like a trusted advisor review, not a sales deck.

## Arguments

The user must pass a **merchant domain or name** (e.g., `/merchant-onboarding-report pandalondon.com`).
If no argument is provided, ask for the merchant domain before proceeding.
Store as `{merchant_domain}` and `{merchant_name}`.

---

## Steps

Run steps 1–5 in parallel, then compile into the HTML report.

### 1. Fetch merchant design.md

Fetch: `https://designmd.quick.shopify.io/{merchant_domain}`

Extract:
- `{primary}`, `{secondary}`, `{neutral}`, `{surface}`, `{on_surface}`, `{muted}` — hex colours
- `{headline_font}`, `{body_font}` — font families
- `{logo_url}` — brand logo URL
- `{deck_hint}` — styling guidance
- `{voice_tone}` — brand voice description
- `{aesthetic}` — one-line brand aesthetic

If fetch fails, use Shopify defaults: primary `#008060`, neutral `#F6F6F1`, on-surface `#202223`.

### 2. Pull merchant data

Call `search_data_tool` with dataset `Accounts`:
Filter: `contains(tolower(domain), '{merchant_domain}')`
Select: `account_name, msm_name, gmv_usd_l365d, revenue_l12m, gmv_growth_yearly, adopted_products, eligible_products, b2b_fit_score, d2c_fit_score, retail_fit_score, is_b2b, is_d2c, is_retail, country, industry, employee_count, account_id, banff_risk_level, store_count`

Call `search_salesforce_tool`:
```sql
SELECT Id, Name, BillingCountry, Industry, AnnualRevenue, NumberOfEmployees
FROM Account
WHERE RecordType.Name = 'Merchant'
AND Name LIKE '%{merchant_name}%'
LIMIT 1
```

Store: `{gmv}`, `{plan}`, `{msm_name}`, `{country}`, `{industry}`, `{store_count}`, `{adopted_products}`, `{eligible_products}`.

### 3. Map product activation + peer benchmarks

Using `adopted_products` and `eligible_products`, build two lists:
- `{active_products}` — confirmed adopted
- `{gap_products}` — eligible but not yet adopted

**Per-product health score** — score each product 0–10:
- Active + high peer adoption (>80%): 10/10 green
- Active + ahead of peers (low benchmark): 7/10 green
- Not active + very high peer adoption (>90%): 2/10 red (critical)
- Not active + high peer adoption (70–90%): 3/10 red
- Not active + medium peer adoption (40–70%): 4/10 amber
- Unconfirmed / verify: 5/10 amber

**Overall platform health score**: average of all product scores, displayed as X.X/10.

**Shopify Payments deep-dive checklist** (check each independently):

| Check | How to assess |
|---|---|
| Shopify Payments active | In `adopted_products` |
| Shop Pay enabled | In `adopted_products` |
| Shop Pay Instalments (SPI) | In `adopted_products` — separate from Shop Pay |
| Apple Pay / Google Pay | Infer from Payments being active — flag for verification |
| Multi-currency via Markets | Check `adopted_products` for 'Markets' |
| Local payment methods configured | Flag if Markets is active but LPMs not confirmed |
| Fraud protection configured | Flag for CSM to verify in admin |
| Payout schedule optimised | Flag for CSM to verify |

**Local payment methods by market (flag which apply for this merchant's country/markets):**
- UK: Shop Pay, Apple Pay, Google Pay, Klarna, Clearpay
- DE/AT: SOFORT, Giropay, Klarna
- FR: Carte Bancaire, PayPal
- NL/BE: iDEAL, Bancontact
- US: Shop Pay Instalments, Affirm, PayPal, Venmo

**Peer benchmark percentages** (use these fixed benchmarks for Plus EMEA cohort):
- Shop Pay: 91%
- Shopify Flow: 97%
- Launchpad: 73%
- Plus Store Operations: 98%
- Search & Discovery: 87%
- Sidekick: 78%
- Shopify Email: 70%
- B2B: 34%
- Markets: 58%
- Shopify Payments: 89%
- Shop Pay Instalments: 41%
- POS Pro: 19%
- Shopify Magic: 62%
- Checkout Extensibility: 44%

### 4. Pull case studies

For each product in `{gap_products}`, call `search_sales_call_transcripts`:
- Queries tailored to the product (e.g. for SPI: `['Shop Pay Instalments conversion uplift home brand', 'BNPL merchant success DTC']`)
- `search_mode: summaries`, `opportunity_result: closed_won`, `limit: 2`

Extract per case study: merchant type, product adopted, key outcome, one-line result.
Anonymise merchant names. If no case study found, mark "Case study coming soon."

### 5. Build the HTML report

Write a single self-contained HTML file at `~/{merchant_domain}-platform-review.html`.

Apply the merchant's design tokens throughout. Where custom fonts (e.g. Futura) aren't available via Google Fonts, fall back to `'Nunito', 'Century Gothic', Arial, sans-serif` for headlines and `'Nunito', Arial, sans-serif` for body.

---

## Report structure

### Section 0 — Header
Co-brand lockup: merchant logo left, Shopify logo right, thin vertical divider between them.
Stamp: "Onboarding Best Practice Review · {month} {year}" in small muted text.

### Section 1 — Hero
Dark background (`{on_surface}`). Primary accent bar at bottom (`{primary}`, 6px).

Headline (large, headline font): `h1` = "Onboarding Best Practice Review"

Sub-headline: "Optimise your Shopify Journey"

KPI row (5 stats, `repeat(5, 1fr)` grid):
- GMV L365D
- YoY GMV growth %
- Products active (X / total eligible)
- Industry (`{industry}`)
- Stores (`{store_count}`)

### Section 2 — Your Success Team

Three cards side by side (`grid-template-columns: 1fr 1fr 1fr`), white background, `{primary}` left border accent.

**Card 1 — Your Customer Success Manager**
- Heading: "Your Customer Success Manager"
- Name: `{msm_name}`
- Body: "Your CSM is your dedicated Shopify strategist. They help you get more from the platform — finding the features that fit your growth stage, connecting you with the right resources, and meeting regularly to review what's working and what's next. Think of them as a business partner who knows Shopify inside out."
- Bullet list:
  - Proactive platform and business reviews
  - Connects you to Shopify experts, partners, and programmes
  - Helps prioritise which features to adopt next and in what order
  - Your first call to guide you on your Shopify journey

**Card 2 — Shopify Plus Support**
- Heading: "Your Plus Support"
- Body: "As a Shopify Plus merchant, you have access to priority support — 24/7, with faster response times and agents who know the Plus platform deeply."
- Bullet list:
  - 24/7 priority support via chat, email, and phone
  - Shopify Plus Academy — exclusive training and certification
  - Proactive outreach around platform updates that affect your store

**Card 3 — Growth Services**
- Heading: "Growth Services"
- Sub-heading: "Shopify Growth Services"
- Body: "Shopify's specialist team for merchants ready to go further, faster. Growth Services works directly with Plus merchants on complex builds, integrations, and performance improvements that go beyond the standard platform."
- Bullet list:
  - Custom checkout and theme development
  - Third-party system integrations and migrations
  - Performance audits and conversion optimisation
  - Technical consultation for complex merchant requirements

### Section 3 — Platform Activation Status

Eyebrow: "Platform health"
Heading: "What's live, what's not, and how you compare."

**Health score banner** — dark background (`{on_surface}`), two-column layout:
- Left: large score number (`{overall_score}/10`) in `{secondary}` colour, label "Platform Health"
- Right: heading + 2-sentence summary of what the score means + health pills (green/amber/red) summarising the key themes

**Column header row** above the activation list:
`Product | Status | Peer adoption | — | Score`

**Activation list** — one row per tracked product. CSS grid: `32px 200px 96px 1fr 140px 64px`
Per row:
- Status icon (✓ green / ✗ red / ? amber)
- Product name
- Status badge (Active / Not active / Verify)
- Peer adoption bar (filled `{primary}`, background `#e5e7eb`, width = benchmark %)
- Peer adoption % label
- Per-product score (X/10, colour-coded: green ≥8, amber 4–7, red ≤3)

**Score legend** below the list:
- Green 8–10: Active and well adopted by peers
- Amber 4–7: Opportunity — worth activating
- Red 1–3: Critical gap — high peer adoption, not yet live

### Section 4 — Payments Deep Dive

Eyebrow: "Payments deep dive"
Heading: "Getting the most from Shopify Payments."
Sub: "The full checklist — from activation through to multi-currency, wallets, and BNPL."

Dark background section (`{on_surface}`).

**4a — Activation checklist**
Visual checklist (tick/cross/warning icon per item):
- Shopify Payments active
- Shop Pay enabled
- Shop Pay Instalments (SPI) active
- Apple Pay enabled
- Google Pay enabled
- Multi-currency via Markets
- Local payment methods configured for your markets
- Fraud protection configured
- Payout schedule reviewed

**4b — Shop Pay Instalments spotlight**
Card with `{secondary}` accent. Content:
- "SPI lets customers spread the cost of larger purchases — no extra cost to you, no risk. For merchants selling premium products (high AOV), it directly reduces purchase hesitation."
- Stat: "Merchants in home, lifestyle and premium DTC categories typically see conversion uplift on high-AOV items after enabling SPI."
- First step: Enable in Admin → Settings → Payments → Buy now, pay later

**4c — Multi-currency & local payment methods**
Two-column layout:
- Left: Explain Shopify Markets multi-currency — single storefront, local pricing, local checkout currency, automatic FX
- Right: LPM grid by market (show which LPMs are available for the merchant's active/target markets based on their country data)

**4d — Wallets**
Three wallet cards (Apple Pay, Google Pay, Shop Pay) — each with: what it is, who uses it, why it converts better on mobile. Stat where available (e.g. "Shop Pay has a 91% peer adoption rate among similar Plus merchants").

### Section 5 — Gaps & Benefits

Eyebrow: "Gaps & benefits"
Heading: "What to activate next, and why it matters."
Sub: "Each gap explained — what it is, how to turn it on, what it does for the business."

One card per product in `{gap_products}`. For each card:
- Product name + score badge
- **What it is**: one sentence
- **How to activate**: specific Admin path or first step
- **Business benefit**: 2–3 sentences on the commercial impact, tailored to the merchant's industry
- **Peer adoption**: "X% of similar Plus merchants use this" with a mini bar
- **Case study** (if found in Step 4): merchant type + key outcome in a tinted callout box. If not found: "Case study coming soon."

Do not reuse wording from other merchants' reports — write benefit copy fresh for this merchant's industry and product mix.

### Section 6 — AI Plays

Eyebrow: "AI plays"
Heading: "What's running, and what's still on the shelf."
Sub: "The AI surfaces inside Shopify you can use without leaving the platform."

Two columns:
- **Already running** (light background): AI features confirmed active
- **Worth switching on** (dark `{on_surface}` background): highest-leverage untapped AI features

AI features to reference: Sidekick (admin Q&A, analytics, segment building), Shopify Magic (product descriptions, image alt-text, email subject lines, theme copy), Search & Discovery (semantic search), Shop Reviews (AI moderation)

### Section 7 — Resources

Eyebrow: "Resources"
Heading: "Tools and channels to get more from Shopify."

Four resource cards (2×2 grid):
1. **Sidekick** — "Your AI assistant inside Shopify admin. Ask it anything: sales data, customer segments, product insights. Available in Admin → Assistant."
2. **Help Centre** — "Shopify's full documentation library. Step-by-step guides for every feature. help.shopify.com"
3. **Shopify Plus Academy** — "Exclusive training and certification for Plus merchants and their teams. Covers platform features, strategy, and best practices."
4. **Shopify Changelog** — "Stay up to date with every platform update, new feature, and improvement. shopify.com/changelog"

### Footer
Co-brand lockup (mini), date. Footer reads: "Prepared by **Shopify Customer Success**" — quiet and understated.

---

## File output

Save to: `~/{merchant_domain}-platform-review.html`
Open in browser: `open ~/{merchant_domain}-platform-review.html`

Then post a brief chat summary:
```
# Platform Review — {Merchant Name}

**File:** ~/{merchant_domain}-platform-review.html

## Platform health
Overall score: {X.X}/10

## Activation snapshot
✅ Active ({n}): {list}
⬜ Gap ({n}): {list}

## Payments checklist
[tick/cross per item]

## Top 3 gaps by score
1. ...
2. ...
3. ...
```

If any data source is unavailable, note it in the summary and continue. Do not fail silently.
