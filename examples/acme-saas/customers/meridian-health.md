---
name: Meridian Health
description: Enterprise healthcare customer, largest account by data volume, design partner for usage analytics
explicit_edges:
  "people/alex-kim.md": "Biweekly design partner syncs for usage analytics"
  "projects/usage-analytics.md": "Design partner for the beta"
  "incidents/2025-02-14-pipeline-outage.md": "Triggered and was most impacted by the Valentine's Day outage"
  "products/features/data-pipeline.md": "Heaviest user of the ingestion pipeline"
---

# Meridian Health

Meridian Health is Acme's largest #enterprise customer by data volume — they push roughly 400M events/day through the [[products/features/data-pipeline.md|data pipeline]], accounting for about 20% of total platform throughput. They're a regional #healthcare network using Acme to aggregate patient intake data across 14 hospital sites.

They've been a customer since early 2024 and are on a custom #enterprise plan. The relationship is strong but took a hit after the [[incidents/2025-02-14-pipeline-outage.md|Valentine's Day outage]], which they inadvertently triggered with a bulk historical re-ingestion. They lost ~3 hours of live patient intake data, which required manual re-entry by their clinical staff. Not great.

On the positive side, [[people/alex-kim.md|Alex]] has been running biweekly design partner sessions with their data team for the [[projects/usage-analytics.md|usage analytics]] feature. They're excited about it — they currently have zero visibility into their ingestion throughput and have been asking for it since onboarding.

## Account Details

```csv
field,value
plan,enterprise (custom)
arr,$285000
contract_renewal,2025-09-01
data_volume,~400M events/day
primary_contact,Dana Wright (VP of Data)
csm,Rachel Ng
```

## Key Concerns

- They need better visibility into ingestion health (usage analytics will help)
- Still waiting on the automated backfill tooling promised after the outage
- Their contract renewal is September — the outage response and analytics feature will be factors
