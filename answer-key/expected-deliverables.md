# Expected Deliverables - lab-dynatrace

Per-lab inventaris van wat onder `/workshop/state/` moet bestaan na succesvolle uitvoering.

> **Not for cursists** - this file is for trainers/reviewers.

---

## Lab 01 - Engagement Kickoff
```
/workshop/state/engagement/engagement-plan.md
```
Quality: 5 spine questions captured, stakeholders mapped, business impact math shown (using corrected EUR rule from commit e026d1f).

## Lab 02 - RUM Deep Dive
```
/workshop/state/rum/
  framing.md
  analysis.md      # one section per Web Vital
  segments.md
  funnel-impact.md
  queries.dql
```
Quality: regression localized to `tulipa-checkout`, not generic "codebase is bad"; iOS Safari hit hardest (INP 980 ms); INP framing correct (1,240 ms is on `/checkout/address` Continue button, not Payment page).

## Lab 03 - Response Times
```
/workshop/state/response-times/analysis.md
```
Quality: payment-api identified as bottleneck; iDEAL 1,920 ms / 2.4% error called out; counter-evidence for "iDEAL is just flaky" hypothesis.

## Lab 04 - Dependency Mapping
```
/workshop/state/dependencies/
  graph.md
  spofs.md
  ideal-cascade.md
```
Quality: 4 services hitting RDS identified (cart-api, product-api, payment-api, fraud-check); SPOFs ranked; iDEAL as ~60% NL orders SPOF.

## Lab 05 - Anomaly Detection
```
/workshop/state/anomalies/
  timeline.md
  missed-signals.md
```
Quality: anomaly events correlated with Q1 release on 2026-03-18; tulipa-checkout 6->8 task scaling on 2026-04-15 reflected (not "always 8").

## Lab 06 - Root Cause Analysis
```
/workshop/state/rca/
  findings-inventory.md
  hypotheses.md
  timeline-tests.md
  root-cause.md
```
Quality: at least 3 hypotheses with counter-evidence; timeline test applied to each; final root-cause acknowledges multi-cause possibility (not one neat root cause).

## Lab 07 - Capacity & Business Impact
```
/workshop/state/capacity/
  bf-readiness.md
  bf-plan.md
/workshop/state/business-impact/narrative.md
```
Quality: tier-by-tier readiness with traffic-light; RDS 200-connection cap identified as biggest BF risk; investment math (EUR X recovers EUR Y, payback Z); per-stakeholder TL;DRs for Eva/Lukas/Yara/Daan.

## Lab 08 - Deliver Report
```
/workshop/state/report/
  outline.md
  executive-summary.md
  final-report.md
  build/index.html (+ PDF via generate_report.py)
  kernwaarden-review.md
```
Plus published at `/workshop/published/`. Quality: rapporteur stitches, does not regenerate; exec summary <= 1 page; kernwaarden-bewaker review applied.
