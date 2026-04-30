---
name: merchant-onboarding-report
description: "Generates a branded best practice report for new merchant onboarding — activation status, post-activation optimisation across Shopify Payments, Shop Pay, B2B, Retail/POS, and more, with case studies from similar merchants. Styled using the merchant's design.md. Output is a Google Slides deck."
---

# Merchant Onboarding Report

Produces a branded, CS-ready best practice report for a merchant that has launched on Shopify. Covers current activation status, product-by-product optimisation recommendations, and case studies from comparable merchants. Styled using the merchant's live design.md.

## Arguments

The user must pass a **merchant domain or name** (e.g., `/merchant-onboarding-report pandalondon.com` or `/merchant-onboarding-report Panda London`).

If no argument is provided, ask the user for the merchant domain before proceeding.

Store as `{merchant_domain}` (normalise to domain format, e.g. `pandalondon.com`).

---

## Steps

Run steps 1–4 in parallel, then compile into the deck.

### 1. Fetch merchant design.md

Fetch: `https://designmd.quick.shopify.io/{merchant_domain}`

Extract and store:
- `{primary_color}` — primary brand colour (hex)
- `{secondary_color}` — secondary/accent colour (hex)
- `{neutral_color}` — background/neutral colour (hex)
- `{on_surface_color}` — text colour (hex)
- `{headline_font}` — headline font family
- `{body_font}` — body font family
- `{brand_aesthetic}` — one-sentence aesthetic description
- `{merchant_logo_url}` — logo URL if present
- `{deck_hint}` — deck styling guidance if present
- `{voice_tone}` — brand voice/tone

If the design.md fetch fails, use Shopify brand defaults:
- primary: `#008060`, secondary: `#5C6AC4`, neutral: `#F6F6F1`, on-surface: `#202223`
- headline: `ShopifySans`, body: `ShopifySans`

### 2. Pull merchant data

Call `search_data_tool` with dataset `Shops`.
Filter: `tolower(myshopify_domain) eq '{merchant_domain}'` OR `contains(tolower(myshopify_domain), '{merchant_name}')`
Select: `shop_id, myshopify_domain, plan_name, gmv_usd_l365d, revenue_l12m, risk_score_label, cs_health_score, msm_name, created_at`

Call `search_data_tool` with dataset `Accounts`.
Filter: `contains(tolower(account_name), '{merchant_name}')`
Select: `account_name, msm_name, gmv_usd_l365d, risk_score_label, cs_health_score, active_products`

Call `search_salesforce_tool`:
```sql
SELECT Id, Name, BillingCountry, Industry, AnnualRevenue,
       NumberOfEmployees, Description
FROM Account
WHERE RecordType.Name = 'Merchant'
AND Name LIKE '%{merchant_name}%'
LIMIT 1
```

Store: GMV, plan, health score, active products, account ID, country.

### 3. Assess product activation

Using `active_products` from step 2, map activation status for each product area:

| Product | Active? | Signal to check |
|---|---|---|
| Shopify Payments | ✅ / ❌ | `active_products` contains 'payments' |
| Shop Pay | ✅ / ❌ | `active_products` contains 'shop_pay' |
| B2B | ✅ / ❌ | `active_products` contains 'b2b' |
| Retail / POS | ✅ / ❌ | `active_products` contains 'pos' |
| Shopify Markets | ✅ / ❌ | `active_products` contains 'markets' |
| Shopify Shipping | ✅ / ❌ | `active_products` contains 'shipping' |
| Shopify Capital | ✅ / ❌ | `active_products` contains 'capital' |
| Shopify Balance | ✅ / ❌ | `active_products` contains 'balance' |

If `active_products` is unavailable, mark all as unknown and note the gap.

Build two lists:
- `{activated_products}` — confirmed active
- `{gap_products}` — not yet active (priority recommendations)

### 4. Pull case studies

For each product in `{gap_products}`, search for 1–2 relevant case studies from comparable merchants.

Call `search_sales_call_transcripts` for each product:
- Queries: e.g. for Shopify Payments: `['merchant activated Shopify Payments results', 'switched to Shopify Payments conversion uplift', 'Shopify Payments success story']`
- Filter: `plan_name` matching merchant's plan tier; `opportunity_result: 'closed_won'`
- `search_mode: 'summaries'`
- `limit: 3`

Call `search_media_insights` for each product:
- Keywords: e.g. `'Shopify Payments'`, `'Shop Pay checkout'`, `'POS retail'`
- `sentiment: 'Positive'`
- `limit: 3`

For each case study found, extract:
- Merchant name (anonymise if needed)
- Product adopted
- Key outcome / metric (conversion lift, revenue uplift, time saved)
- One-line quote if available

If no case studies found for a product, note "case study pending" and suggest the CSM adds one manually.

---

## Recommendation logic per product

For each product in `{gap_products}`, generate a recommendation using this structure:

**Shopify Payments:**
- Why: eliminate third-party gateway fees, unlock Shop Pay, simplify reconciliation
- Quick win: estimated fee saving based on GMV (use 0.5–2% uplift assumption)
- First step: enable in Admin → Settings → Payments

