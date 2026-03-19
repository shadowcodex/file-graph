---
name: Marcus Jones
description: Senior engineer on Platform, owns the data pipeline, built most of the ingestion layer
explicit_edges:
  "teams/platform.md": "Senior engineer since 2022"
  "products/features/data-pipeline.md": "Built and maintains the core ingestion pipeline"
  "incidents/2025-02-14-pipeline-outage.md": "Diagnosed the root cause and wrote the fix"
  "runbooks/pipeline-backpressure.md": "Authored this runbook after the Feb outage"
---

# Marcus Jones

Marcus is a #senior-engineer on the [[teams/platform.md|Platform team]] and the person who knows the [[products/features/data-pipeline.md|data pipeline]] better than anyone — he built most of the original ingestion layer when he joined in 2022. Before Acme he was at Datadog working on their metrics pipeline, so he thinks about everything in terms of #observability and throughput.

He diagnosed the root cause of the [[incidents/2025-02-14-pipeline-outage.md|Valentine's Day outage]] (Kafka consumer group rebalancing under load) and wrote the [[runbooks/pipeline-backpressure.md|backpressure runbook]] afterward so the next on-call wouldn't have to figure it out from scratch.

Quiet in meetings, extremely thorough in code review. If Marcus approves your PR without comments, you should be suspicious.

## Contact

```csv
channel,handle
email,marcus.jones@acme.io
slack,@marcus.jones
github,@mjones-acme
```
