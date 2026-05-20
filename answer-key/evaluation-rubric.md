# Evaluation Rubric - lab-dynatrace

Quick pass/fail per lab. See `expected-deliverables.md` for full deliverable list.

> **Not for cursists** - trainers/reviewers only.

---

## Lab 01 - Engagement Kickoff
- [ ] 5 spine questions in engagement-plan (matching incident-timeline.md)
- [ ] EUR math uses 0.1pp = ~EUR 3.7M rule (not the old "EUR 4M / 1%" mistake)
- [ ] Stakeholders Eva/Lukas/Yara/Daan all mapped

## Lab 02 - RUM Deep Dive
- [ ] Regression scoped to tulipa-checkout (not "the codebase")
- [ ] iOS Safari called out as worst device class
- [ ] INP 1,240 ms correctly framed as `/checkout/address` Continue-button click (not Payment page)
- [ ] DQL queries use explicit dates (2026-04-27..05-03 vs pre-Q1 2026-02-23..03-01), not generic "4 weeks earlier"

## Lab 03 - Response Times
- [ ] payment-api flagged as bottleneck
- [ ] iDEAL 1,920 ms / 2.4% error rate cited
- [ ] Counter-evidence applied (e.g. "iDEAL is just flaky" disproved)

## Lab 04 - Dependency Mapping
- [ ] All 4 RDS-touching services identified (cart, product, payment, fraud-check)
- [ ] SPOFs ranked, not just listed
- [ ] iDEAL cascade documented separately

## Lab 05 - Anomaly Detection
- [ ] Onset clustered on 2026-03-18 (Q1 release)
- [ ] tulipa-checkout scaling 6 -> 8 on 2026-04-15 reflected (not "always 8")
- [ ] Missed-signals.md identifies what alerts SHOULD have fired

## Lab 06 - Root Cause Analysis
- [ ] At least 3 hypotheses with counter-evidence
- [ ] Timeline test per hypothesis
- [ ] Final root-cause acknowledges multi-cause (Q1 regression + connection pool + iDEAL = compounding)
- [ ] Spoiler block not used as a shortcut (cursist did own analysis first)

## Lab 07 - Capacity + Business Impact
- [ ] Tier-by-tier with traffic-light
- [ ] RDS 200-connection cap = biggest BF risk
- [ ] Investment math shown, not just stated
- [ ] Per-stakeholder TL;DR present
- [ ] Budget envelope (EUR 80K + 40K/year) confronted honestly

## Lab 08 - Deliver Report
- [ ] Exec summary <= 1 page
- [ ] No overpromise language ("fully restored", "guaranteed")
- [ ] Yara/RSC narrative constructive, not blameful
- [ ] PDF rendered via generate_report.py
- [ ] `publish` executed, file in `/workshop/published/`

---

## Red flags

- "1 root cause" claims in lab-06 (the scenario is genuinely multi-cause)
- EUR math that doesn't reconcile with the 0.3pp regression = ~EUR 11M/year
- "tulipa-checkout has always been 8 tasks" (it was 6 at launch, scaled to 8 on 04-15)
- BF 2025 "Peak RPS to checkout flow 3,800" cited as fact (was never separately measured pre-Q1)
- Capacity plan that ignores 2026-11-01 RDS maintenance deadline
