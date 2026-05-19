# Incident Timeline

Compiled by the consultant from PagerDuty, GitLab release notes, and Slack archaeology. Used as the canonical narrative against which Dynatrace findings will be correlated.

## Q1 2026 - the checkout redesign

| Date | Event | Source |
|------|-------|--------|
| 2026-02-12 | Q1 OKR approved: "Migrate checkout to Next.js 15 RSC for better SEO + Web Vitals" | Confluence |
| 2026-02-26 | Feature branch merged behind feature flag, 5% canary in production | GitLab MR !2418 |
| 2026-03-11 | Canary at 25% - "all metrics green" per Yara in #release-checkout | Slack |
| 2026-03-18 | **Full rollout to 100%**. Old `/checkout` route deprecated. Marketing announces "new and improved checkout" | GitLab tag v2.0.0 |
| 2026-03-19 | First customer complaint to support: "checkout is broken on my iPhone" | Zendesk #44182 |
| 2026-03-22 | iDEAL 502 errors start appearing in PagerDuty. Hypothesis at the time: "iDEAL having a bad day" | PagerDuty INC-2884 |
| 2026-04-01 | payment-api connection pool exhaustion (Hikari timeout). Hotfix: restart all payment-api tasks. No root cause filed | PagerDuty INC-2911 |
| 2026-04-08 | Yara raises in retro: "Hydration errors in Sentry have been climbing since release. Not sure if related to perf complaints" | Retro notes |
| 2026-04-12 | Redis OOM event. ~600 carts dropped. Hotfix: increase maxmemory-policy from allkeys-lru to volatile-lru. No deeper investigation | PagerDuty INC-2952 |
| 2026-04-23 | Lukas escalates to Eva: "Conversion has not recovered. Marketing wants to know why" | Email |
| 2026-04-30 | Quadero engaged | Email |

## Hypotheses already considered (and discarded) by Tulipa

| Hypothesis | Outcome |
|------------|---------|
| "iDEAL is just flaky" | Disproved when same error rate not seen by sector peers |
| "It is a Cloudflare regional issue" | Cloudflare confirmed no incidents in the window |
| "Mobile networks got worse" | Speedtest data shows no change |
| "We need more ECS capacity" | Daan scaled tulipa-checkout from 6 to 8 tasks on 2026-04-15. No measurable improvement |
| "It is the holiday/Easter dip" | Conversion did not recover after the holiday |

## What the engagement must explain

1. Why did `tulipa-checkout` Web Vitals regress while `tulipa-storefront` did not?
2. Why does the iDEAL error rate appear linked to a frontend refactor that did not touch the iDEAL integration?
3. Why is payment-api connection pool exhausting now and not before?
4. Why are Redis evictions up 220% post-Q1?
5. Are these one root cause or several?

These five questions are the spine of the final report.
