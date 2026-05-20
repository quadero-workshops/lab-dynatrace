# Lab 08 - Deliver Report

**Time:** 25-45 minutes
**Skill:** `/genereer-klantrapport`
**Agents:** klant-communicator + rapporteur + kernwaarden-bewaker
**Workflow:** rapportage-workflow
**Goal:** Stitch the artefacts from Labs 01-07 into one client-ready final report using the Quadero template, then publish it.

---

## Prerequisites

All prior lab deliverables should exist:
```bash
for f in engagement/engagement-plan.md rum/analysis.md response-times/analysis.md dependencies/spofs.md anomalies/timeline.md rca/root-cause.md capacity/bf-readiness.md business-impact/narrative.md; do
  ls /workshop/state/$f 2>/dev/null || echo "MISSING /workshop/state/$f"
done
```

## Setup

You should have, in `/workshop/state/`:

- `engagement/engagement-plan.md` (Lab 01)
- `rum/analysis.md` + supporting files (Lab 02)
- `response-times/analysis.md` (Lab 03)
- `dependencies/graph.md` + `dependencies/spofs.md` + `dependencies/ideal-cascade.md` (Lab 04)
- `anomalies/timeline.md` + `anomalies/missed-signals.md` (Lab 05)
- `rca/root-cause.md` (Lab 06)
- `capacity/bf-readiness.md` + `capacity/bf-plan.md` (Lab 07)
- `business-impact/narrative.md` (Lab 07)

The Quadero report template lives in the team-dynatrace scaffold: `/workshop/templates/reports/quadero-report-template.html` (with CSS sibling and `/workshop/scripts/generate_report.py`).

### Workshop platform commands used in this lab

This lab uses two workshop platform commands you have not seen in earlier labs:

- `publish` - copies a built report to `/workshop/published/` so it shows up at your seat URL.
- `workshop-export` - bundles the entire `/workshop/state/` directory into a downloadable zip.

See the deelnemer-handleiding for the full command reference.

```bash
cd /workshop/
claude
```

---

## Steps

### 1. Outline

```
List all artefacts in /workshop/state/. Group them by the engagement spine question they answer. Produce the final report outline at /workshop/state/report/outline.md. Structure:

1. Executive summary (1 page max)
2. Engagement scope and approach
3. Findings (organised by the 5 spine questions)
4. Root cause analysis
5. Remediation roadmap
6. Black Friday readiness
7. Recommended dashboards & alerts (handover to platform team)
8. Recommendations for next engagement
9. Appendix: DQL queries, evidence trail

Use the klant-communicator agent.
```

### 2. Executive summary first

```
Write the Executive Summary section to /workshop/state/report/executive-summary.md. Constraints:

- max 1 page (about 350 words)
- must reference all 4 stakeholders' concerns (without naming them)
- 3 hard numbers (e.g. EUR 920K / month at risk, BF readiness X of Y tiers green, conversion target 3.3% by Q4)
- one paragraph on the root cause framed as a hypothesis, not a fact
- finish with the one decision the CTO needs to make next (likely: approve the EUR X investment for BF prep)
- no jargon - assumes Eva and the audit team will read this
```

### 3. Full report draft

```
/genereer-klantrapport

Assemble /workshop/state/report/final-report.md using the outline. Pull content directly from the artefacts in /workshop/state/ - quote and cite, do not regenerate. Each section opens with one paragraph of context, then the body, then a 'Evidence' subsection listing the source artefacts and the DQL queries that reproduce the finding.

Use the klant-communicator agent to handle stitching and the rapporteur agent for any Quadero-house-style polishing.
```

### 4. Render to HTML using the Quadero template

```
Read /workshop/templates/reports/quadero-report-template.html and /workshop/scripts/generate_report.py. Use the script (if it accepts a markdown input) or follow the template's variable convention to render /workshop/state/report/final-report.md into /workshop/state/report/build/index.html.

Keep the markdown source alongside the HTML.
```

### 5. Quality gate

```
Have kernwaarden-bewaker review /workshop/state/report/final-report.md against Quadero core values. Flag:

- any claim not supported by an artefact in /workshop/state/
- any recommendation without a clear owner or deadline
- any language that overpromises (e.g. "fully restored", "guaranteed", "complete")
- the residual findings from Lab 06 must be visible in the report, not hidden in an appendix
- the Yara/RSC narrative must remain constructive

Write the review to /workshop/state/report/kernwaarden-review.md. Apply the review back to the report.
```

### 6. Publish

```
Use the `publish` command to make the rendered report available at the workshop seat URL:

publish /workshop/state/report/build

Confirm with `ls /workshop/published/` that the report is there.
```

### 7. Take-home

Optional - zip the entire engagement deliverable so the consultant can download it:

```bash
workshop-export
```

---

## What to Notice

| Behaviour | Why it matters |
|-----------|----------------|
| klant-communicator handles the stitching, rapporteur handles the style | Two agents, two responsibilities - cleaner than one mega-prompt |
| Executive summary is written before the body | The summary forces clarity about the story; if you can't write it, the body isn't ready |
| Residuals stay visible | Hidden residuals = hidden risk = bad consulting |
| `publish` is the official path to client-visible output | Writing to /workshop/published/ directly is not the convention |

---

## Expected output

`/workshop/state/report/`:

- `outline.md`
- `executive-summary.md`
- `final-report.md`
- `build/index.html` (+ CSS, images)
- `kernwaarden-review.md`

And `/workshop/published/index.html` viewable at `apps.workshop.quaderolabs.com/<your-seat>/`.

---

## Success Criteria

- Final report exists in both markdown and rendered HTML
- Executive summary is under 1 page
- All 5 spine questions from Lab 01 are addressed in the body
- Residuals from Lab 06's RCA appear in the report
- Report is published and viewable via the seat URL
- kernwaarden-bewaker review applied

---

## Wrap-up

You now have a full Dynatrace performance engagement produced through team-dynatrace's agents and skills. The same pattern - one team-config, a continuous scenario across linked skills, a published HTML deliverable - is how you would run a real Quadero Dynatrace engagement.

Sister lab: [lab-splunk](https://github.com/quadero-workshops/lab-splunk) runs the same pattern for the team-splunk config.
