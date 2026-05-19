# Lab 01 - Engagement Kickoff

**Time:** 8-12 minutes
**Skill:** (none - direct agent invocation)
**Agents:** business-impact-analist + kernwaarden-bewaker
**Workflow:** volledige-klantanalyse (planning phase)
**Goal:** Turn raw kickoff notes into a structured engagement plan with clear stakeholder mapping and a 5-question spine for the rest of the engagement.

---

## Setup

> **Note:** `/workshop/labs/` is read-only. All work in this lab lands in `/workshop/state/engagement/` so it survives container restarts and feeds the next labs.

You should already have `workshop-pick team-dynatrace` active. Verify with:

```bash
ls /workshop/.claude/agents/  # should list business-impact-analist, dql-expert, etc.
```

If that directory is empty, run `workshop-pick team-dynatrace` first.

Start Claude Code from `/workshop/`:

```bash
cd /workshop/
claude
```

---

## Steps

### 1. Read the brief

```
Read /workshop/labs/lab-dynatrace/starter-env/client-brief.md and summarise the engagement in 5 bullets: who, what is broken, scope, stakeholders, "good" at handover.
```

Confirm the summary matches your reading of the brief.

### 2. Lift the 5 questions from the incident timeline

The incident timeline contains the canonical questions the engagement must answer:

```
Read /workshop/labs/lab-dynatrace/starter-env/requirements/incident-timeline.md. Extract the 5 "what the engagement must explain" questions verbatim. Save them as the opening section of /workshop/state/engagement/engagement-plan.md under a heading 'Engagement Spine'.
```

### 3. Build the engagement plan

```
Using the business-impact-analist agent, expand /workshop/state/engagement/engagement-plan.md with:

- Stakeholder concern -> proposed deliverable -> owning team-dynatrace agent -> week
  (one row per stakeholder from client-brief.md)
- Week 1 / Week 2 / Week 3 milestone breakdown aligned to the 5 spine questions
- "In scope / out of scope" section pulled from the brief
- A sign-off block for Eva (CTO) + Lukas (Head of E-commerce)
```

### 4. Quantify the stakes

```
Read /workshop/labs/lab-dynatrace/starter-env/requirements/business-context.md. Append a 'Stakes' section to /workshop/state/engagement/engagement-plan.md with:
- Monthly revenue lost to the conversion regression (show the math)
- Annualised number
- Eva's "1% conversion = EUR 4M / year" rule of thumb
- The two Black Friday non-negotiables (peak survival, conversion back to 3.4%)
```

### 5. Quality gate

```
Have kernwaarden-bewaker review /workshop/state/engagement/engagement-plan.md and flag anything that:
- promises a result the data cannot deliver
- skips stakeholder communication
- hides risk

Address the flags before moving on.
```

---

## What to Notice

| Behaviour | Why it matters |
|-----------|----------------|
| The 5 spine questions become the table of contents for the final report | Every subsequent lab answers one or more of them |
| business-impact-analist insists on the financial math up front | Tech findings without EUR attached are easy to ignore |
| kernwaarden-bewaker catches overpromising | Conversion recovery is not something the consultant can guarantee |

---

## Expected output

`/workshop/state/engagement/engagement-plan.md` with:

- Engagement Spine: the 5 verbatim questions
- Stakeholder -> deliverable -> agent -> week table
- Week 1/2/3 milestones
- In/Out scope section
- Stakes section with revenue math
- Sign-off block

---

## Success Criteria

- Engagement plan exists and is reviewed by kernwaarden-bewaker
- All 5 spine questions present
- All 4 stakeholders from the brief appear at least once
- Revenue math is shown explicitly (not just the final number)

---

**Next:** [Lab 02 - RUM Deep Dive](lab-02-rum-deep-dive.md)
