# Service Dependency Map - Tulipa Webshop

Pretend export from Dynatrace `fetch spans | summarize by peer.service` for the 7 days before kickoff.

## Outbound calls per service

### tulipa-checkout (8 ECS tasks)

| Peer service | Calls/day | Avg duration | Error % |
|--------------|-----------|--------------|---------|
| cart-api | 1.2M | 42 ms | 0.02% |
| product-api | 980K | 28 ms | 0.01% |
| payment-api | 410K | 1,840 ms | 0.42% |
| fraud-check (via SQS) | 187K | (async) | n/a |
| Algolia (search) | 88K | 110 ms | 0.04% |

### tulipa-storefront (12 ECS tasks)

| Peer service | Calls/day | Avg duration | Error % |
|--------------|-----------|--------------|---------|
| product-api | 6.8M | 22 ms | 0.01% |
| cart-api | 2.1M | 18 ms | 0.01% |
| Algolia (search) | 412K | 90 ms | 0.02% |
| Cloudflare images | (CDN, not traced) | - | - |

### payment-api (4 ECS tasks)

| Peer service | Calls/day | Avg duration | Error % |
|--------------|-----------|--------------|---------|
| Stripe API | 138K | 410 ms | 0.08% |
| ABN AMRO iDEAL API | 89K | 1,920 ms | **2.4%** |
| RDS PostgreSQL | 612K | 18 ms | 0.01% |
| Redis (cache) | 980K | 4 ms | 0.01% |

> ABN AMRO iDEAL: 2.4% error rate (mostly 502s). 1.92s average is also slow vs Stripe's 410ms.

### cart-api (6 ECS tasks)

| Peer service | Calls/day | Avg duration | Error % |
|--------------|-----------|--------------|---------|
| Redis (cache) | 4.8M | 3 ms | 0.01% |
| RDS PostgreSQL | 1.4M | 24 ms | 0.01% |

### product-api (6 ECS tasks)

| Peer service | Calls/day | Avg duration | Error % |
|--------------|-----------|--------------|---------|
| RDS PostgreSQL | 7.2M | 14 ms | 0.01% |
| Redis (cache) | 6.1M | 2 ms | 0.01% |

### fraud-check (Lambda)

| Peer service | Calls/day | Avg duration | Error % |
|--------------|-----------|--------------|---------|
| External fraud scoring SaaS | 187K | 320 ms | 0.12% |
| RDS PostgreSQL | 187K | 22 ms | 0.01% |

## Fan-out per user action (worst case)

A single "Place order" click triggers, in order:

1. tulipa-checkout receives POST /checkout/place-order
2. cart-api GET /v1/cart/{id} (lock the cart)
3. product-api GET /v1/product/{id} x N items (validate stock)
4. payment-api POST /v1/intent (create intent)
5. payment-api -> ABN AMRO iDEAL API (if iDEAL chosen)
6. fraud-check SQS message published (async)
7. payment-api POST /v1/confirm (after iDEAL redirect)
8. cart-api PUT /v1/cart/{id} (mark order)

Worst-case timeline observed (P99 trace): 11.2 seconds end-to-end.

## Single Points of Failure (SPOFs) identified

| SPOF | Impact | Notes |
|------|--------|-------|
| payment-api (4 tasks) | Total checkout outage if it falls over | Connection pool sized 50, may saturate at peak |
| ABN AMRO iDEAL API | NL iDEAL payments (~60% of NL orders) | External, no fallback configured |
| RDS PostgreSQL (single instance) | All catalog + order writes | Multi-AZ enabled, no read replica |
| ElastiCache Redis (single shard) | Cart loss on failure | No replica configured |
| fraud-check Lambda (no provisioned concurrency) | Order placement delays during cold start | Async, but 18% cold starts is high |

## Service health summary

| Service | Status | Reason |
|---------|--------|--------|
| tulipa-storefront | green | Baseline, no regression |
| tulipa-checkout | red | Web Vitals + response times regressed |
| product-api | green | Healthy |
| cart-api | green | Healthy |
| payment-api | amber | iDEAL dependency flaking, P95 +38% |
| fraud-check | amber | High cold start rate |
