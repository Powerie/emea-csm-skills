---
name: merchant-onboarding-report
description: "Generates a branded HTML best practice report for a Shopify Plus merchant — explains their CS team, covers their full platform stack by theme, delivers a Shopify Payments deep dive (Shop Pay, SPI, multi-currency, LPMs, wallets), benchmarks them against peers, and ranks opportunities by impact. Output is a self-contained HTML quick-site styled with the merchant's design.md."
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
Select: `account_name, msm_name, gmv_usd_l365d, revenue_l12m, gmv_growth_yearly, adopted_products, eligible_products, b2b_fit_score, d2c_fit_score, retail_fit_score, is_b2b, is_d2c, is_retail, country, industry, employee_count, account_id, banff_risk_level`

Call `search_salesforce_tool`:
```sql
SELECT Id, Name, BillingCountry, Industry, AnnualRevenue, NumberOfEmployees
FROM Account
WHERE RecordType.Name = 'Merchant'
AND Name LIKE '%{merchant_name}%'
LIMIT 1
```

Store: `{gmv}`, `{plan}`, `{msm_name}`, `{country}`, `{adopted_products}`, `{eligible_products}`, `{risk_level}`.

### 3. Map product activation + peer benchmarks

Using `adopted_products` and `eligible_products`, build two lists:
- `{active_products}` — confirmed adopted
- `{gap_products}` — eligible but not yet adopted

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
Stamp: "Shopify Review · {date}" in small muted text.

### Section 1 — Hero
Dark background (`{on_surface}`). Primary accent bar at bottom (`{primary}`, 6px).

Headline (large, headline font): a single evocative sentence about where growth comes from — calm, brand-appropriate, not sales-y. Match the merchant's voice tone.

Sub-headline: "A look at what your Shopify stack is doing well, where the opportunities are, and the moves that pay back fastest."

KPI row (3 stats):
- GMV L365D
- YoY GMV growth %
- Products active (X / total eligible)

### Section 2 — Your Success Team

Two cards side by side, white background, `{primary}` left border accent.

**Card 1 — Your Merchant Success Manager**
- Heading: "Your Merchant Success Manager"
- Name: `{msm_name}`
- Body: "Your MSM is your dedicated Shopify strategist. They help you get more from the platform — finding the features that fit your growth stage, connecting you with the right resources, and meeting regularly to review what's working and what's next. Think of them as a business partner who knows Shopify inside out."
- Bullet list of what your MSM does:
  - Proactive platform reviews and quarterly business reviews (QBRs)
  - Connects you to Shopify experts, partners, and beta programmes
  - Helps prioritise which features to adopt next and in what order
  - Your first call when something isn't working the way it should

**Card 2 — Shopify Plus Support**
- Heading: "Your Plus Support"
- Body: "As a Shopify Plus merchant, you have access to priority support — 24/7, with faster response times and agents who know the Plus platform deeply."
- Bullet list:
  - 24/7 priority support via chat, email, and phone
  - Dedicated Plus support agents — not general-tier
  - Launch Engineer access for major builds, migrations, and peak events
  - Shopify Plus Academy — exclusive training and certification
  - Early access to beta features and product previews
  - Proactive outreach around platform updates that affect your store

### Section 3 — The Headlines

Eyebrow: "The headlines"
Heading: "Six things worth knowing."
Sub: "Two strengths, two gaps, two opportunities — a quick read before we go deeper."

Generate 6 headline cards based on the merchant's data:
- 2 × **Strong** (green tag): things the merchant is doing well, drawn from `{active_products}`, `{gmv_growth}`, or strong fit scores
- 2 × **Gap** (amber tag): products in `{gap_products}` or low fit-score areas with clear improvement potential
- 2 × **Opportunity** (blue/primary tag): longer-term moves worth exploring (Markets consolidation, new channels, AI features)

Card format: tag badge + bold headline (one sentence) + two-line explanation.

### Section 4 — The Range

Eyebrow: "The range"
Heading: "Your stack, grouped by what it does."
Sub: "Each section shows what's active and a few worth-a-look additions for a brand like yours."

Render theme cards — one per category. For each card:
- Category name + count of active tools
- One-line summary of what's live
- "In your stack" chip row (green chips) — from `{active_products}`
- "Worth a look" chip row (amber/outline chips) — from `{gap_products}` + general recommendations

