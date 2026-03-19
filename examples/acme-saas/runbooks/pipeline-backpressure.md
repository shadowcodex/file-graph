---
name: Pipeline Backpressure Runbook
description: On-call runbook for handling data pipeline backpressure and consumer lag spikes
explicit_edges:
  "people/marcus-jones.md": "Authored this runbook after the Feb 2025 outage"
  "products/features/data-pipeline.md": "This runbook is for operating the pipeline"
  "products/features/api-gateway.md": "Gateway circuit breaker is the first mitigation step"
  "incidents/2025-02-14-pipeline-outage.md": "Written as a direct result of this incident"
---

# Pipeline Backpressure Runbook

This #runbook covers what to do when the [[products/features/data-pipeline.md|data pipeline]] is experiencing backpressure — consumer lag is growing and ingestion latency is degraded. Written by [[people/marcus-jones.md|Marcus]] after the [[incidents/2025-02-14-pipeline-outage.md|Valentine's Day outage]] so the next #on-call engineer doesn't have to debug from scratch.

## When to Use This

You're getting paged for `pipeline.consumer_lag.critical` or you see consumer lag growing steadily in the Datadog dashboard at `dd/acme-pipeline-health`.

## Step 1: Assess Severity

Check the Datadog dashboard. If consumer lag is:
- **Growing slowly (<1M messages/min increase):** You have time. Move to Step 2.
- **Growing rapidly (>1M messages/min increase):** Skip to Step 3 and engage the circuit breaker immediately.
- **Consumers are in a rebalance loop:** This is the Valentine's Day scenario. Skip to Step 4.

## Step 2: Identify the Source

```sh
# Check per-customer topic lag to find who's spiking
kafka-consumer-groups --bootstrap-server $KAFKA_BROKER --describe --group pipeline-consumers | sort -k5 -nr | head -20
```

If one customer is dominating, check whether they're doing a bulk ingestion or if something is wrong on their end. Check their recent API key activity in the gateway logs.

## Step 3: Engage Circuit Breaker

The [[products/features/api-gateway.md|API gateway]] has a per-topic circuit breaker. To throttle a specific customer's ingestion:

```sh
# Enable rate limiting on a specific customer topic
kubectl exec -it gateway-admin -- acme-gateway rate-limit set --customer-id $CUSTOMER_ID --max-rps 1000
```

This will return 429s to the customer for requests above the limit. Communicate with CS before doing this for enterprise customers.

## Step 4: Consumer Rebalance Loop

If consumers are stuck in a rebalance loop (you'll see repeated `ConsumerRebalanceListener` logs):

1. Scale consumers to 0: `kubectl scale deployment pipeline-consumers --replicas=0`
2. Wait 30 seconds for the consumer group to fully release
3. Scale back to normal: `kubectl scale deployment pipeline-consumers --replicas=6`
4. Monitor lag — it should start decreasing immediately

If the rebalance loop resumes, the trigger is likely still active (a customer pushing extreme volume). Go back to Step 3 and throttle first, then retry.

## Step 5: Recovery

Once lag is decreasing:
- Monitor until lag returns to normal (<100K messages across all topics)
- If you engaged the circuit breaker, gradually increase the rate limit back to normal
- Post in #incidents with a summary of what happened

## Escalation

If none of the above works, page Marcus (`@marcus.jones` on PagerDuty). If Marcus is unavailable, page Sarah.
