---
name: Platform Team
description: Core infrastructure team responsible for APIs, auth, and data pipeline reliability
explicit_edges:
  "people/sarah-chen.md": "Tech lead, joined 2023"
  "people/marcus-jones.md": "Senior engineer, owns the data pipeline"
  "people/priya-sharma.md": "Engineer, focused on auth and identity"
  "products/features/api-gateway.md": "Primary owners of the API gateway"
  "products/features/data-pipeline.md": "Primary owners of the ingestion pipeline"
  "projects/auth-migration.md": "Currently leading the OAuth2 migration"
---

# Platform Team

The #platform team owns the core #infrastructure that the rest of Acme SaaS runs on — the API gateway, authentication layer, and data ingestion pipeline. They're a small team (3 engineers) but touch almost everything, so most #incidents end up in their queue at some point.

The team runs two-week sprints and does on-call rotation weekly. Their Slack channel is #team-platform and they do standups async in that channel every morning.

## Current Focus

Right now the team is heads-down on the [[projects/auth-migration.md|OAuth2 migration]] — moving all customers off the legacy session-based auth by end of Q2. This is blocking several enterprise deals that require SSO support.

## Services Owned

```csv
service,repo,oncall_schedule,status
api-gateway,acme/api-gateway,weekly rotation,stable
auth-service,acme/auth-service,weekly rotation,migrating
data-pipeline,acme/data-pipeline,weekly rotation,stable
```
