---
name: Valentine's Day Pipeline Outage
description: "4-hour data pipeline outage on Feb 14, 2025 caused by Kafka consumer group rebalancing under load"
explicit_edges:
  "products/features/data-pipeline.md": "Primary system affected"
  "products/features/api-gateway.md": "Gateway failed to shed load, making the outage worse"
  "people/sarah-chen.md": "Incident commander"
  "people/marcus-jones.md": "Diagnosed root cause and implemented the fix"
  "customers/meridian-health.md": "Most impacted customer, lost 3 hours of ingestion data"
  "runbooks/pipeline-backpressure.md": "Written as a direct result of this incident"
---

# Valentine's Day Pipeline Outage

On February 14, 2025, the [[products/features/data-pipeline.md|data pipeline]] experienced a full #outage lasting approximately 4 hours (08:12 - 12:35 UTC). This was a #severity/s1 #incident that impacted all customers' data ingestion, with [[customers/meridian-health.md|Meridian Health]] being the hardest hit — they lost roughly 3 hours of patient intake data that had to be manually re-ingested.

## Timeline

```csv
time_utc,event
08:12,Datadog alerts fire for pipeline consumer lag spike
08:15,On-call (Priya) acknowledges and begins investigating
08:22,Sarah joins as incident commander
08:30,Consumer group enters repeated rebalancing loop — all ingestion stalled
08:45,Marcus joins and identifies Kafka consumer group rebalancing as root cause
09:15,First mitigation attempt: restart consumers — fails as rebalancing resumes
09:40,Marcus identifies trigger: Meridian Health pushed 10x normal volume starting at 08:00
10:00,Gateway rate limiting was not enforced on Meridian's legacy API key
10:30,Hotfix deployed: emergency rate limit on the overloaded topic + increased consumer count
11:15,Pipeline processing resumes but working through backlog
12:35,Backlog cleared — all systems nominal
```

## Root Cause

[[customers/meridian-health.md|Meridian Health]] started a bulk historical data re-ingestion at 08:00 UTC that pushed 10x their normal event volume. The [[products/features/api-gateway.md|API gateway]]'s rate limiting wasn't enforced on their legacy API key (predated the rate limiting feature), so the full volume hit Kafka directly. The sudden load spike triggered Kafka consumer group rebalancing, which cascaded into a rebalance loop that stalled all consumers across all customer topics.

## Action Items

- [x] Add rate limiting to all legacy API keys (Sarah — completed 2025-02-17)
- [x] Implement adaptive consumer scaling based on lag metrics (Marcus — completed 2025-02-28)
- [x] Write [[runbooks/pipeline-backpressure.md|backpressure runbook]] (Marcus — completed 2025-02-21)
- [x] Add circuit breaker at gateway for per-topic volume spikes (Sarah — completed 2025-03-05)
- [ ] Customer communication template for ingestion outages (Product — pending)
- [ ] Automated backfill tooling for missed ingestion windows (Marcus — Q2)
