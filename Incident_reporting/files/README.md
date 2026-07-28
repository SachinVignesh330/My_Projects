# Incident Management Process

This repository documents the incident management process for [Team/Org
Name]: how incidents are detected, declared, responded to, communicated,
and reviewed. It's the source of truth for how we handle production
incidents.

## Overview

An **incident** is any unplanned event that degrades or disrupts a service
below acceptable levels. This repo defines:
- Severity levels and what each one means
- Roles and responsibilities during an incident
- Step-by-step response process
- Communication and escalation paths
- Post-incident review requirements

## Repository Structure

```
.
├── docs/
│   ├── ARCHITECTURE.md    # How the incident process/tooling fits together
│   └── RUNBOOK.md         # Step-by-step response procedures
├── templates/
│   ├── incident-report-template.md
│   └── postmortem-template.md
└── README.md
```

## Severity Levels

| Severity | Definition | Example | Response time |
|----------|------------|---------|----------------|
| SEV1 | Full outage or critical data loss, affects all/most users | Site down, payments failing | Immediate, all-hands |
| SEV2 | Major functionality degraded, affects a subset of users | One region down, key feature broken | < 30 min |
| SEV3 | Minor issue, workaround available | Slow response times, cosmetic bug | Next business day |
| SEV4 | Cosmetic or negligible impact | Typo in UI, non-user-facing log noise | Backlog |

## Roles

| Role | Responsibility |
|------|-----------------|
| Incident Commander (IC) | Owns the incident end-to-end; coordinates response, makes final calls |
| Communications Lead | Updates stakeholders, status page, customers |
| Subject Matter Expert(s) (SMEs) | Investigate and remediate the technical issue |
| Scribe | Logs timeline of actions/decisions in real time |

## Tooling

| Purpose | Tool |
|---------|------|
| Alerting / paging | PagerDuty (or equivalent) |
| Incident channel | Slack (#incidents, auto-created per incident) |
| Status page | Statuspage / Instatus (or equivalent) |
| Postmortem docs | This repo / Confluence / Notion |

## Getting Started

New to on-call or incident response? Read, in order:
1. [Architecture](docs/ARCHITECTURE.md) — how the pieces fit together
2. [Runbook](docs/RUNBOOK.md) — what to actually do during an incident
3. `templates/incident-report-template.md` — used to log the incident live
4. `templates/postmortem-template.md` — used after resolution

## Contributing

This process evolves. To propose a change:
1. Open a PR against this repo with the proposed edit.
2. Tag the ITOps/SRE lead for review.
3. Once merged, announce the change in #itops-infra (or your team channel).

## Ownership

| Role | Owner |
|------|-------|
| Process owner | ITOps / SRE Team |
| On-call rotation | See PagerDuty schedule |
| Slack channel | #incidents |
