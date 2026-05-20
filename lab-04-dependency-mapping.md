# Lab 04 - Dependency Mapping

**Time:** 25-45 minutes
**Skill:** `/map-dependencies`
**Agent:** dependency-mapper
**Workflow:** dependency-health-check
**Goal:** Identify bottlenecks, single points of failure, and the cascade path of an iDEAL outage.

---

## Prerequisites

This lab can run after Lab 01 alone (no Lab 02/03 deliverables strictly required), but cross-references to prior analyses sharpen the SPOF discussion:
```bash
ls /workshop/state/engagement/engagement-plan.md 2>/dev/null || echo "Lab 01 not done - run it first"
```

## Setup

Input: `starter-env/environment/dependency-map.md` and `starter-env/environment/application-topology.md`.

```bash
cd /workshop/
claude
```

---

## Steps

### 1. Build the dependency graph from the two inputs

```
/map-dependencies

Synthesise /workshop/labs/lab-dynatrace/starter-env/environment/application-topology.md and /workshop/labs/lab-dynatrace/starter-env/environment/dependency-map.md into a single normalised dependency model.

Output /workshop/state/dependencies/graph.md with:

1. ASCII graph or Mermaid diagram showing all services + datastores + third parties
2. Per-edge annotations: calls/day, avg duration, error %
3. A 'Edge health' table sorted by error % descending

Use the dependency-mapper agent.
```

### 2. Identify SPOFs

```
List every Single Point of Failure in /workshop/state/dependencies/spofs.md. For each:

- the SPOF (service, datastore, or third party)
- which user actions become impossible if it fails
- blast radius (which downstream services degrade)
- existing mitigation (if any)
- proposed mitigation, with effort estimate

Pull candidates from dependency-map.md's SPOF table but verify each by tracing the dependency graph.
```

### 3. Trace the worst case "Place order" path

```
Read the 'Fan-out per user action (worst case)' section of dependency-map.md. Reproduce the trace in /workshop/state/dependencies/place-order-trace.md as a timeline:

| Step | Service | Operation | Duration P95 | Cumulative P95 |

The cumulative number is the customer-facing wait time. If it exceeds 5 seconds, flag it. Connect this directly to the funnel drop-off finding from Lab 02.
```

### 4. iDEAL cascade analysis

```
Trace what happens when the ABN AMRO iDEAL API returns 502 (the post-Q1 pattern observed in error-snapshot.md):

1. Which service handles the 502 first
2. Whether retries amplify or absorb the failure
3. How the customer experiences it (do they see an error, a hang, a retry loop?)
4. How it propagates to the cart-api / fraud-check / Redis

Save to /workshop/state/dependencies/ideal-cascade.md. This feeds spine question #2 ("Why does iDEAL error rate appear linked to a frontend refactor?").
```

### 5. DQL the trace

```
/schrijf-dql

Write the DQL query that reproduces the worst-case "Place order" trace from spans:
- fetch spans
- filter on the dt.entity.service or service.name for the path involved
- show trace_id, parent_span_id, service, duration
- order by trace_id, start_time

Save to /workshop/state/dependencies/queries.dql. Add a second query for "top 10 longest traces in the last 24h" with the same fields.
```

---

## What to Notice

| Behaviour | Why it matters |
|-----------|----------------|
| dependency-mapper distinguishes "data dependency" from "synchronous call" | Fraud-check via SQS is async; treating it as sync misleads the cascade analysis |
| SPOF analysis includes datastores, not just services | RDS at 200-conn cap is a SPOF even though it's "managed" |
| The 11-second worst-case trace is concrete | Abstract latency claims lose to concrete trace evidence |

---

## Expected output

`/workshop/state/dependencies/`:

- `graph.md` - normalised graph + edge health
- `spofs.md` - per-SPOF with blast radius and mitigation
- `place-order-trace.md` - cumulative timeline
- `ideal-cascade.md` - failure propagation story
- `queries.dql`

Key SPOFs you should land on:
- payment-api (4 tasks, connection pool 50)
- ABN AMRO iDEAL (external, no fallback)
- RDS PostgreSQL (200 connection hard cap)
- ElastiCache Redis (single shard, no replica)
- fraud-check Lambda cold start tail

---

## Success Criteria

- Dependency graph is complete (every service + datastore + third party from topology.md)
- At least 4 SPOFs identified with blast radius
- Worst-case trace shows >5s cumulative
- iDEAL cascade explains why a "frontend refactor" can correlate with "iDEAL 502s"

---

**Next:** [Lab 05 - Anomaly Detection](lab-05-anomaly-detection.md)
