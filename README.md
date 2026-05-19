# Dynatrace Performance Analysis - Workshop Labs

8 hands-on labs that walk you through a full Dynatrace performance engagement using the `team-dynatrace` config. All labs share one fictional client: **Tulipa Webshop B.V.**

> **Prerequisite:** Run `workshop-pick team-dynatrace` before starting. `team-dynatrace` becomes the active team at `/workshop/` and its 12 agents, 12 skills, and 7 workflows become available to Claude Code.

## Scenario

Tulipa Webshop B.V. is a mid-sized Dutch e-commerce platform (EUR ~80M annual revenue, ~250 staff). After their Q1 2026 checkout redesign, conversion dropped 8% and customer complaints about slow pages spiked. Black Friday is in 6 months and the IT director cannot say with confidence whether the platform will survive the peak.

You have been engaged for a 3-week assessment: **assess current state -> root cause the regression -> deliver a Black Friday readiness plan**.

The labs follow the engagement chronologically. Each lab produces a deliverable in `/workshop/state/` that feeds the next.

## Note on language

Tulipa is a Dutch client, the team-dynatrace skills are named in Dutch (e.g. `/analyse-rum-data`, `/schrijf-dql`, `/genereer-klantrapport`), but lab instructions are written in English. When invoking a skill, use its exact Dutch name as shown in the lab.

## Lab Index

| # | Lab | Skill | Time |
|---|-----|-------|------|
| 01 | [Engagement Kickoff](lab-01-engagement-kickoff.md) | (no skill - business-impact-analist) | 8-12 min |
| 02 | [RUM Deep Dive](lab-02-rum-deep-dive.md) | `/analyse-rum-data` | 10-15 min |
| 03 | [Response Times](lab-03-response-times.md) | `/analyse-response-times` | 10-12 min |
| 04 | [Dependency Mapping](lab-04-dependency-mapping.md) | `/map-dependencies` | 10-12 min |
| 05 | [Anomaly Detection](lab-05-anomaly-detection.md) | `/detecteer-anomalieen` | 10-12 min |
| 06 | [Root Cause Analysis](lab-06-root-cause.md) | `/root-cause-analyse` | 12-15 min |
| 07 | [Capacity & Business Impact](lab-07-capacity-business-impact.md) | `/capacity-forecast` + `/business-impact` | 12-15 min |
| 08 | [Deliver Report](lab-08-deliver-report.md) | `/genereer-klantrapport` | 12-15 min |

## Suggested Sequences

**Quickstart (35-45 min):** 01 -> 02 -> 06 -> 08

**Frontend focus (45-60 min):** 01 -> 02 -> 05 -> 06 -> 08

**Backend focus (45-60 min):** 01 -> 03 -> 04 -> 06 -> 08

**Full engagement (full day, with breaks):** 01 -> 02 -> 03 -> 04 -> 05 -> 06 -> 07 -> 08

## Setup

> **Note:** `/workshop/labs/` is read-only by design - any files you create or change here will be wiped on the next container boot. All lab output lands in `/workshop/state/`, which is persistent.

```bash
workshop-pick team-dynatrace    # activate team-dynatrace config
cd /workshop/
claude
```

When Claude Code is running, point it at the lab you want to start:

```
Read /workshop/labs/lab-dynatrace/lab-01-engagement-kickoff.md and walk me through it.
```

## Dynatrace MCP (optional)

The team-dynatrace config ships with `.mcp.json` that exposes the official `@dynatrace-oss/dynatrace-mcp-server`. If you have a Dynatrace tenant and want to run real DQL queries during the labs, set the env vars before starting Claude:

```bash
export DT_ENVIRONMENT="https://your-tenant.apps.dynatrace.com"
export DT_CLIENT_ID="dt0s02.XXXXX"
export DT_CLIENT_SECRET="dt0s02.XXXXX.YYYYY"
```

The labs work fine **without** a live tenant - the `starter-env/` contains realistic simulated exports.

## Materials

All shared scenario materials live in [`starter-env/`](starter-env/):

- [`client-brief.md`](starter-env/client-brief.md) - intake notes from kickoff
- [`environment/`](starter-env/environment/) - current-state Dynatrace exports (RUM, response times, dependency map, infra metrics, error snapshot)
- [`requirements/`](starter-env/requirements/) - new-work specs (incident timeline, business context, capacity baseline)
