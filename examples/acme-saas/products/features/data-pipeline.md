---
name: Data Pipeline
description: Kafka-based data ingestion pipeline that processes customer data uploads and event streams
explicit_edges:
  "teams/platform.md": "Owned by the Platform team"
  "people/marcus-jones.md": "Built and maintains the pipeline"
  "products/features/api-gateway.md": "Receives ingestion requests routed through the gateway"
  "incidents/2025-02-14-pipeline-outage.md": "Was the primary system affected in the Valentine's Day outage"
  "runbooks/pipeline-backpressure.md": "Operational runbook for handling backpressure"
---

# Data Pipeline

The data pipeline is the core #ingestion system at Acme — it's how customer data actually gets into the platform. Customers push data via the [[products/features/api-gateway.md|API gateway]] and the pipeline handles validation, transformation, deduplication, and storage. It processes roughly 2 billion events per day across all customers.

[[people/marcus-jones.md|Marcus]] built most of the original architecture and remains the #subject-matter-expert on it. The system is Kafka-based with a fleet of consumer services written in Go.

## Architecture

```yml
message_broker: Kafka (MSK, 12 partitions per topic)
consumers: Go services (3 consumer groups)
storage: S3 (raw) + Postgres (processed)
throughput: ~2B events/day
latency_p99: 340ms end-to-end
repo: acme/data-pipeline
deploy: Kubernetes, us-east-1
```

## Post-Outage Improvements

After the [[incidents/2025-02-14-pipeline-outage.md|Valentine's Day outage]], the team added adaptive consumer scaling and a circuit breaker at the gateway level. The [[runbooks/pipeline-backpressure.md|backpressure runbook]] now covers recovery procedures.
