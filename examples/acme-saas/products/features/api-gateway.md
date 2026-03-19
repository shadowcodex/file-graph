---
name: API Gateway
description: Central API gateway handling routing, rate limiting, and request authentication for all public endpoints
explicit_edges:
  "teams/platform.md": "Owned by the Platform team"
  "products/features/data-pipeline.md": "Routes ingestion requests to the pipeline"
  "products/features/billing.md": "Feeds request counts to billing for usage-based pricing"
  "runbooks/pipeline-backpressure.md": "Gateway rate limiting is first line of defense against pipeline overload"
---

# API Gateway

The API gateway is the front door to Acme's #infrastructure — every customer API request hits this service first. It handles #routing, #rate-limiting, authentication token validation, and request logging before proxying to the appropriate backend service.

Built on Envoy with a custom Lua filter layer for the auth checks. It feeds request data to two downstream systems: the [[products/features/data-pipeline.md|data pipeline]] for ingestion payloads, and the [[products/features/billing.md|billing system]] for usage-based pricing metering.

## Architecture

```yml
runtime: Envoy proxy
auth_filter: custom Lua (validates JWT from auth-service)
rate_limiting: sliding window, per-customer, configurable via dashboard
logging: structured JSON to Datadog
repo: acme/api-gateway
deploy: Kubernetes, us-east-1 and eu-west-1
```

## Known Concerns

The Lua filter layer is getting complex. Sarah has an RFC in draft to replace it with a Go sidecar but it keeps getting deprioritized behind the [[projects/auth-migration.md|auth migration]].
