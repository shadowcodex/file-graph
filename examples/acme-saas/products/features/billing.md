---
name: Billing System
description: Usage-based billing system integrated with Stripe, handles metering, invoicing, and plan management
explicit_edges:
  "teams/product.md": "Owned by the Product team"
  "people/jordan-taylor.md": "Sole owner of the billing codebase"
  "products/features/api-gateway.md": "Receives usage metrics from the gateway for metering"
  "products/features/dashboard.md": "Billing UI lives in the dashboard"
  "customers/cascade-analytics.md": "Custom annual billing logic was built for this customer"
---

# Billing System

The billing system handles everything #payments-related at Acme — usage metering, invoice generation, plan management, and the Stripe integration. It's a #critical-path system: if billing breaks, customers can't upgrade, downgrade, or (more importantly from a revenue perspective) pay us.

[[people/jordan-taylor.md|Jordan]] is the sole owner, which is a known #bus-factor risk that comes up every retro. The Stripe integration is solid but the usage-based pricing logic has accumulated some gnarly edge cases, particularly around proration when customers change plans mid-cycle.

## How It Works

The [[products/features/api-gateway.md|API gateway]] emits usage events (request counts, data volume) to a metering service, which aggregates them into per-customer hourly buckets. At the end of each billing cycle, the billing service reads those buckets, calculates charges per the customer's plan, and creates an invoice in Stripe.

## Pricing Tiers

```csv
plan,monthly_base,included_requests,overage_per_1k,data_included_gb,overage_per_gb
starter,49,100000,0.50,10,0.10
growth,199,1000000,0.30,100,0.07
enterprise,custom,custom,custom,custom,custom
```

## Known Issues

- Proration logic for mid-cycle plan changes is fragile — [[customers/cascade-analytics.md|Cascade Analytics]]' annual migration surfaced several bugs
- No automated reconciliation between metering aggregates and Stripe invoices
- Jordan is the only person who can debug billing issues end-to-end
