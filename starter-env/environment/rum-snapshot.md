# RUM Snapshot - Tulipa Webshop

Pretend export from the Dynatrace `dt.rum.user_action` table for the 7 days before kickoff (2026-04-27 to 2026-05-03). Compared against a pre-Q1 baseline week (2026-02-23 to 2026-03-01), about 9 weeks before the analysis window - chosen to predate the 2026-03-18 Q1 release rather than the standard 4-week comparison, which would otherwise fall after the regression.

## Overall Web Vitals (P75)

| Metric | Last 7d | Baseline (pre-Q1) | Threshold |
|--------|---------|-------------------|-----------|
| LCP | 3,810 ms | 2,210 ms | <2,500 ms good, >4,000 poor |
| INP | 412 ms | 178 ms | <200 ms good, >500 poor |
| CLS | 0.18 | 0.06 | <0.1 good, >0.25 poor |
| FCP | 1,950 ms | 1,310 ms | <1,800 ms good |
| TTFB | 690 ms | 410 ms | <800 ms good |

All Web Vitals regressed post-Q1. LCP and INP crossed from "good/needs-improvement" into "needs-improvement/poor". CLS doubled.

## By application

| Application | Sessions (7d) | LCP P75 | INP P75 | CLS P75 |
|-------------|---------------|---------|---------|---------|
| tulipa-storefront (legacy) | 1,420,000 | 2,400 ms | 195 ms | 0.07 |
| tulipa-checkout (new) | 290,000 | 5,200 ms | 680 ms | 0.31 |

> The regression is concentrated in the new `tulipa-checkout` application. The legacy storefront is roughly at baseline.

## By device class

| Device | Sessions | LCP P75 (checkout) | INP P75 (checkout) |
|--------|----------|---------------------|---------------------|
| Desktop | 102,000 | 3,100 ms | 240 ms |
| Mobile - Android | 121,000 | 5,800 ms | 720 ms |
| Mobile - iOS Safari | 67,000 | 6,400 ms | 980 ms |

> iOS Safari is the worst-affected device class. INP nearly 1 second.

## By geography

| Country | Sessions | LCP P75 | Notes |
|---------|----------|---------|-------|
| NL | 1,505,000 | 3,700 ms | Primary market |
| BE | 180,000 | 4,100 ms | |
| DE | 25,000 | 5,200 ms | Cloudflare edge selection suspected |

## JavaScript errors (last 7d, top 5)

| Error | Count | First seen | Browsers |
|-------|-------|------------|----------|
| `TypeError: Cannot read properties of undefined (reading 'price')` | 18,920 | 2026-03-18 (Q1 release week) | Safari, Chrome |
| `Hydration failed because the initial UI does not match...` | 12,440 | 2026-03-18 | Mobile (all) |
| `NetworkError: payment-api/v1/intent timeout` | 4,210 | 2026-03-20 | All |
| `Maximum update depth exceeded` | 2,180 | 2026-03-18 | iOS Safari only |
| `Unhandled promise rejection: AbortError` | 1,610 | 2026-04-02 | All |

## User actions with worst INP (last 7d)

| Page | User action | INP P75 | Sessions |
|------|-------------|---------|----------|
| /checkout/address | "Continue to payment" button click | 1,240 ms | 38,000 |
| /checkout/payment | iDEAL bank selector dropdown | 980 ms | 22,000 |
| /checkout/review | "Place order" button click | 740 ms | 41,000 |
| /product/[slug] | "Add to cart" button click | 520 ms | 192,000 |

## Funnel impact

| Funnel step | Pre-Q1 dropoff | Post-Q1 dropoff | Delta |
|-------------|----------------|------------------|-------|
| Cart -> Address | 12% | 14% | +2pp |
| Address -> Payment | 8% | 19% | +11pp |
| Payment -> Review | 5% | 12% | +7pp |
| Review -> Order placed | 3% | 6% | +3pp |
| **Overall conversion** | **3.4%** | **3.1%** | **-0.3pp (-8.8% relative)** |

The biggest dropoff increase is **Address -> Payment**. This is the same step where INP P75 is worst (1,240 ms on the "Continue" button).
