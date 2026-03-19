---
name: Usage Analytics
description: New dashboard feature giving customers visibility into API usage, data volume, and cost projections
explicit_edges:
  "teams/product.md": "Product team is building this"
  "people/alex-kim.md": "Leading the feature build"
  "customers/meridian-health.md": "Design partner for the beta"
  "products/features/dashboard.md": "New analytics tab in the dashboard"
  "products/features/billing.md": "Pulls usage data from the metering layer"
  "products/features/api-gateway.md": "Needs request-level metrics from the gateway"
---

# Usage Analytics

Usage analytics is the most customer-requested #feature on the roadmap — giving customers a #dashboard tab where they can see their API call volume, data ingestion throughput, and projected costs in real time. Right now customers have zero visibility into their usage, which means surprises on invoices and no way to detect ingestion issues on their end.

[[people/alex-kim.md|Alex]] is leading the build on the [[teams/product.md|Product team]]. [[customers/meridian-health.md|Meridian Health]] is the design partner — Alex runs biweekly demos with their data team to validate direction.

## What It Shows

- API request volume over time (hourly/daily/monthly)
- Data ingestion throughput and success rate
- Current billing period usage vs. plan included amounts
- Projected end-of-period cost based on current trajectory
- Alerting thresholds (notify me if usage exceeds X)

## Data Sources

The feature pulls from two existing systems:
- The [[products/features/api-gateway.md|API gateway]] already emits per-request metrics to Datadog — we need to make a subset of that available to customers
- The [[products/features/billing.md|billing system]]'s metering layer has the aggregated usage buckets

The main #technical challenge is making this data available to the dashboard without exposing internal Datadog metrics or building a whole new time-series store. Alex's current approach is to have a lightweight aggregation service that reads from the metering layer and caches in Redis.

## Timeline

```csv
milestone,target_date,status
design partner agreement (Meridian),2025-02-01,done
wireframes and data model,2025-02-28,done
backend aggregation service,2025-03-31,in progress
dashboard UI (charts + tables),2025-04-15,not started
alerting thresholds,2025-04-30,not started
beta with Meridian,2025-04-30,not started
GA rollout,2025-05-15,not started
```

## Risks

- The Product team bandwidth is split with the [[projects/auth-migration.md|auth migration]] (dashboard SSO flow). If auth slips, analytics slips.
- Meridian's contract renewal is September — having analytics live before then strengthens the renewal conversation.
