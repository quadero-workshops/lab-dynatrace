# Error Snapshot - Tulipa Webshop

Pretend export from Dynatrace error analytics for the 7 days before kickoff.

## Top errors by impact

| Rank | Error | Service | Count (7d) | Sessions affected | First seen |
|------|-------|---------|------------|-------------------|------------|
| 1 | `TypeError: Cannot read properties of undefined (reading 'price')` | tulipa-checkout (browser) | 18,920 | 11,200 | 2026-03-18 |
| 2 | `Hydration failed because the initial UI does not match the server` | tulipa-checkout (browser) | 12,440 | 8,840 | 2026-03-18 |
| 3 | `502 Bad Gateway` from `iDEAL gateway` | payment-api -> ABN AMRO | 4,820 | 2,140 | 2026-03-22 |
| 4 | `NetworkError: payment-api/v1/intent timeout` | tulipa-checkout (browser) | 4,210 | 2,840 | 2026-03-20 |
| 5 | `ConnectionPoolTimeoutException: Timeout waiting for connection from pool` | payment-api | 2,180 | n/a (server) | 2026-04-01 |
| 6 | `Maximum update depth exceeded` | tulipa-checkout (browser, iOS Safari only) | 2,180 | 1,210 | 2026-03-18 |
| 7 | `Unhandled promise rejection: AbortError` | tulipa-checkout (browser) | 1,610 | 1,140 | 2026-04-02 |
| 8 | `Lambda init timeout (fraud-check)` | fraud-check | 412 | n/a (async) | (pre-Q1) |
| 9 | `Redis OOM` | cart-api -> ElastiCache | 184 | ~600 (carts lost) | 2026-04-12 |
| 10 | `Stripe 429 rate limit` | payment-api -> Stripe | 92 | 92 | 2026-04-15 |

## Error onset clusters

- **2026-03-18** (Q1 release day): errors #1, #2, #6. Frontend-side, all in `tulipa-checkout`. Cause clearly linked to the release.
- **2026-03-20 to 2026-03-22**: errors #3, #4. Payment-related, slightly delayed - either the new checkout pattern increased load on iDEAL gateway or the iDEAL gateway itself degraded.
- **2026-04-01**: error #5 (payment-api connection pool). Three weeks after release - likely a slow-burn effect of changed traffic pattern.
- **2026-04-12**: error #9 (Redis OOM). Pattern of rising evictions tipping into actual OOM events.

## Error #1 sample stack (browser console)

```
TypeError: Cannot read properties of undefined (reading 'price')
    at AddressStep (tulipa-checkout.js:14:8421)
    at processChild (react-dom-server.browser.production.min.js:1:42)
    at renderRootSync (react-dom.production.min.js:32:88)
```

Captured user actions before the error: viewed cart, clicked "Continue to address", filled address form, clicked "Continue to payment". Error fires during render of the payment step.

## Error #5 sample (payment-api logs via Grail)

```
2026-04-12T14:22:18.241Z [ERROR] [payment-api] ConnectionPoolTimeoutException:
Timeout waiting for connection from pool after 30000ms.
Active: 50, Max: 50, Available: 0, Pending: 12.
   at com.zaxxer.hikari.HikariDataSource.getConnection(HikariDataSource.java:189)
   at nl.tulipa.payment.intent.IntentService.createIntent(IntentService.java:78)
```

Pool size 50, max 50, 12 requests pending. Pool exhausted.

## What is NOT in this snapshot

- No errors from tulipa-storefront (legacy) above 0.12% rate.
- No errors from product-api or cart-api above 0.02%.
- No infra-level errors (host crashes, ECS task failures) - all green.

The error story is concentrated in **frontend (tulipa-checkout) + payment-api + the iDEAL dependency**. Everything else is background noise.
