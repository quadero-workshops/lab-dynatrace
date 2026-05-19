# Lab 02 - RUM Deep Dive

**Time:** 10-15 minutes
**Skill:** `/analyse-rum-data`
**Agent:** frontend-specialist
**Workflow:** frontend-deep-dive
**Goal:** Explain what Real User Monitoring data says about the conversion regression, with evidence per device class and funnel step.

---

## Setup

Input: `starter-env/environment/rum-snapshot.md` and the engagement plan from Lab 01.

```bash
cd /workshop/
claude
```

---

## Steps

### 1. Frame the question

```
Using the frontend-specialist agent, restate spine question #1 from the engagement plan: "Why did tulipa-checkout Web Vitals regress while tulipa-storefront did not?". Frame the RUM analysis around that question. Do not look at data yet.

Write the framing to /workshop/state/rum/framing.md.
```

### 2. Analyse the RUM snapshot with /analyse-rum-data

```
/analyse-rum-data

Analyse /workshop/labs/lab-dynatrace/starter-env/environment/rum-snapshot.md. Use the PULSE methodology that frontend-specialist applies. Deliverables in /workshop/state/rum/:

1. analysis.md - one section per Web Vital (LCP, INP, CLS, FCP, TTFB) covering:
   - current state vs baseline
   - severity (green/amber/red, using Dynatrace defaults: see team CLAUDE.md)
   - which application/device/geography is worst affected
   - leading hypothesis for the cause

2. segments.md - findings broken down by:
   - application (storefront vs checkout)
   - device class (desktop / Android / iOS Safari)
   - geography (NL / BE / DE)

3. funnel-impact.md - mapping of "where users drop off" to "where the worst Web Vitals are". The Address -> Payment step is the headline.
```

### 3. Cross-reference JavaScript errors

```
Append a 'JS errors and Web Vitals' section to /workshop/state/rum/analysis.md correlating the top 5 errors from rum-snapshot.md with the worst-performing pages. The TypeError on 'price' and the hydration error are the prime suspects - explain why each is consistent with the RUM regression pattern.
```

### 4. Write the DQL queries the platform team can re-run

The Tulipa platform team must be able to reproduce these findings:

```
/schrijf-dql

Generate the DQL queries that would reproduce the RUM findings in /workshop/state/rum/analysis.md against the real Dynatrace tenant. One query per section heading. Save to /workshop/state/rum/queries.dql.

Each query:
- explicit time range (last 7d vs baseline 4 weeks earlier - use `from:` and `to:`)
- filter on application.name in ("tulipa-checkout", "tulipa-storefront") where relevant
- explicit percentile (P75 - the Dynatrace default for Web Vitals)
- one-line comment above each query explaining what it shows
```

### 5. Quality gate

```
Have kernwaarden-bewaker review the RUM analysis. Specifically check:
- every claim cites a number from rum-snapshot.md
- no claim asserted about the new checkout that does not also acknowledge that storefront did NOT regress
- the iOS Safari finding is reported sensitively (Yara, the frontend lead, takes Q1 personally per business-context.md)
```

---

## What to Notice

| Behaviour | Why it matters |
|-----------|----------------|
| frontend-specialist segments by device + geography by default | Aggregate P75 hides the iOS Safari catastrophe |
| The funnel-step correlation is the punchline | "Worst Web Vitals are exactly where users drop off" - this is the conversion story |
| DQL queries are written even though we are working off a snapshot | The platform team owns reproducibility after handover |

---

## Expected output

`/workshop/state/rum/`:

- `framing.md`
- `analysis.md` - one section per Web Vital + JS error correlation
- `segments.md` - by application, device, geography
- `funnel-impact.md` - drop-off vs Web Vital map
- `queries.dql` - reproducible DQL

Key facts your analysis should land on:
- The regression is entirely in `tulipa-checkout`, not `tulipa-storefront`
- iOS Safari is hit hardest (INP P75 980 ms)
- Address -> Payment drop-off is +11pp, matching the worst INP (1,240 ms)
- The TypeError on 'price' is consistent with the new RSC hydration story

---

## Success Criteria

- All 5 deliverables exist
- analysis.md has a section per Web Vital
- The hypothesis is grounded in the storefront vs checkout split, not "the codebase is bad"
- DQL queries have explicit time bounds and applications

---

**Next:** [Lab 03 - Response Times](lab-03-response-times.md)
