# Lab 06 - Root Cause Analysis

**Time:** 12-15 minutes
**Skill:** `/root-cause-analyse`
**Agent:** root-cause-detective
**Workflow:** incident-analyse
**Goal:** Consolidate findings from Labs 02-05 into one or two true root causes, with explicit hypotheses and counter-evidence.

---

## Setup

You should have, in `/workshop/state/`:
- `rum/analysis.md` (Lab 02)
- `response-times/analysis.md` (Lab 03)
- `dependencies/spofs.md` + `dependencies/ideal-cascade.md` (Lab 04)
- `anomalies/timeline.md` (Lab 05)

Plus `engagement/engagement-plan.md` with the 5 spine questions.

```bash
cd /workshop/
claude
```

---

## Steps

### 1. Inventory the findings

```
Read all four prior analyses in /workshop/state/. Produce /workshop/state/rca/findings-inventory.md - a flat list of every distinct finding so far, with:

- finding (one sentence)
- source lab (02, 03, 04, 05)
- which spine question it touches

Use the root-cause-detective agent. Do not analyse yet - just inventory.
```

### 2. Hypothesise

```
/root-cause-analyse

Use the SPOOR methodology (or whatever the root-cause-detective agent applies) to:

1. Propose 3-5 candidate root causes that could each explain at least 3 spine questions
2. For each candidate, list:
   - supporting evidence (which findings back it)
   - counter-evidence (what the data shows that would NOT be true if this were the root cause)
   - falsifiability test (what query/check would disprove it)

Save to /workshop/state/rca/hypotheses.md.
```

Likely candidates (the lab solution converges on these but you should derive them):

A. The new RSC checkout serialises too much per-request work (RUM + checkout CPU + Redis writes)
B. payment-api connection pool sized for pre-Q1 traffic shape (longer iDEAL holds + new RSC traffic pattern)
C. Mobile/iOS Safari specifically can't handle the RSC bundle (RUM device split + hydration errors)
D. Tulipa got two independent regressions in the same week (frontend RSC + iDEAL gateway degradation)
E. fraud-check cold start contention compounds checkout latency

### 3. Test each hypothesis against the timeline

```
For each candidate from hypotheses.md, run a timeline test using /workshop/state/anomalies/timeline.md:

- Does the candidate explain WHY it took 2-3 weeks for the loud errors to appear?
- Does it explain WHY storefront did not regress?
- Does it explain WHY iOS Safari is worse than Android?

Write the test results in /workshop/state/rca/timeline-tests.md.
```

### 4. Converge to root cause(s)

```
Pick the 1-2 root causes that survive the timeline test best. Write /workshop/state/rca/root-cause.md with:

- the chosen root cause(s)
- the causal chain from root cause to each spine question
- explicit acknowledgement of which findings are NOT explained by the chosen cause(s) (the residual)
- proposed remediation per root cause, ordered by impact and effort
```

### 5. Sanity check with kernwaarden-bewaker

```
Have kernwaarden-bewaker review root-cause.md. Specifically:
- Is the conclusion presented as a hypothesis, not a fact? (Even strong RCAs are hypotheses.)
- Is Yara's frontend code criticised constructively or punitively?
- Are residuals (unexplained findings) listed honestly?
- Is the remediation path realistic given the engagement budget and the BF deadline?
```

---

## What to Notice

| Behaviour | Why it matters |
|-----------|----------------|
| root-cause-detective requires counter-evidence for every hypothesis | Confirmation bias is the bug of RCAs |
| The timeline test discriminates between "release caused it" and "release + something else" | Coincident causes are real; pretending they are one cause is wrong |
| Residual findings are listed, not hidden | A consultant who claims "we explained everything" is a consultant who explained nothing |

---

## Expected output

`/workshop/state/rca/`:

- `findings-inventory.md`
- `hypotheses.md`
- `timeline-tests.md`
- `root-cause.md`

The most defensible conclusion, given the simulated data, is that **the Q1 release shipped two compounding regressions** (new checkout's heavier per-request workload + payment-api connection pool sized for a different traffic shape) and that **a third independent issue** (iDEAL gateway degradation, possibly NL-sector-wide) overlapped them. Any answer that claims one neat root cause is probably oversimplifying.

---

## Success Criteria

- All 4 RCA deliverables present
- At least 3 hypotheses with counter-evidence
- Timeline test applied to each hypothesis
- Final root-cause.md acknowledges residuals
- kernwaarden-bewaker review applied

---

**Next:** [Lab 07 - Capacity & Business Impact](lab-07-capacity-business-impact.md)