Categories to cover:
1. **Payments & Checkout** — Shopify Payments, Shop Pay, SPI, wallets, BNPL, Checkout Extensibility
2. **AI & Automation** — Shopify Flow, Launchpad, Sidekick, Shopify Magic
3. **Search & Discovery** — Search & Discovery app, Combined Listings, Markets
4. **Marketing & Channels** — Shopify Email, social channels, Shop channel
5. **Loyalty & Reviews** — Shop Reviews, loyalty apps
6. **B2B & Operations** — B2B, Plus Store Operations, Matrixify, Order Routing
7. **Retail & Fulfilment** — POS Pro, Shopify Shipping, fulfilment apps
8. **Analytics & Data** — Shopify Analytics, Sidekick queries, reporting

### Section 5 — Payments Deep Dive

Eyebrow: "Payments deep dive"
Heading: "Getting the most from Shopify Payments."
Sub: "The full checklist — from activation through to multi-currency, wallets, and BNPL."

Dark background section (`{on_surface}`).

**5a — Activation checklist**
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

**5b — Shop Pay Instalments spotlight**
Card with `{secondary}` accent. Content:
- "SPI lets customers spread the cost of larger purchases — no extra cost to you, no risk. For merchants selling premium products (high AOV), it directly reduces purchase hesitation."
- Stat: "Merchants in home, lifestyle and premium DTC categories typically see conversion uplift on high-AOV items after enabling SPI."
- First step: Enable in Admin → Settings → Payments → Buy now, pay later

**5c — Multi-currency & local payment methods**
Two-column layout:
- Left: Explain Shopify Markets multi-currency — single storefront, local pricing, local checkout currency, automatic FX
- Right: LPM grid by market (show which LPMs are available for the merchant's active/target markets based on their country data)

**5d — Wallets**
Three wallet cards (Apple Pay, Google Pay, Shop Pay) — each with: what it is, who uses it, why it converts better on mobile. Stat where available (e.g. "Shop Pay has a 91% peer adoption rate among similar Plus merchants").

### Section 6 — Peer Adoption

Eyebrow: "Adoption vs peers"
Heading: "How your stack compares to similar Plus merchants."
Sub: "Bars show % of comparable EMEA Plus merchants running each tool. The marker shows your status."

Render horizontal bar chart rows for each benchmark product. Three statuses:
- **In your stack** — `{primary}` marker, green label
- **Worth considering** — amber/warning marker and label
- **Covered by alternative** — muted marker and label

Use the fixed benchmark percentages from Step 3.

### Section 7 — Top 5 Opportunities

Eyebrow: "Top five"
Heading: "The opportunities ranked by impact."
Sub: "Each paired with an effort estimate and, where relevant, an AI play."

Generate 5 ranked opportunities from `{gap_products}` and platform analysis. For each:
- Rank number (large, `{primary}`)
- Bold headline
- 2–3 sentence explanation
- Pill tags: impact level (High/Medium/Low), effort level (High/Medium/Low)
- AI play (optional): how Sidekick or Shopify Magic could support this

### Section 8 — AI Plays

Eyebrow: "AI plays"
Heading: "What's running, and what's still on the shelf."
Sub: "The AI surfaces inside Shopify you can use without leaving the platform."

Two columns:
- **Already running** (light background): AI features confirmed active
- **Worth switching on** (dark `{on_surface}` background): highest-leverage untapped AI features

AI features to reference: Sidekick (admin Q&A, analytics, segment building), Shopify Magic (product descriptions, image alt-text, email subject lines, theme copy), Search & Discovery (semantic search), Shop Reviews (AI moderation)

### Section 9 — Best Practices

Eyebrow: "Best practices"
Heading: "Habits that work for merchants like you."
Sub: "Drawn from how the strongest {industry} Plus merchants run their stacks."

Generate 6–7 best practice bullets tailored to the merchant's industry and product mix. Make them specific and actionable — not generic tips.

### Section 10 — Next Steps

Dark background (`{on_surface}`). Eyebrow: "Next steps". Heading: "What we'll do together."
Sub: "A short, practical list. A sensible order to take them in."

Numbered list (5–6 items) — collaborative tone, first-person plural ("we'll"). Each step references a specific opportunity from Section 7.

Final line: "We'll reconvene in six weeks to review signal and prioritise the next two moves."

### Footer
Co-brand lockup (mini), date, "prepared with care" — quiet and understated.

---

## File output

Save to: `~/{merchant_domain}-platform-review.html`
Open in browser: `open ~/{merchant_domain}-platform-review.html`

Then post a brief chat summary:
```
# Platform Review — {Merchant Name}

**File:** ~/{merchant_domain}-platform-review.html

## Activation snapshot
✅ Active ({n}): {list}
⬜ Gap ({n}): {list}

## Payments checklist
[tick/cross per item]

## Top 3 opportunities
1. ...
2. ...
3. ...
```

If any data source is unavailable, note it in the summary and continue. Do not fail silently.
