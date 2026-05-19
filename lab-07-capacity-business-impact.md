# Lab 07 - Capacity & Business Impact

**Time:** 12-15 minutes
**Skills:** `/capacity-forecast` + `/business-impact`
**Agents:** capacity-planner + business-impact-analist
**Workflow:** capacity-review
**Goal:** Translate the technical RCA into (a) a Black Friday readiness assessment and (b) a financial impact narrative the CTO can sign off on.

---

## Setup

Inputs:
- `starter-env/requirements/capacity-baseline.md`
- `starter-env/requirements/business-context.md`
- `/workshop/state/rca/root-cause.md` (Lab 06)
- `/workshop/state/dependencies/spofs.md` (Lab 04)

```bash
cd /workshop/
claude
```

---

## Steps

### 1. Black Friday readiness assessment

```
/capacity-forecast

Using /workshop/labs/lab-dynatrace/starter-env/requirements/capacity-baseline.md as the input model, produce a tier-by-tier readiness assessment for Black Friday 2026 (peak ~7.5x daily peak, with 15% YoY growth on top).

For each tier in capacity-baseline.md's "Modelling target" table:
- current sustained capacity
- required BF peak
- gap (concrete: "+ X tasks" or "+ Y GB RAM")
- mitigation (which scaling lever from the document)
- risk if not mitigated

Save to /workshop/state/capacity/bf-readiness.md.

Add a traffic-light summary table at the top: green / amber / red per tier.
```

### 2. Sequence the work

```
The maintenance window matters: capacity-baseline.md says the last RDS resize window is 2026-11-01. Produce a Gantt-style table in /workshop/state/capacity/bf-plan.md:

| Action | Owner | Effort | Deadline | Depends on |

For everything that has to happen before BF, including:
- ECS task count increases (low-risk, do early)
- RDS instance resize (the constrained one)
- Redis shard split
- Lambda provisioned concurrency
- payment-api connection pool resize - but ONLY if RDS has headroom
```

### 3. Business impact narrative

```
/business-impact

Translate the technical findings into the business narrative Eva (CTO) needs.

Inputs:
- /workshop/state/rca/root-cause.md
- /workshop/labs/lab-dynatrace/starter-env/requirements/business-context.md

Produce /workshop/state/business-impact/narrative.md with:

1. Current state: EUR per month being lost to the conversion regression (use the math already in engagement-plan.md from Lab 01)
2. Black Friday at risk: revenue exposure if the platform fails (use the EUR 4.1M single-day BF revenue + an estimated outage probability per tier)
3. Proposed investment: total EUR for the remediation + capacity uplift, against the EUR 80K + EUR 40K/year budget envelope
4. Net financial argument: "spending EUR X recovers EUR Y, payback period Z"

Be honest if the budget envelope is too small. business-impact-analist should not pretend the math works if it does not.
```

### 4. Stakeholder views

Eva, Lukas, Yara, and Daan each need a different framing:

```
Append a 'Per-stakeholder TL;DR' section to /workshop/state/business-impact/narrative.md. One 2-3 sentence paragraph per stakeholder:

- Eva (CTO): financial argument + BF risk
- Lukas (E-commerce): conversion recovery timeline + roll-back option
- Yara (Frontend): what the data does and does not say about the RSC migration
- Daan (Platform): the alert rules + dashboard items from Lab 05's missed-signals.md
```

### 5. Quality gate

```
Have kernwaarden-bewaker review narrative.md. Specifically check:

- Numbers are sourced (every EUR figure traces to business-context.md or capacity-baseline.md)
- The "Yara" paragraph is constructive, not blameful
- The investment ask is realistic against the stated budget
- BF risk is described in probability terms, not certainty
```

---

## What to Notice

| Behaviour | Why it matters |
|-----------|----------------|
| capacity-planner refuses to assume linear scaling for all tiers | RDS connection cap is non-linear; it cliffs |
| business-impact-analist insists on payback period, not just absolute EUR | Eva needs an investment thesis, not a cost list |
| The plan respects the 2026-11-01 RDS deadline | Capacity plans without sequencing are wishlists |

---

## Expected output

`/workshop/state/capacity/`:

- `bf-readiness.md` - tier-by-tier with traffic-light summary
- `bf-plan.md` - sequenced action list

`/workshop/state/business-impact/`:

- `narrative.md` - the business argument + per-stakeholder TL;DRs

Key conclusions you should land on:
- RDS connection cap is the single biggest BF risk (will fail without intervention)
- ECS task increases are quick wins (low effort, low risk)
- Redis needs replica + larger shard for BF
- Provisioned concurrency for fraud-check is small money, large impact
- Conversion recovery cannot be guaranteed by BF; the goal is "back to 3.2% by Sept, 3.3% by BF"

---

## Success Criteria

- Both capacity and business-impact deliverables exist
- Traffic-light summary present for BF readiness
- Per-stakeholder TL;DR section covers all 4 stakeholders
- Investment math is shown (not just stated)
- kernwaarden-bewaker review applied

---

**Next:** [Lab 08 - Deliver Report](lab-08-deliver-report.md)
