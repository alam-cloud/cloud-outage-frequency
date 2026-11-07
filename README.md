# Cloud Outage Frequency

**How often do AWS, Azure and GCP regions and availability zones actually fall over?**

A data-driven breakdown of 62 major cloud outages since 2020 — incident frequency, the SLI/SLO/error-budget maths behind why multi-AZ isn't enough, exactly how each incident was resolved step by step, and what each provider changed afterwards.

**Live page:** https://apj53vkwd8wk9.space.minimax.io

> Prefer a `github.io` URL? Enable GitHub Pages for this repo — **Settings → Pages → Source: Deploy from a branch → `main` / `(root)`** — and the page will also serve at https://alam-cloud.github.io/cloud-outage-frequency/

---

## Why this exists

"How do you design for regional failure?" is one of the most common platform engineering and SRE interview questions — and the textbook answer (*multi-AZ, done*) is incomplete in a way that matters.

The incident data shows four things:

1. **Major regions fail two to three times a year.** AWS logged 14 major incidents, Azure 20 and Google Cloud 12 since 2020, with a median duration of roughly eight hours.
2. **Multi-AZ inside one region is not regional resilience.** In the October 2025 `us-east-1` outage, teams that had spread replicas across availability zones still went down — because every zone in the region depends on the same regional control plane. Google said the same thing explicitly after `europe-west2`: *"there were a set of unexpected failures which caused impact outside of the zonal failure domain."*
3. **55% of major outages start with a change or code**, not hardware failure — 32% configuration or deployment changes, 23% software bugs or automation, and only 18% power or cooling.
4. **Recovery, not detection, is what takes the hours.** AWS found the root cause of the DynamoDB outage in 50 minutes and restored DNS within 2.5 hours — but the outage ran for 14.5 hours. The rest was throttles, stale leases, queue backlogs and false health checks.

## The SLI → SLO → error budget → SLA chain

| Term | What it is | Example |
|---|---|---|
| **SLI** | A precisely defined measurement | proportion of valid requests served in <400ms |
| **SLO** | Your internal target for that SLI | 99.9% over 28 days |
| **Error budget** | (1 − SLO) × window = how much failure you're allowed to buy | 99.99% → **4.32 min/month** |
| **SLA** | An external contract with financial consequences | AWS EC2 multi-AZ: 99.99%, 10–100% credits |

The maths that usually gets skipped: a stack of `ALB (99.99%) × EC2 multi-AZ (99.99%) × RDS Multi-AZ (99.95%) × S3 (99.9%)` composes to **~99.83%** — about 73 minutes of expected dependency downtime per month, which is *worse* than the 99.9% most teams are about to promise a customer. The page has an interactive calculator for this.

And the remedy attached to the SLA is a percentage of your bill: the AWS October 2025 event produced a **$1,500 credit** on a $5,000/month EC2 bill, against roughly **$27.7M** of business impact at the $1.85M/hour rate New Relic reports for high-impact outages.

## Alerting on SLOs

The Google SRE Workbook's multi-window, multi-burn-rate approach — an alert fires only when *both* the long and short window breach:

| Budget consumed | Long window | Short window | Burn rate | Action |
|---|---|---|---|---|
| 2% | 1 hour | 5 minutes | 14.4× | Page |
| 5% | 6 hours | 30 minutes | 6× | Page |
| 10% | 1 day | 2 hours | 3× | Ticket |
| 10% | 3 days | 6 hours | 1× | Ticket |

```
burn_rate        = observed_error_ratio / (1 - SLO_target)
budget_consumed  = burn_rate * alert_window / slo_period
time_to_exhaust  = slo_period / burn_rate
```

## What's in the page

- Incident frequency per year across all three providers (2020–2026)
- **SLI → SLO → error budget → SLA** explained with the burn-rate alerting table
- Every major incident plotted against the SLA error budget it burned
- **Interactive error-budget calculator** — pick your architecture, see your composed availability and how many historical outages would have blown your budget
- **Anatomy of a failure**: the AWS DynamoDB DNS race, in eight steps, with the minute-by-minute recovery
- **How each incident was resolved** — 15 incident cards across all three providers, each with the remediation steps and what changed afterwards
- Root-cause distribution
- SLA tier comparison and the service-credit reality check
- An SLO playbook: which SLIs to measure, how to pick a target, do's and don'ts

## Data sources

- [Control Plane Cloud Outage Tracker](https://controlplane.com/cloud-outage-tracker) — 62 major incidents since 2020
- [AWS Post-Event Summaries](https://aws.amazon.com/premiumsupport/technology/pes/)
- [Google Cloud Service Health incident reports](https://status.cloud.google.com/incidents.json)
- [Azure post-incident reviews](https://azure.status.microsoft/en-us/status/history/) — PLP3-1W8, YKYN-BWZ
- [Amazon EC2 SLA](https://aws.amazon.com/compute/sla/)
- [Azure Virtual Machines SLA](https://azure.microsoft.com/en-us/support/legal/sla/virtual-machines/v1_9/)
- [Google SRE Workbook — Alerting on SLOs](https://sre.google/workbook/alerting-on-slos/)
- [Uptime Institute — Annual Outage Analysis 2026](https://intelligence.uptimeinstitute.com/entity_pdf/node/156341/pdf)
- [New Relic — 2026 Observability Forecast](https://newrelic.com/resources/report/observability-forecast/2026)
- [IncidentHub — AWS outages 2025](https://blog.incidenthub.cloud/definitive-aws-outage-report-2025-reliability)

> Incident counts reflect what each provider publicly documents, not a head-to-head reliability ranking — Microsoft publishes post-incident reviews for more incidents than the others, so Azure's 20 are the broadest selected, not all of them. Durations are provider-stated worst-case service impact, not every customer's experience; where a provider's summary and timeline disagree, the provider's timeline is used. The "error budget burned" figure divides each incident's duration by 4.32 minutes, the monthly downtime allowance implied by a 99.99% SLA.

## Stack

Single self-contained `index.html` — no build step, no bundler, no framework. Chart.js from CDN, Outfit + JetBrains Mono from Google Fonts. Responsive down to mobile.

Built with [MiniMax Code](https://www.minimax.io).

## License

MIT
