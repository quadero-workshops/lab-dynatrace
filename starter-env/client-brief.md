# Client Brief - Tulipa Webshop B.V.

> Intake notes from the kickoff meeting on 2026-05-04. Captured by the lead consultant.

## Company

- **Name:** Tulipa Webshop B.V.
- **Sector:** E-commerce (home & garden, plus a fast-growing fashion line)
- **Size:** ~250 employees, HQ in Utrecht, warehouse in Tilburg
- **Revenue:** EUR ~80M annual, ~3M orders/year
- **Splunk... sorry, Dynatrace footprint:** Dynatrace SaaS, ~140 hosts, ~80 services, RUM enabled on `tulipa.nl` and `tulipa.be`. Environment: `https://abc12345.apps.dynatrace.com`.

## Stakeholders

| Name | Role | Concern |
|------|------|---------|
| Eva van Leeuwen | CTO | "Conversion is down 8% since the Q1 checkout redesign. We have 6 months until Black Friday and I cannot say if we will survive the peak." |
| Lukas Verhoeven | Head of E-commerce | "Customers are emailing support about slow pages. The dev team says everything looks green in Dynatrace. I do not know who is right." |
| Yara Smit | Lead Frontend Engineer | "We refactored checkout to React Server Components. Lighthouse looked great in staging. Real users seem to disagree." |
| Daan Hofstra | Head of Platform | "Our payment service has gotten flakier since Q1. I suspect a dependency issue but I have not had time to chase it." |

## Engagement scope (3 weeks)

1. **Week 1 - Assess.** Full-stack performance baseline: RUM, response times, dependencies, infra. Quick wins identified.
2. **Week 2 - Root cause.** Explain the conversion regression. Map every finding to a business-relevant metric.
3. **Week 3 - Plan.** Black Friday readiness assessment with capacity forecast and a prioritised remediation plan.

Out of scope: code rewrite of the checkout, vendor renegotiation with payment provider, SOC2 audit prep (separate engagement next quarter).

## Known issues (per intake)

- Conversion rate dropped from 3.4% to 3.1% between week 12 and week 14 (immediately after Q1 release).
- Average time-on-checkout went from 38s to 71s for mobile users.
- Customer support tickets containing "slow", "freeze", or "won't load" up 240% vs same period last year.
- Payment service throws sporadic 502s - currently retried client-side, masking real impact.
- Mobile users on iOS Safari report layout shifts mid-checkout.
- Black Friday 2025 saw 7x normal traffic; infra "just barely held" per Daan.

## Dynatrace context

- Real User Monitoring enabled on production, staging excluded.
- Service-level monitoring via OneAgent on all hosts.
- Application name: `tulipa-storefront` and `tulipa-checkout` (the latter is the post-Q1 split).
- Geographic tagging: NL (88%), BE (10%), DE (2%).
- Data retention: 35 days hot, no Grail long-term currently.

## What "good" looks like at handover

- A documented explanation of what changed between week 12 and week 14 that caused the conversion drop.
- A prioritised list of frontend, backend, and infra issues with effort estimates.
- A Black Friday readiness assessment: green / amber / red per service.
- Capacity forecast for the Black Friday peak (modelled at 7x baseline as per last year).
- A signed-off final report (CTO + Head of E-commerce).
- The platform team can reproduce every finding by running the documented DQL queries themselves.

## Access

- Dynatrace web UI: `https://abc12345.apps.dynatrace.com`
- Read-only Dynatrace OAuth client provisioned for the engagement (scope: storage:*:read, app-engine:apps:run).
- Production Grafana, GitLab, and PagerDuty read-only access on request.
