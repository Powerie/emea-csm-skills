---
name: bob-composition
description: "Pulls a live composition snapshot of any Book of Business — account counts, GMV distribution, risk tiers, critical accounts, and graduation pipeline. Pass a CSM name or use your own BoB. Use when reviewing BoB health, prepping for QBRs, or checking the current state of a segment."
---

# BoB Composition

Produces a structured snapshot of a Book of Business. Works for any CSM or segment.

## Arguments

The user may pass:
- A **CSM name** (e.g., `/bob-composition Hannah Johnson`) — analyse that person's BoB
- A **list of CSM names** (e.g., `/bob-composition Hannah Johnson, Eoghan O'Keeffe`) — aggregate across multiple CSMs
- **No argument** — ask the user whose BoB to analyse, or infer from context (e.g., CLAUDE.md or memory)

Store the resolved CSM name(s) as `{csm_names}` for use in queries below.

---

## Steps

Run steps 1–3 in parallel, then compile into the report.

### 1. Account counts, GMV & risk by CSM

Call `search_data_tool` with dataset `Accounts`.

Filter: one of:
- `tolower(msm_name) eq '{csm_name}'` for a single CSM
- `tolower(msm_name) eq 'name1' or tolower(msm_name) eq 'name2'` for multiple

**Aggregation — totals per CSM:**
```
groupby((msm_name), aggregate(
  gmv_usd_l365d with sum as total_gmv,
  revenue_l12m with sum as total_revenue,
  account_id with count as account_count
))
```

**Aggregation — risk tier breakdown:**
```
groupby((msm_name, risk_score_label), aggregate(
  account_id with count as count,
  gmv_usd_l365d with sum as gmv
))
```

**Top accounts — detail list:**
Select: `account_name, msm_name, gmv_usd_l365d, revenue_l12m, risk_score_label, cs_health_score`
Order by: `gmv_usd_l365d desc`
Top: 10

### 2. Local CSV files (if available)

Check for CSV files in common locations:
- `~/Downloads/Reports/` — look for files matching `*Weekly*`, `*CRITICAL*`, `*GRADUATION*`, `*BoB*`, `*book*`
- Any path mentioned in CLAUDE.md or conversation context

For each CSV found:
- Weekly tracker: count rows by risk level
- Critical accounts: list accounts with lowest health scores
- Graduations: count candidates and note target dates

If no CSVs are found, note it and continue with MCP data only.

### 3. Imminent churn watch

Call `search_data_tool` with dataset `Accounts`.
Filter: CSM filter AND `tolower(risk_score_label) eq 'imminent churn'`
Select: `account_name, msm_name, gmv_usd_l365d, risk_score_label, cs_health_score`
Order by: `gmv_usd_l365d desc`

---

## Output format

```
# BoB Composition — {CSM name(s)} — {today's date}

## Segment Overview
| Metric | {CSM 1} | {CSM 2} | Total |
|---|---|---|---|
| Accounts | ... | ... | ... |
| GMV L365D | ... | ... | ... |
| Revenue L12M | ... | ... | ... |

## Risk Distribution
| Risk Tier | # Accounts | GMV |
|---|---|---|
| 🔴 Imminent Churn | ... | ... |
| 🟠 High Risk | ... | ... |
| 🟡 Medium Risk | ... | ... |
| 🟢 Low / Healthy | ... | ... |

## Imminent Churn Watch
Account | CSM | GMV L365D | Health Score

## Top 10 Accounts by GMV
Account | CSM | GMV L365D | Risk Label

## Critical Accounts (from CSV, if available)
Account | CSM | Health Score | GMV | Notes

## Graduation Pipeline (from CSV, if available)
Count of candidates + any with confirmed target dates.

## Gaps / Flags
- Accounts with no weekly notes logged (if CSV available)
- Data discrepancies between CSV and Copilot counts
```

If any data source is unavailable, note it clearly and continue with what's available. Do not fail silently.
