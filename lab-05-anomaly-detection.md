# Lab 05 - Anomaly Detection

**Time:** 25-45 minutes
**Skill:** `/detecteer-anomalieen`
**Agent:** anomalie-detector
**Workflow:** performance-review (continued)
**Goal:** Find silent degradation - patterns the dashboards missed because they crossed thresholds slowly.

---

## Prerequisites

Labs 02-04 deliverables should exist:
```bash
for d in rum response-times dependencies; do
  ls /workshop/state/$d/ >/dev/null 2>&1 || echo "MISSING /workshop/state/$d/ - run Lab 02-04 first"
done
```

## Setup

Inputs: `starter-env/environment/infra-snapshot.md`, `starter-env/environment/error-snapshot.md`, and the prior analyses from Labs 02-04.

```bash
cd /workshop/
claude
```

---

## Steps

### 1. Survey the slow-burn metrics

```
Using the anomalie-detector agent, list metrics from infra-snapshot.md that have been drifting but never broke a threshold loud enough to page anyone. For each, state the baseline, the current value, and the trend direction.

Candidates to consider:
- Redis evictions (was 0, now 42/sec P95)
- Redis cache hit rate (99.1% -> 96.4%)
- RDS connections (rising toward 200 ceiling)
- Lambda cold start rate (was <5% pre-Q1, now 18%)
- payment-api task CPU (was <50%, now 72% P95)
- tulipa-checkout task RAM (91% P95)

Save to /workshop/state/anomalies/silent-drift.md.
```

### 2. Run anomaly detection against errors

```
/detecteer-anomalieen

Analyse error-snapshot.md for patterns that emerge between Q1 release (2026-03-18) and now. Specifically:

1. Group errors by onset date - which cluster around release day vs which appear weeks later
2. For "weeks later" errors (#5 connection pool, #9 Redis OOM): trace which earlier anomaly (from silent-drift.md) probably caused them
3. Flag any error that is currently below a paging threshold but trending toward one

Save to /workshop/state/anomalies/error-pattern.md.
```

### 3. Build the silent-degradation timeline

```
Combine silent-drift.md and error-pattern.md into /workshop/state/anomalies/timeline.md. Format:

| Date | Metric/Event | Severity | Why it matters |

Span from 2026-03-18 (release day) to today. Include both anomalies and loud errors. The point is to show how a quiet Q1 release became a loud Q2 problem.
```

### 4. Identify what the dashboards should have caught

The Tulipa team has dashboards. Anomaly detection asks: why didn't the dashboards flag this?

```
Write /workshop/state/anomalies/missed-signals.md listing:

- 5 metrics that should be on a "platform health" dashboard but probably are not
- 3 alert rules that would have surfaced the regression within 1 week instead of 6
- 1 paragraph on the limits of threshold-based alerting for slow drift

Frame it as a handover note for Daan (Platform).
```

### 5. DQL the detectors

```
/schrijf-dql

Write DQL queries for the 3 alert rules you proposed in missed-signals.md. Each query:
- uses tolling baseline (e.g. 4w trailing avg) as the reference, not a fixed threshold
- triggers when current value deviates by >2 sigma over a 1h window
- includes the dimensions (service, region, etc.) needed to make the alert actionable

Save to /workshop/state/anomalies/detectors.dql.
```

---

## What to Notice

| Behaviour | Why it matters |
|-----------|----------------|
| anomalie-detector defaults to baselines, not thresholds | "Healthy" is a moving target; static thresholds miss drift |
| The 6-week delay between release and customer escalation is the story | Quiet failures cost more than loud ones |
| Proposed alerts use rolling sigma, not fixed thresholds | The platform team needs alerting that survives growth |

---

## Expected output

`/workshop/state/anomalies/`:

- `silent-drift.md` - metrics that drifted without paging
- `error-pattern.md` - error onset clustering + caused-by chains
- `timeline.md` - combined release-to-now narrative
- `missed-signals.md` - dashboard + alert recommendations
- `detectors.dql` - rolling-baseline DQL alerts

Key chains your timeline should surface:
- 2026-03-18 release -> elevated Redis writes from new RSC checkout -> evictions rise -> hit rate drops -> 2026-04-12 Redis OOM
- 2026-03-18 release -> longer payment-api hold times due to iDEAL latency -> connection pool fills faster -> 2026-04-01 pool exhaustion
- 2026-03-18 release -> new traffic pattern -> more fraud-check invocations -> cold start rate climbs -> P99 stretches

---

## Success Criteria

- All 5 deliverables present
- timeline.md spans release to current week
- At least 2 cause-effect chains identified
- Proposed detectors use rolling baselines, not fixed thresholds

---

**Next:** [Lab 06 - Root Cause Analysis](lab-06-root-cause.md)
