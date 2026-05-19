# Business Context

Numbers Eva (CTO) gave during kickoff. Use these to translate technical findings into financial impact.

## Conversion economics

| Metric | Value |
|--------|-------|
| Average order value (AOV) | EUR 47.20 |
| Pre-Q1 conversion rate | 3.4% |
| Post-Q1 conversion rate | 3.1% |
| Visitor sessions / month (mobile + desktop) | ~6.5M |
| Annual revenue (FY 2025) | EUR 78.4M |
| Operating margin | 11% |

## Calculated impact of the regression

(Math the report should show.)

- Lost orders / month from regression: 6,500,000 * (0.034 - 0.031) = **19,500 orders / month**
- Lost revenue / month: 19,500 * EUR 47.20 = **EUR 920,400 / month**
- Lost annualised revenue: **~EUR 11M / year** if the regression persists
- Lost margin: ~EUR 1.2M / year

> Eva's number to remember: every 1% of conversion is worth ~EUR 4M / year at current AOV.

## Black Friday context

| Metric | Black Friday 2025 |
|--------|-------------------|
| Peak concurrent sessions | 92,000 |
| Peak RPS to tulipa-storefront | 14,200 |
| Peak RPS to checkout flow | 3,800 |
| Multiplier vs daily peak | ~7x |
| Revenue (single day) | EUR 4.1M |

> Black Friday 2026 is 2026-11-27. The platform must handle at least the same peak. Eva is asking whether the post-Q1 codebase can sustain that.

## Customer experience SLAs (informal, not contractual)

| Metric | Internal target | Current state |
|--------|-----------------|---------------|
| Add to cart | <300 ms perceived | OK (storefront) |
| Checkout page load | <2 s | FAILING (3.8s P75) |
| Place order success | >99% | FAILING (99.58%, was 99.92%) |
| Payment confirmation | <3 s | FAILING (P95 6.2s) |

## Sensitivities

- Eva does not want to be told "the codebase has bad architecture". She has a 3-year strategic plan that includes the RSC migration. She wants tactical fixes for now + a clear remediation roadmap.
- Lukas (e-commerce) wants conversion back to 3.4% by Black Friday. He is willing to roll back parts of the Q1 release if necessary.
- Yara (frontend) takes Q1 personally. Findings about frontend code need to be evidence-led, not blame-led.
- Daan (platform) is overstretched. The handover must be self-serve - the platform team cannot become consultants on their own platform.
