# Application Topology - Tulipa Webshop

Snapshot taken from the Dynatrace Smartscape view during kickoff. Captured 2026-05-05.

## High-level layout

```
                                            +--------------------+
                                            |  Cloudflare CDN    |
                                            |  + WAF             |
                                            +---------+----------+
                                                      |
                                                      v
                                            +--------------------+
                                            |   ALB (AWS)        |
                                            +---------+----------+
                                                      |
                          +---------------------------+--------------------------+
                          |                                                      |
                +-------------------+                                 +-------------------+
                | tulipa-storefront |                                 | tulipa-checkout   |
                | (Next.js 14, ECS) |                                 | (Next.js 15 RSC,  |
                | 12 tasks          |                                 |  ECS, 8 tasks)    |
                +---------+---------+                                 +---------+---------+
                          |                                                      |
                          +--------------------+ +------------------+ +----------+
                                               | |                  | |
                                       +----------------+   +----------------+   +----------------+
                                       | product-api    |   | cart-api       |   | payment-api    |
                                       | (Node, ECS)    |   | (Node, ECS)    |   | (Java SB, ECS) |
                                       | 6 tasks        |   | 6 tasks        |   | 4 tasks        |
                                       +-------+--------+   +-------+--------+   +-------+--------+
                                               |                    |                    |
                                               v                    v                    v
                                       +----------------+   +----------------+   +----------------+
                                       | RDS PostgreSQL |   | ElastiCache    |   | Stripe + ABN   |
                                       | (catalog)      |   | (Redis)        |   | AMRO iDEAL     |
                                       +----------------+   +----------------+   +----------------+
                                                                                          ^
                                                                                          |
                                                                            +-------------+-------------+
                                                                            | fraud-check (Python)      |
                                                                            | (Lambda, async via SQS)   |
                                                                            +---------------------------+
```

## Services in Dynatrace

| Service | Tech | Tasks/Instances | Owner |
|---------|------|-----------------|-------|
| tulipa-storefront | Next.js 14 (legacy) | 12 ECS tasks | Yara (Frontend) |
| tulipa-checkout | Next.js 15 RSC (new Q1) | 8 ECS tasks | Yara (Frontend) |
| product-api | Node 20 + Express | 6 ECS tasks | Daan (Platform) |
| cart-api | Node 20 + Fastify | 6 ECS tasks | Daan (Platform) |
| payment-api | Java 21 / Spring Boot | 4 ECS tasks | Daan (Platform) |
| fraud-check | Python 3.11 / Lambda | on-demand | Daan (Platform) |

## Datastores

| Datastore | Purpose | Health |
|-----------|---------|--------|
| RDS PostgreSQL (db.r6g.xlarge) | Product catalog, orders | CPU avg 38%, peaks 78% |
| ElastiCache Redis (cache.r6g.large) | Cart sessions, hot product | Evictions rising last 14d |
| SQS - fraud-check queue | Async fraud verdicts | Backlog spikes during peaks |

## Third-party dependencies

| Dependency | Purpose | Health |
|------------|---------|--------|
| Stripe API | Card payments | Healthy |
| ABN AMRO iDEAL API | iDEAL payments (NL) | Sporadic 502s post-Q1 |
| Cloudflare CDN | Static assets, WAF | Healthy |
| Sendgrid | Order confirmation email | Healthy |
| Algolia | Product search | Healthy |

## Known smells (from Daan's intake)

- `tulipa-checkout` (new) and `tulipa-storefront` (legacy) share cart-api and product-api - any regression there hits both.
- payment-api connection pool sized for 50 concurrent connections; nobody has revisited this since 2024.
- fraud-check Lambda cold starts visible during off-peak hours.
- ElastiCache eviction rate rising - hot product cache may be undersized.
