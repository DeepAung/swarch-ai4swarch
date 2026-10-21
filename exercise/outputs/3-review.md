Agent: Claude Code (omp), model anthropic/claude-sonnet-5-5

# Architecture review: ADR-0002 (RabbitMQ for post-order events)

Read: the ADR and `docs/scenario.md` only.

## 1. Quality-attribute scenarios

| # | Source · Stimulus · Environment | Response | Response measure |
|---|---|---|---|
| S1 Performance | Shopper · `PlaceOrder` · 11.11 peak, 200 orders/s, MailFast at p95 1.5 s / p99 8 s | Checkout does payment, shipping, cart, publish; no wait on consumers | p95 `PlaceOrder` ≤ 500 ms (today about 300 ms, so the publish has about 200 ms of headroom) |
| S2 Availability | Broker, `loyaltyservice` or MailFast goes down · normal and peak load | Order still succeeds; work is queued or retried | 0 orders failed because of email, loyalty or analytics; includes the case where RabbitMQ itself is down at publish time |
| S3 Correctness | Order placed · consumer crashes or message is redelivered · peak | Points are awarded once | Points visible ≤ 1 min after order, exactly once, even with 11.11 backlog on other queues |

## 2. Findings

| Type | Finding | Why it matters |
|---|---|---|
| Non-risk | Async publish removes MailFast (p95 1.5 s) from the request path | Directly meets S1. Options 1 and 2 are correctly rejected on latency and on loss at pod restart. |
| Non-risk | One durable queue per consumer; loyalty is idempotent on `order_id` | Isolates failures (S2). The email backlog does not delay loyalty. Redelivery is handled for S3. |
| Non-risk | RabbitMQ is open-source, runs on Kubernetes, and two of four developers know it | Fits the stated operations preference and the team's skills. |
| Risk | Event is lost if publish fails after payment. The ADR offers "confirms and a short retry" and drops the outbox because `checkoutservice` has no store | Payment has already succeeded, so a lost event means a paid order with no email and no points. This breaks S3 and the "exactly once" claim. The ADR also never says whether a failed publish fails the order. Failing it would break S2; swallowing it loses events. |
| Risk | Email backlog is unbounded. 200/s in, 50/s out is +150/s. A 30-minute peak is about 270k queued emails and a drain time of about 90 minutes | The "confirmation" email may arrive hours late. The ADR calls this an "open question" but accepts anyway. Queue memory and disk usage on the broker also grows without limit. |
| Risk | Email consumer is "one consumer with bounded prefetch" | A single consumer is a single point of failure for email, which contradicts the ADR's own availability argument. Sustaining 50/s at 1.5 s latency needs about 75 in-flight calls (Little's law), and more at p99 8 s. A small prefetch would cap throughput well below 50/s. |
| Risk | The event carries user email and order details to every consumer, including analytics | Personal data is spread across queues and the dead-letter queue. The ADR has no data-minimisation or access-control statement. |
| Sensitivity point | Prefetch and consumer concurrency on the email queue | These set the rate against MailFast's 50/s limit. Too high gets rate-limited, too low builds a backlog. |
| Sensitivity point | Publish latency and confirm timeout inside `PlaceOrder` | Waiting for a confirm adds to the 500 ms budget. A long retry eats the headroom. |
| Trade-off point | Eventual consistency against latency and availability | The shopper sees "order placed" before the email or points. The ADR accepts this, but the product has not agreed a maximum email delay. |
| Trade-off point | Adding a stateful broker against modifiability | New consumers need no change to `checkoutservice`. The cost is a new component, and the shared fate of all post-order actions if it fails. |

## 3. Missing or weak

- **Options not considered:**
  - A transactional outbox, or an outbox-style store, as a variant of option 3. It is dismissed in a risk bullet and never analysed.
  - A hybrid: call `loyaltyservice` over gRPC with a timeout, and use the broker only for email and analytics.
  - Other self-hosted brokers (e.g. Kafka, NATS JetStream) for comparison. Option 4 is dismissed on "no team experience given" and "cost not given", which is weak.
- **"Exactly once"** is claimed through idempotency. That gives effective once only if the event is published at all. The publish gap is not closed.
- **HA claim:** "clustered, durable, replicated queues" has no queue type, replica count or failure-domain detail. It is not backed by evidence.
- **Rate limit:** "50/s shared across replicas" is stated, but no mechanism is given beyond the single consumer.
- **Peak duration** is not in the scenario and not assumed. Without it the backlog and delay figures cannot be sized.
- **Dead-letter queue:** no owner, no replay procedure, no alert.
- **Status** is `Accepted` on the same day as drafting, while its own open question is unresolved.
- **Validation** is a good start, but it lacks a test of "publish fails after payment" and a test with a peak of realistic length.

## 4. Verdict

**Accept with changes.** The direction is sound for this scenario. The gaps are in the publish step and in email capacity.

1. **Close the publish gap.** State what `PlaceOrder` does when the publish fails (do not fail the order). Choose between an outbox and an accepted loss rate. Add a failure test for it.
2. **Size the email path.** Get a maximum email delay from the business. Compute prefetch and concurrency for 50/s at 1.5 s latency. Run at least two consumers with a shared limiter. Add a backlog alert.
3. **Spell out the HA and data design.** Name the queue type and replica count, define the DLQ owner and replay, and minimise the personal data in the event. Change the status to `Proposed` until these are done.
