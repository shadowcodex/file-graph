---
name: Cascade Analytics
description: Mid-market customer on growth plan, recently migrated to annual billing
explicit_edges:
  "people/jordan-taylor.md": "Jordan handled their billing migration"
  "products/features/billing.md": "Their plan migration surfaced several proration bugs"
---

# Cascade Analytics

Cascade is a mid-market #customer on the growth plan — they're a data analytics consultancy that uses Acme to aggregate client datasets. Solid customer, low-touch, rarely files support tickets. They've been with Acme since mid-2024.

The notable thing about Cascade is their billing migration. When they switched from monthly to annual billing in January 2025, [[people/jordan-taylor.md|Jordan]] discovered that the proration logic in the [[products/features/billing.md|billing system]] didn't handle mid-cycle plan changes correctly when combined with usage overages. Fixing it took about a week and the patches are now permanent fixtures in the billing codebase.

They're happy, they pay on time, and they're the kind of customer you wish you had more of.

## Account Details

```csv
field,value
plan,growth (annual)
arr,$2388
contract_start,2025-01-15
data_volume,~5M events/day
primary_contact,Nina Patel (Head of Engineering)
```
