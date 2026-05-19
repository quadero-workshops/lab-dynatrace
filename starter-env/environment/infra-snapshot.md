# Infrastructure Snapshot - Tulipa Webshop

Pretend Dynatrace host + container metrics for the 7 days before kickoff.

## ECS hosts (Fargate)

| Service | Tasks | vCPU/task | RAM/task | CPU P95 | RAM P95 |
|---------|-------|-----------|----------|---------|---------|
| tulipa-storefront | 12 | 2 | 4 GB | 58% | 64% |
| tulipa-checkout | 8 | 2 | 4 GB | **89%** | **91%** |
| product-api | 6 | 1 | 2 GB | 41% | 52% |
| cart-api | 6 | 1 | 2 GB | 38% | 48% |
| payment-api | 4 | 2 | 4 GB | 72% | 81% |

> tulipa-checkout tasks are running hot. Either undersized or leaking. Pre-Q1 these tasks did not exist; service launched 2026-03-18 at 6 tasks, scaled to 8 on 2026-04-15 (no measurable improvement). Sizing was inherited from storefront without re-evaluation.

## RDS PostgreSQL (db.r6g.xlarge)

| Metric | 7d avg | 7d P95 | Peak |
|--------|--------|--------|------|
| CPU | 38% | 61% | 78% |
| Memory | 71% | 82% | 88% |
| Read IOPS | 4,200 | 7,800 | 11,200 |
| Write IOPS | 1,800 | 3,200 | 4,800 |
| Connections | 142 / 200 max | 168 / 200 | **198 / 200** |
| Slow query log entries | 4,210/day avg | - | - |

> Connection ceiling hit 198/200 during peak. Connection pool exhaustion is a real risk.

## ElastiCache Redis (cache.r6g.large)

| Metric | 7d avg | 7d P95 | Notes |
|--------|--------|--------|-------|
| CPU | 22% | 38% | Healthy |
| Memory | 81% | 89% | High |
| Evictions/sec | 14 | 42 | **Rising trend (+220% vs baseline)** |
| Cache hit rate | 96.4% | - | Was 99.1% pre-Q1 |

> Eviction rate up 220%, cache hit rate dropped from 99.1% to 96.4%. Either cache is undersized for the new checkout workload or TTL is too short.

## SQS - fraud-check queue

| Metric | 7d avg | 7d max |
|--------|--------|--------|
| Messages in queue | 12 | 1,840 |
| Age of oldest message | 0.3s | 38s |
| DLQ messages | 0 | 4 (last 30d total) |

## Lambda - fraud-check

| Metric | Value |
|--------|-------|
| Invocations/day | 187K |
| Provisioned concurrency | 0 |
| Cold start % | 18% |
| P99 duration | 4,820 ms |
| Errors / day | 12 |

## Cloudflare CDN

| Metric | Value |
|--------|-------|
| Cache hit ratio (static) | 98.2% |
| Cache hit ratio (HTML) | 0% (uncacheable - signed-in users) |
| Edge response time P95 | 32 ms |
| Origin response time P95 | (see service tables above) |

## Quarter-over-quarter changes

- Compute: tulipa-checkout introduced Q1 with 6 tasks at 2vCPU/4GB; scaled to 8 tasks on 2026-04-15 in response to incident pressure (no measurable improvement, ticket INC-2952 retro).
- RDS: no instance change; query volume up ~12% post-Q1.
- Redis: no shard change; new checkout writes more cart fragments.
- Lambda: no provisioned concurrency added despite +40% invocation growth post-Q1.
