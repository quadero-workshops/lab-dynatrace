# Capacity Baseline - Black Friday Modelling

Inputs Daan (Platform) provided for the Black Friday readiness assessment.

## Traffic baseline (current)

| Tier | Metric | Daily avg | Daily peak | 7d P95 |
|------|--------|-----------|------------|--------|
| Edge | RPS to Cloudflare | 4,200 | 11,400 | 9,800 |
| Storefront | RPS to tulipa-storefront | 1,820 | 4,800 | 4,100 |
| Checkout flow | RPS to tulipa-checkout | 410 | 1,180 | 980 |
| Payment | TPS payment-api | 38 | 92 | 78 |
| Database | RDS connections | 142 | 198 | 168 |

## Black Friday 2025 actual

| Tier | Metric | BF peak | Multiplier vs daily peak |
|------|--------|---------|---------------------------|
| Edge | RPS to Cloudflare | 84,000 | 7.4x |
| Storefront | RPS to tulipa-storefront | 36,000 | 7.5x |
| Checkout flow | RPS to tulipa-checkout (then: legacy checkout in storefront) | (n/a, combined) | - |
| Payment | TPS payment-api | 640 | 7.0x |
| Database | RDS connections | (saturated 200 cap multiple times) | - |

> The "saturated 200 connection cap multiple times" line from Daan is important. They got through Black Friday 2025 partly because Daan was awake.

## Modelling target for Black Friday 2026

Assumption: traffic grows 15% YoY (sector average + Tulipa's own marketing plan).

| Tier | Required peak | Headroom needed |
|------|---------------|-----------------|
| Edge (Cloudflare) | ~97,000 RPS | Cloudflare scales infinitely - n/a |
| tulipa-storefront | ~41,000 RPS | Currently sized for ~12,000 RPS sustained. Gap. |
| tulipa-checkout (new) | ~8,500 RPS | Currently sized for ~2,500 RPS sustained. Larger gap. |
| payment-api | ~735 TPS | Currently sized for ~150 TPS sustained. Significant gap. |
| RDS PostgreSQL | >230 connections | **Hard ceiling at 200.** Will fail. |
| ElastiCache Redis | >2x current memory | Already at 81% avg, 89% P95. Will OOM. |
| Lambda fraud-check | ~1,300 invocations/sec peak | At 18% cold start rate today, peak will compound to >3s checkout delays. |

## Known scaling levers

| Lever | Effort | Risk |
|-------|--------|------|
| ECS task count: tulipa-storefront 12 -> 30 | Low | Low |
| ECS task count: tulipa-checkout 8 -> 24 | Low | Low (but cost) |
| ECS task count: payment-api 4 -> 12 | Low | Need to check downstream limits (iDEAL TPS!) |
| RDS instance size up r6g.xlarge -> r6g.4xlarge | Medium | One maintenance window |
| RDS read replica | Medium | Needs app changes |
| Redis shard split | Medium | Needs migration window |
| Lambda provisioned concurrency for fraud-check | Low | Cost |
| Connection pool resize payment-api 50 -> 200 | Low | Need to verify RDS can take it (it cannot - see above) |

## Hard constraints

- iDEAL gateway has a contractual TPS limit of 500. We are nowhere near it today, but at BF projected 735 TPS payment we may approach it for iDEAL specifically.
- RDS instance size changes need a 4-hour maintenance window. Last possible window before BF: 2026-11-01.
- Budget envelope for capacity uplift: EUR ~80K one-off + EUR ~40K/year ongoing.
