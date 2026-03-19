---
name: Northwind Logistics
description: Enterprise prospect in late-stage sales, SSO is a hard blocker for the deal
explicit_edges:
  "people/priya-sharma.md": "Weekly syncs on SSO requirements"
  "projects/auth-migration.md": "SSO support depends on this project completing"
---

# Northwind Logistics

Northwind is a #prospect in late-stage #enterprise sales — they're a national logistics company that wants to use Acme for supply chain event tracking. The deal is worth roughly $180K ARR but has a hard blocker: they require #sso (SAML 2.0 specifically) and Acme doesn't support it yet.

The [[projects/auth-migration.md|OAuth2 migration]] will unlock SSO, and [[people/priya-sharma.md|Priya]] has been doing weekly syncs with Northwind's IT team to make sure the implementation meets their specific requirements (SAML 2.0 with Okta as their IdP, SCIM provisioning for user lifecycle).

Sales has been telling them "Q2" for SSO availability. If the auth migration slips, this deal slips with it — and the sales team will not be quiet about it.

## Deal Details

```csv
field,value
projected_arr,$180000
stage,technical evaluation
blocker,SSO (SAML 2.0 + SCIM)
idp,Okta
primary_contact,Tom Huang (CTO)
sales_owner,Lisa Park
target_close,2025-06-30
```
