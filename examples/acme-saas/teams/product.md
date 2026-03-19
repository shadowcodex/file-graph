---
name: Product Team
description: Frontend and product engineering — builds the dashboard, billing UI, and customer-facing features
explicit_edges:
  "people/alex-kim.md": "Tech lead, owns the dashboard"
  "people/jordan-taylor.md": "Engineer, billing and payments"
  "products/features/dashboard.md": "Primary owners of the customer dashboard"
  "products/features/billing.md": "Primary owners of the billing system"
  "projects/usage-analytics.md": "Building the new usage analytics feature"
---

# Product Team

The #product team builds everything customers actually see and interact with — the [[products/features/dashboard.md|dashboard]], [[products/features/billing.md|billing system]], and any new customer-facing #frontend features. They work closely with design and product management to ship what customers are actually asking for.

Two engineers right now. They've been asking for a third hire since Q4 but headcount is frozen until ARR hits the next milestone.

## Current Focus

The big push is [[projects/usage-analytics.md|usage analytics]] — customers have been asking for visibility into their API consumption for months. The beta is targeting mid-April with [[customers/meridian-health.md|Meridian Health]] as the design partner.

## Services Owned

```csv
service,repo,oncall_schedule,status
dashboard-app,acme/dashboard,weekly rotation,stable
billing-service,acme/billing,shared with platform,stable
```
