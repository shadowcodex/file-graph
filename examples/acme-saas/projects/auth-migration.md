---
name: OAuth2 Migration
description: Migrating all customers from legacy session auth to OAuth2/OIDC, unlocking SSO for enterprise deals
explicit_edges:
  "teams/platform.md": "Platform team is leading the migration"
  "people/sarah-chen.md": "Wrote the RFC and is driving the project"
  "people/priya-sharma.md": "Implementing SSO and IdP integrations"
  "customers/northwind-logistics.md": "SSO is a hard blocker for this enterprise deal"
  "products/features/api-gateway.md": "Gateway auth filter needs to support JWT validation for the new flow"
---

# OAuth2 Migration

The auth migration is the highest-priority #project at Acme right now — moving every customer off the legacy cookie-based session auth to OAuth2/OIDC. This is a #security and #compliance effort at its core, but the real driver is business: multiple #enterprise deals (especially [[customers/northwind-logistics.md|Northwind Logistics]]) are blocked until Acme supports SSO.

[[people/sarah-chen.md|Sarah]] wrote the original RFC and is driving the project. [[people/priya-sharma.md|Priya]] is doing the bulk of the implementation, particularly the SSO/SAML integration work.

## Scope

The migration touches three systems:
1. **auth-service** — new OAuth2 provider with OIDC support, SAML 2.0 SP for enterprise SSO, SCIM endpoint for user provisioning
2. **[[products/features/api-gateway.md|API gateway]]** — Lua auth filter needs to validate JWTs instead of session cookies
3. **[[products/features/dashboard.md|Dashboard]]** — login flow needs to support SSO redirect, token refresh, and multi-org switching

## Timeline

```csv
milestone,target_date,status
RFC approved,2025-01-15,done
auth-service OAuth2 flow,2025-02-28,done
SAML 2.0 SP implementation,2025-03-31,in progress
gateway JWT validation,2025-04-15,not started
dashboard SSO login flow,2025-04-30,not started
legacy session deprecation,2025-05-31,not started
full cutover,2025-06-15,not started
```

## Risks

- Gateway Lua filter changes are risky — any bug takes down all API traffic. Sarah wants to do the Go sidecar rewrite first but there's no time.
- Dashboard SSO flow depends on the [[teams/product.md|Product team]], who are also deep in the [[projects/usage-analytics.md|usage analytics]] build. Bandwidth conflict.
- If this slips past June, the Northwind deal is at risk and sales will escalate to the CEO.
