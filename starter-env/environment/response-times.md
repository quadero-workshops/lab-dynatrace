# Service Response Times - Tulipa Webshop

Pretend export from Dynatrace service metrics for the 7 days before kickoff. P50/P90/P95/P99 in milliseconds, request count over the period.

## tulipa-storefront (Next.js 14, legacy)

| Endpoint | Requests | P50 | P90 | P95 | P99 | Error % |
|----------|----------|-----|-----|-----|-----|---------|
| GET / | 1,890,000 | 110 | 230 | 320 | 580 | 0.08% |
| GET /product/[slug] | 4,210,000 | 95 | 180 | 240 | 410 | 0.12% |
| GET /category/[slug] | 980,000 | 130 | 250 | 360 | 620 | 0.05% |
| GET /api/cart | 2,310,000 | 22 | 48 | 71 | 140 | 0.03% |

## tulipa-checkout (Next.js 15 RSC, new)

| Endpoint | Requests | P50 | P90 | P95 | P99 | Error % |
|----------|----------|-----|-----|-----|-----|---------|
| GET /checkout | 312,000 | 380 | 1,210 | 1,840 | 3,420 | 0.04% |
| GET /checkout/address | 298,000 | 410 | 1,380 | 2,010 | 3,790 | 0.06% |
| GET /checkout/payment | 241,000 | 920 | 2,810 | 4,210 | 7,840 | 0.21% |
| GET /checkout/review | 218,000 | 280 | 880 | 1,310 | 2,420 | 0.05% |
| POST /checkout/place-order | 187,000 | 1,640 | 3,920 | 5,810 | 11,240 | 0.42% |

> All five checkout endpoints are 3-5x slower than the storefront. /checkout/payment and POST /place-order are the worst offenders.

## product-api

| Endpoint | Requests | P50 | P90 | P95 | P99 | Error % |
|----------|----------|-----|-----|-----|-----|---------|
| GET /v1/product/{id} | 4,840,000 | 18 | 42 | 64 | 110 | 0.01% |
| GET /v1/search | 1,920,000 | 88 | 240 | 380 | 920 | 0.02% |
| GET /v1/recommendations | 1,210,000 | 42 | 110 | 160 | 280 | 0.01% |

## cart-api

| Endpoint | Requests | P50 | P90 | P95 | P99 | Error % |
|----------|----------|-----|-----|-----|-----|---------|
| GET /v1/cart/{id} | 3,180,000 | 14 | 38 | 58 | 110 | 0.01% |
| PUT /v1/cart/{id}/items | 1,420,000 | 31 | 78 | 120 | 240 | 0.02% |
| DELETE /v1/cart/{id}/items/{sku} | 410,000 | 19 | 52 | 78 | 140 | 0.01% |

## payment-api

| Endpoint | Requests | P50 | P90 | P95 | P99 | Error % |
|----------|----------|-----|-----|-----|-----|---------|
| POST /v1/intent | 241,000 | 380 | 1,820 | 3,210 | 8,420 | 0.18% |
| POST /v1/confirm | 187,000 | 920 | 3,810 | 6,240 | 14,210 | 0.41% |
| GET /v1/status/{id} | 612,000 | 110 | 380 | 540 | 1,210 | 0.04% |

> payment-api `/v1/confirm` P99 is 14 seconds. That is the call behind the "Place order" button. Investigate.

## fraud-check (Lambda)

| Endpoint | Invocations | P50 | P90 | P95 | P99 | Cold start % |
|----------|-------------|-----|-----|-----|-----|--------------|
| (async via SQS) | 187,000 | 240 | 1,210 | 2,140 | 4,820 | 18% |

> 18% cold start rate is high for a service this busy. Investigate provisioned concurrency.

## Period comparison vs pre-Q1 baseline

| Service | P95 change |
|---------|------------|
| tulipa-storefront | +6% |
| tulipa-checkout | (did not exist pre-Q1) |
| product-api | +4% |
| cart-api | -2% |
| payment-api | **+38%** |
| fraud-check | +12% |

> payment-api regressed 38% on P95 despite no deploys to payment-api in Q1. Something changed in its environment.
