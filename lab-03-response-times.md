# Lab 03 - Response Times

**Time:** 25-45 minutes
**Skill:** `/analyse-response-times`
**Agent:** performance-analist
**Workflow:** performance-review
**Goal:** Establish whether backend response times alone can explain the checkout regression, or whether the frontend findings (Lab 02) are dominant.

---

## Prerequisites

This lab can run after Lab 01 alone (no Lab 02 deliverables strictly required), but the framing in `/workshop/state/engagement/engagement-plan.md` improves quality:
```bash
ls /workshop/state/engagement/engagement-plan.md 2>/dev/null || echo "Lab 01 not done - run it first for the spine questions"
```

## Setup

Input: `starter-env/environment/response-times.md`.

```bash
cd /workshop/
claude
```

---

## Steps

### 1. Profile the services

```
/analyse-response-times

Analyse /workshop/labs/lab-dynatrace/starter-env/environment/response-times.md. For each service (tulipa-storefront, tulipa-checkout, product-api, cart-api, payment-api, fraud-check):

- one paragraph on current state with P50/P90/P95/P99 in milliseconds
- whether the service is healthy, slow, or pathological
- the single worst endpoint per service

Use the performance-analist agent. Save to /workshop/state/response-times/analysis.md.
```

### 2. Spot the outliers

```
Append a 'Outliers worth investigating' section to /workshop/state/response-times/analysis.md with:

- payment-api /v1/confirm P99 = 14,210 ms (this is the "Place order" call)
- payment-api regressed 38% on P95 despite no deploys
- fraud-check 18% cold start rate
- /checkout/payment endpoint 7.8s P99

For each outlier: what evidence points to a cause, what evidence is missing.
```

### 3. Period comparison with /vergelijk-periodes

```
/vergelijk-periodes

Use the period comparison data in response-times.md (the "Period comparison vs pre-Q1 baseline" table) to:

1. List which services regressed and which did not
2. Flag the contradiction: tulipa-checkout did not exist pre-Q1, so its "+0%" is meaningless - what does it tell us about how the team measures changes?
3. Explain why payment-api regressed +38% even though it was not redeployed

Save to /workshop/state/response-times/period-comparison.md.
```

### 4. DQL the queries

```
/schrijf-dql

Write the DQL queries for:
- service response time percentiles P50/P90/P95/P99 by endpoint, last 7d
- response time period-over-period comparison (last 7d vs baseline 4 weeks ago)
- cold start rate for the fraud-check Lambda

Save to /workshop/state/response-times/queries.dql.
```

### 5. Frame against the spine

```
Append a 'Answers to spine questions' section to /workshop/state/response-times/analysis.md mapping the findings to spine questions #3 and #4 (connection pool exhaustion and Redis evictions). State explicitly which spine questions are NOT answered by response time data alone (#1 RUM regression, #2 iDEAL).
```

---

## What to Notice

| Behaviour | Why it matters |
|-----------|----------------|
| performance-analist refuses to compare a service against itself when it did not exist | Period comparison needs a meaningful baseline |
| P99 is highlighted alongside P50/P90/P95 | At BF traffic, P99 events fire thousands of times per minute |
| Cold start rate is reported as a first-class metric | 18% cold starts on a busy Lambda is silent latency |

---

## Expected output

`/workshop/state/response-times/`:

- `analysis.md` - one section per service + outliers + spine answers
- `period-comparison.md` - what regressed and why the baseline question matters
- `queries.dql` - reproducible DQL

Key facts to surface:
- POST /checkout/place-order P99 = 11+ seconds
- payment-api /v1/confirm P99 = 14 seconds (the slowest call in the engagement)
- payment-api regressed +38% without a deploy -> downstream or dependency cause
- fraud-check cold start rate 18% is high enough to require provisioned concurrency

---

## Success Criteria

- All 6 services analysed
- At least 4 outliers explicitly listed
- Period comparison file calls out the "did not exist before" baseline issue
- Spine question mapping is honest about what backend data does and does not explain

---

**Next:** [Lab 04 - Dependency Mapping](lab-04-dependency-mapping.md)
