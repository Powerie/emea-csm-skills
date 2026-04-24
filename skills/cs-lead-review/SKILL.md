---
name: cs-lead-review
description: "Full CS Lead book review — BoB health summary, CSM performance metrics (activity, pipeline, retention), and prioritised recommendations. Pass a CSM name or comma-separated list; defaults to context. Use when running a weekly/monthly review, prepping for 1-1s, or QBR prep."
---

# CS Lead Review

Produces a structured review of a Book of Business from a CS Lead perspective: health snapshot, CSM performance, and ranked recommendations.

## Arguments

The user may pass:
- A **CSM name** (e.g., `/cs-lead-review Hannah Johnson`) — review that CSM's book
- A **comma-separated list** (e.g., `/cs-lead-review Hannah Johnson, Eoghan O'Keeffe`) — review multiple CSMs
- **No argument** — infer from context (CLAUDE.md, memory) or ask the user

Store the resolved CSM name(s) as `{csm_names}`.

---

## Steps

Run steps 1–4 in parallel, then compile the report.

### 1. BoB health snapshot

Call `search_data_tool` with dataset `Accounts`.

Filter by CSM:
- Single: `tolower(msm_name) eq '{csm_name}'`
- Multiple: `tolower(msm_name) eq 'name1' or tolower(msm_name) eq 'name2'`

**Risk distribution per CSM:**
```
groupby((msm_name, risk_score_label), aggregate(
  account_id with count as count,
  gmv_usd_l365d with sum as gmv
))
```

**Totals per CSM:**
```
groupby((msm_name), aggregate(
  gmv_usd_l365d with sum as total_gmv,
  revenue_l12m with sum as total_revenue,
  account_id with count as account_count
))
```

**Imminent churn accounts:**
Filter: CSM filter AND `tolower(risk_score_label) eq 'imminent churn'`
Select: `account_name, msm_name, gmv_usd_l365d, cs_health_score, risk_score_label`
Order by: `gmv_usd_l365d desc`

**High-risk accounts (top 10 by GMV):**
Filter: CSM filter AND `tolower(risk_score_label) eq 'high risk'`
Select: `account_name, msm_name, gmv_usd_l365d, cs_health_score`
Order by: `gmv_usd_l365d desc`
Top: 10

### 2. CSM activity metrics (Salesloft)

For each CSM, call `search_salesloft_tool` with `resource_type: people` to confirm the CSM's Salesloft identity, then pull recent activity:

Call `search_salesloft_tool` with `resource_type: activities`:
- Filter: last 30 days (`created_at[gte]: {date 30 days ago}`)
- Count by activity type: calls, emails, meetings

Call `search_salesloft_tool` with `resource_type: calls`:
- Filter: last 30 days
- Group by disposition to surface connection rates and outcomes
- Note any calls with negative sentiment

If Salesloft data is unavailable for a CSM, note it and continue.

### 3. Pipeline & retention metrics (Salesforce)

Call `search_salesforce_tool` with:

**Open opportunities:**
```sql
SELECT Id, Name, Account.Name, Amount, StageName, CloseDate, OwnerId
FROM Opportunity
WHERE RecordType.Name = 'Sales'
AND Owner.Name IN ({csm_names})
AND IsClosed = false
ORDER BY Amount DESC
```

**Closed won/lost (last 90 days):**
```sql
SELECT Id, Name, Account.Name, Amount, StageName, CloseDate, OwnerId
FROM Opportunity
WHERE RecordType.Name = 'Sales'
AND Owner.Name IN ({csm_names})
AND IsClosed = true
AND CloseDate = LAST_90_DAYS
ORDER BY CloseDate DESC
```

**Open cases (support burden):**
```sql
SELECT Id, Subject, Status, Priority, Account.Name, OwnerId, CreatedDate
FROM Case
WHERE Owner.Name IN ({csm_names})
AND IsClosed = false
ORDER BY CreatedDate DESC
LIMIT 20
```

### 4. Local CSV data (if available)

Check for CSV files in:
- `~/Downloads/Reports/` — look for `*Weekly*`, `*CRITICAL*`, `*GRADUATION*`, `*BoB*`
- Any path mentioned in CLAUDE.md or conversation context

Extract:
- Weekly tracker: row counts by risk level, flag accounts with no recent notes
- Critical accounts: health scores, last touch dates
- Graduation pipeline: candidates, blockers, target dates

If no CSVs found, note it and continue with MCP/Salesforce data only.

---

## Recommendations logic

After compiling all data, generate a **Top 5 Priorities** list using this ranking:

1. **Imminent churn accounts** — any account with `risk_score_label = 'Imminent Churn'`, ranked by GMV descending
2. **High-risk accounts with no recent activity** — high-risk accounts where Salesloft shows no touch in 14+ days
3. **Open opportunities past close date** — Salesforce opps with `CloseDate < today` and still open
4. **High-value accounts with declining health scores** — top-GMV accounts where `cs_health_score` is below 60
5. **Graduation candidates with no progression** — accounts in graduation pipeline with no forward movement in 30 days

For each priority, state: the account/issue, the risk, and the recommended next action.

---

## Output format

```
# CS Lead Review — {CSM name(s)} — {today's date}

## BoB Health Summary
| Metric | {CSM 1} | {CSM 2} | Total |
|---|---|---|---|
| Accounts | | | |
| GMV L365D | | | |
| Revenue L12M | | | |

## Risk Distribution
| Risk Tier | # Accounts | GMV at Risk |
|---|---|---|
| 🔴 Imminent Churn | | |
| 🟠 High Risk | | |
| 🟡 Medium Risk | | |
| 🟢 Low / Healthy | | |

## CSM Activity (Last 30 Days)
| Activity | {CSM 1} | {CSM 2} |
|---|---|---|
| Calls | | |
| Emails | | |
| Meetings | | |
| Call connection rate | | |

## Pipeline Summary
| | {CSM 1} | {CSM 2} |
|---|---|---|
| Open opps (value) | | |
| Closed won L90D | | |
| Closed lost L90D | | |
| Open cases | | |

## Imminent Churn Watch
| Account | CSM | GMV L365D | Health Score | Last Touch |
|---|---|---|---|---|

## Top 5 Priorities & Recommendations
1. [Account / Issue] — [Risk] — **Recommended action**
2. ...
3. ...
4. ...
5. ...

## Graduation Pipeline
Count of candidates + any with confirmed target dates or blockers noted.

## Flags
- CSMs with below-benchmark activity (< X calls/emails in 30 days)
- Accounts with open cases + high risk score
- Data gaps or unavailable sources noted here
```

If any data source is unavailable, note it clearly and continue. Do not fail silently.
