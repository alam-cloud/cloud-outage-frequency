# Cloud Outage Frequency

**How often do AWS and Azure regions and availability zones actually fall over?**

A data-driven breakdown of 62 major cloud outages since 2020 — incident frequency, error-budget burn against real SLAs, and the root causes behind them.

**Live page:** https://alam-cloud.github.io/cloud-outage-frequency/

---

## Why this exists

"How do you design for regional failure?" is one of the most common platform engineering and SRE interview questions — and the textbook answer (*multi-AZ, done*) is incomplete in a way that matters.

The incident data shows three things:

1. **Major regions fail two to three times a year.** AWS logged 14 major incidents and Azure 20 since 2020, with a median duration of roughly eight hours.
2. **Multi-AZ inside one region is not regional resilience.** In the October 2025 `us-east-1` outage, teams that had spread replicas across availability zones still went down — because every zone in the region depends on the same regional control plane.
3. **55% of major outages start with a change or code**, not hardware failure — an incomplete storage allow list published region-wide, a malformed certificate pushed to all zones at once, a race condition in DynamoDB's DNS automation.

## The SLA maths

| Deployment | SLA | Downtime / month | Downtime / year |
|---|---|---|---|
| AWS EC2, 2+ AZs | 99.99% | 4.32 min | 52.6 min |
| Azure VM, 2+ AZs | 99.99% | 4.32 min | 52.6 min |
| AWS RDS Multi-AZ | 99.95% | 21.6 min | 4.4 h |
| Azure VM, availability set | 99.95% | 21.6 min | 4.4 h |
| AWS single EC2 instance | 99.5% | 216 min | 43.2 h |
| Azure single VM, Premium SSD | 99.9% | 43.2 min | 8.8 h |

A single 15-hour outage consumes **17× the entire annual 99.99% downtime allowance**. The AWS service credit for that event was roughly 30% of one month's EC2 bill — against an estimated **~$27.7M** in losses, at $1.85M/hour for high-impact business outages. The credit covers about **1/18,500th** of the damage.

## What's in the page

- Incident frequency per year, AWS vs Azure (2020–2026)
- Every major incident plotted against the SLA error budget it burned
- Incident timelines with provider-stated root causes
- Root-cause distribution
- Full SLA tier comparison and the service-credit reality check
- Four architecture conclusions drawn from the incident data

## Data sources

- [Control Plane Cloud Outage Tracker](https://controlplane.com/cloud-outage-tracker) — 62 major incidents since 2020
- [AWS Post-Event Summaries](https://aws.amazon.com/premiumsupport/technology/pes/)
- [Amazon EC2 SLA](https://aws.amazon.com/compute/sla/)
- [Azure Virtual Machines SLA](https://azure.microsoft.com/en-us/support/legal/sla/virtual-machines/v1_9/)
- [Azure status history and post-incident reviews](https://azure.status.microsoft/en-us/status/history/)
- [Uptime Institute — Annual Outage Analysis 2026](https://intelligence.uptimeinstitute.com/entity_pdf/node/156341/pdf)
- [New Relic — 2026 Observability Forecast](https://newrelic.com/resources/report/observability-forecast/2026)
- [IncidentHub — AWS outages 2025](https://blog.incidenthub.cloud/definitive-aws-outage-report-2025-reliability)

> Incident counts reflect what each provider publicly documents, not a head-to-head reliability ranking — Microsoft publishes post-incident reviews for more incidents than the others. Durations are provider-stated worst-case service impact, not every customer's experience. The "error budget burned" figure divides each incident's duration by 4.32 minutes, the monthly downtime allowance implied by a 99.99% SLA.

## Stack

Single self-contained `index.html` — no build step, no bundler, no framework. Chart.js from CDN, Outfit + JetBrains Mono from Google Fonts. Responsive down to mobile.

Built with [MiniMax Code](https://www.minimax.io).

## License

MIT
