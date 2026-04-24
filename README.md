# EMEA CSM Skills

Claude skills for Customer Success teams at Shopify.

## Skills

### `bob-composition`
Pulls a live composition snapshot of any Book of Business — account counts, GMV distribution, risk tiers, critical accounts, and graduation pipeline. Works for any CSM or segment.

**Usage:** `/bob-composition [CSM name(s)]`

Examples:
- `/bob-composition` — infers from context or prompts you
- `/bob-composition Hannah Johnson` — single CSM
- `/bob-composition Hannah Johnson, Eoghan O'Keeffe` — multiple CSMs

---

### `cs-lead-review`
Full CS Lead book review — BoB health summary, CSM performance metrics (activity, pipeline, retention), and prioritised recommendations. Pulls from Revenue MCP, Salesloft, and Salesforce in parallel.

**Usage:** `/cs-lead-review [CSM name(s)]`

Examples:
- `/cs-lead-review` — infers from context or prompts you
- `/cs-lead-review Hannah Johnson` — single CSM review
- `/cs-lead-review Hannah Johnson, Eoghan O'Keeffe` — team-level review

**Output includes:**
- BoB health summary with risk distribution and GMV at risk
- CSM activity metrics (calls, emails, meetings — last 30 days)
- Pipeline summary (open opps, closed won/lost, open cases)
- Imminent churn watch
- Top 5 prioritised recommendations with suggested next actions
- Graduation pipeline status