**Shop Pay:**
- Why: accelerated checkout, higher conversion, access to Shop Pay Instalments
- Quick win: avg 18% higher checkout conversion for returning customers
- First step: activate via Shopify Payments settings; add Shop Pay button to product pages

**B2B:**
- Why: wholesale pricing, company accounts, net payment terms, custom catalogues
- Quick win: relevant if merchant has trade/wholesale buyers — unlocks new revenue channel
- First step: assess whether merchant has B2B buyers, then enable B2B on Plus

**Retail / POS:**
- Why: unified inventory, in-person sales, omnichannel reporting
- Quick win: relevant for merchants with physical locations or pop-ups
- First step: POS Pro trial, hardware review

**Shopify Markets:**
- Why: multi-currency, local domains, duty/tax management for cross-border
- Quick win: relevant for merchants selling internationally (check Salesforce BillingCountry + GMV distribution)
- First step: enable Markets in Admin, set up local pricing

**Shopify Shipping:**
- Why: discounted rates, label printing, returns portal, tracking pages
- Quick win: avg 10–25% carrier discount vs retail rates
- First step: enable in Admin → Settings → Shipping and delivery

**Shopify Capital:**
- Why: revenue-based funding for inventory, marketing, growth
- Quick win: relevant if merchant has seasonal GMV spikes or growth plans
- First step: check Capital eligibility in Admin → Finances

**Shopify Balance:**
- Why: business account, cashback rewards, faster payouts
- Quick win: available for Shopify Payments merchants
- First step: activate in Admin → Finances → Balance

---

## Build the Google Slides deck

Create a new Google Slides presentation titled: `{Merchant Name} — Shopify Platform Review — {today's date}`

Use `create_file` with `file_type: presentation`.

Then use `batch_workspace_operations` with service `slides` to build the following slides. Apply `{primary_color}`, `{secondary_color}`, `{neutral_color}`, `{on_surface_color}` from the merchant's design.md throughout. Use `{headline_font}` for titles and `{body_font}` for body text where possible (fall back to Arial if custom fonts are unavailable in Slides).

### Slide structure

**Slide 1 — Cover**
- Background: `{neutral_color}`
- Merchant logo (if `{merchant_logo_url}` available) top-left
- Shopify logo top-right with thin vertical divider between them
- Headline (Futura-Heavy or headline font, large): `{Merchant Name} × Shopify`
- Subheadline: `Platform Optimisation Review`
- Date bottom-right in muted text
- Accent bar at bottom: `{primary_color}` full-width, height 8px

**Slide 2 — Where You Are Today**
- Title: `Your Platform Today`
- Left panel (60%): activation status table — two columns (Product | Status), use ✅ for active, ⬜ for not yet active. Style active rows with `{primary_color}` chip/badge.
- Right panel (40%): key stats — GMV L365D, plan, health score, days since launch
- Background: white; section header bar: `{primary_color}`

**Slide 3 — Activation Highlights**
- Title: `What's Working Well`
- One card per activated product (max 4 per slide, overflow to next slide)
- Card style: white card, `{primary_color}` icon/accent, product name bold, one-line benefit
- Background: `{neutral_color}`

**Slide 4–N — Optimisation Opportunities (one slide per gap product)**
For each product in `{gap_products}`, create one slide:
- Title: product name (e.g., `Shopify Payments`)
- Left column:
  - Subheading: `Why it matters`
  - 2–3 bullet benefits
  - Subheading: `Quick win`
  - One metric or outcome
  - Subheading: `First step`
  - One clear action
- Right column: case study box
  - Heading: `Merchant Story` (or `Case Study`)
  - Merchant name, product, key outcome, quote if available
  - Style: `{secondary_color}` background, `{on_surface_color}` text
- Slide accent: `{primary_color}` left border bar (8px wide, full height)
- Background: white

**Slide N+1 — Recommended Next Steps**
- Title: `Your Next 30 Days`
- Numbered list (max 5 items), one per gap product — action + owner (CSM name from step 2)
- Timeline visual: simple horizontal bar with 3 phases (Week 1 / Week 2–3 / Week 4)
- Call to action: `[CSM name] will follow up on [date]`
- Background: `{primary_color}`; text: white or `{on_surface_color}` depending on contrast

**Slide N+2 — Appendix: Resources**
- Title: `Helpful Resources`
- Links to Shopify Help docs for each gap product
- Background: `{neutral_color}`; muted text

After creating all slides, return the Google Slides URL.

---

## Output summary (in chat)

After building the deck, post a brief summary:

```
# Merchant Onboarding Report — {Merchant Name}

**Deck:** [Google Slides link]

## Activation Status
✅ Active ({n}): {list}
⬜ Not yet active ({n}): {list}

## Top 3 Recommendations
1. {Product} — {one-line rationale}
2. {Product} — {one-line rationale}
3. {Product} — {one-line rationale}

## Case Studies Found
{Product}: {merchant} — {outcome}
{Product}: case study pending

## Data gaps
{any missing data noted here}
```

If any step fails, note it and continue. Do not fail silently.
