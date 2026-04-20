---
name: bankrate-databricks
description: Query and analyze Bankrate Databricks data warehouse
version: 1.0.0
authors:
  - Bankrate Analytics Team
---

# Bankrate Databricks Analytics Skill

A collaborative skill for querying and analyzing Bankrate's Databricks data warehouse.

## Environment

- **Warehouse ID**: `a2a6f171e901de84`
- **Catalog**: `bankrate_prod`
- **Schema**: `br_rpt`

## Available Commands

When the user requests analytics or data queries, use the Databricks API to execute SQL statements:

```bash
databricks api post /api/2.0/sql/statements --json '{
  "warehouse_id": "a2a6f171e901de84",
  "statement": "YOUR_SQL_HERE",
  "wait_timeout": "30s"
}'
```

## Common Query Patterns

### Top Pages Analysis
When asked for "top pages", "most visited pages", or "popular pages":

```sql
SELECT
  pagenoquerystring,
  COUNT(DISTINCT instanceid) as pageviews
FROM bankrate_prod.br_rpt.pageview_fact
WHERE sentatest::date >= DATEADD(day, -7, CURRENT_DATE)
GROUP BY 1
ORDER BY 2 DESC
LIMIT 10
```

### Revenue by Vertical
When asked for "revenue", "conversions by vertical", or "vertical performance":

```sql
SELECT
  c_vertical,
  sum(c_actualrevenue) as total_revenue,
  count(*) as conversion_count,
  sum(c_actualrevenue) / count(*) as avg_revenue_per_conversion
FROM bankrate_prod.br_rpt.conversions_core
WHERE c_sentat::date >= DATEADD(day, -7, CURRENT_DATE)
GROUP BY 1
ORDER BY 2 DESC
```

### Form Funnel Analysis
When asked for "funnel", "form drop-off", or "conversion funnel":

```sql
SELECT
  form_step.STEPNAME,
  COUNT(DISTINCT form_step.MESSAGEID) as users
FROM bankrate_prod.br_rpt.form_summary
LEFT JOIN bankrate_prod.br_rpt.form_step
  ON form_summary.FORMID = form_step.FORMID
  AND form_step.formcontinued = 1
WHERE form_summary.formname = 'Cobrand'
  AND form_summary.formloanpurpose = 'Purchase'
  AND form_summary.sentat::date >= DATEADD(day, -7, CURRENT_DATE)
GROUP BY 1
ORDER BY 2 DESC
```

### Landing Page Continuation
When asked for "continuation rate", "bounce rate", or "multi-page sessions":

```sql
WITH top_pages AS (
  SELECT pagenoquerystring
  FROM bankrate_prod.br_rpt.pageview_fact
  WHERE sentatest::date >= DATEADD(day, -7, CURRENT_DATE)
  GROUP BY 1
  ORDER BY COUNT(DISTINCT instanceid) DESC
  LIMIT 10
),
session_pageviews AS (
  SELECT
    s.sessionid,
    s.firstpagenoquerystring,
    COUNT(DISTINCT p.instanceid) as pageviews_in_session
  FROM bankrate_prod.br_rpt.session_fact s
  LEFT JOIN bankrate_prod.br_rpt.pageview_fact p
    ON s.sessionid = p.sessionid
  WHERE s.sessionstarttimeest::date >= DATEADD(day, -7, CURRENT_DATE)
    AND p.sentatest::date >= DATEADD(day, -7, CURRENT_DATE)
    AND s.firstpagenoquerystring IN (SELECT pagenoquerystring FROM top_pages)
  GROUP BY 1, 2
)
SELECT
  firstpagenoquerystring,
  COUNT(*) as total_sessions,
  SUM(CASE WHEN pageviews_in_session > 1 THEN 1 ELSE 0 END) as sessions_with_multiple_pages,
  ROUND(SUM(CASE WHEN pageviews_in_session > 1 THEN 1 ELSE 0 END) * 100.0 / COUNT(*), 1) as continuation_rate
FROM session_pageviews
GROUP BY 1
ORDER BY 2 DESC
```

### Demographics Analysis
When asked for "user demographics", "lead quality", or "credit scores":

```sql
SELECT
  form_summary.formid,
  session_fact.return_visitor,
  MAX(CASE WHEN form_field.fieldname = 'creditScore' THEN form_field.fieldvalue END) AS creditscore,
  MAX(CASE WHEN fieldname = 'employmentStatus' THEN fieldvalue END) AS employmentstatus,
  MAX(CASE WHEN fieldname = 'purchaseStage' THEN fieldvalue END) AS purchaseStage,
  MAX(CASE WHEN fieldname = 'combinedAnnualIncome' THEN fieldvalue END) AS grossIncome
FROM bankrate_prod.br_rpt.form_summary
LEFT JOIN bankrate_prod.br_rpt.form_field
  ON form_summary.FORMID = form_field.FORMID
LEFT JOIN bankrate_prod.br_rpt.session_fact
  ON form_summary.sessionid = session_fact.sessionid
WHERE form_summary.formname = 'Cobrand'
  AND form_summary.formloanpurpose = 'Purchase'
  AND form_field.fieldname IS NOT NULL
  AND form_summary.sentat::date >= DATEADD(day, -7, CURRENT_DATE)
GROUP BY ALL
```

## Tables Reference

### Traffic Tables
- `pageview_fact` - Individual page views (use `instanceid` for distinct counts)
- `session_fact` - User sessions (includes `devicetype`, `return_visitor`)

### Form Tables
- `form_summary` - Form metadata (join via `FORMID` or `sessionid`)
- `form_step` - Step-by-step funnel data
- `form_field` - Field values (use PIVOT pattern with MAX/CASE)

### Conversion Tables
- `conversions_core` - Revenue and conversion events by vertical

### Interaction Tables
- `element_fact` - Button/link click tracking

## Instructions

1. **Execute queries** using the Databricks API endpoint
2. **Present results** in clear tables with insights
3. **Adjust timeframes** based on user requests (default: 7 days)
4. **Format large numbers** with commas for readability
5. **Calculate percentages** and rates where relevant
6. **Provide context** - explain what the data means for the business

## Tips

- Use `::date` to cast timestamps to dates
- Use `DATEADD(day, -N, CURRENT_DATE)` for relative date filtering
- Escape single quotes in bash: `'\''`
- For long queries, increase `wait_timeout` if needed
- Always use `DISTINCT instanceid` for pageview counts
- Always use `DISTINCT sessionid` for session counts

## Contributing

To update this skill:
1. Clone the repo: `git clone <repo-url>`
2. Make your changes to `skill.md`
3. Test with Claude Code
4. Commit and push: `git commit -m "Add new query pattern" && git push`
5. Team members pull updates: `git pull`
