---
name: Customer Dashboard
description: React-based web application where customers manage their account, view data, and configure integrations
explicit_edges:
  "teams/product.md": "Owned by the Product team"
  "people/alex-kim.md": "Primary architect and owner"
  "products/features/billing.md": "Billing management UI lives in the dashboard"
  "products/features/api-gateway.md": "Dashboard calls backend through the gateway"
  "projects/usage-analytics.md": "New analytics tab being added"
---

# Customer Dashboard

The dashboard is the main #frontend application — it's where customers log in to manage their Acme account, view their ingested data, configure integrations, and handle #billing. Built as a React SPA with a Next.js backend-for-frontend layer.

[[people/alex-kim.md|Alex]] owns the architecture and is protective of the component library. The codebase is in decent shape — good test coverage on the critical paths (auth, billing, data views) but the settings pages are a mess from too many quick feature additions.

## Tech Stack

```yml
frontend: React 18 + TypeScript
framework: Next.js 14 (app router)
state: Zustand
styling: Tailwind CSS
testing: Vitest + Playwright
repo: acme/dashboard
deploy: Vercel
```

## Current Work

The [[projects/usage-analytics.md|usage analytics]] feature is adding a new tab to the dashboard — charts showing API call volume, data ingestion throughput, and cost projections. This is the most customer-requested feature on the roadmap.
