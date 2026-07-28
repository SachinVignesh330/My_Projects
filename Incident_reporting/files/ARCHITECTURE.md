# Architecture

## Purpose

This document describes how the incident management process is structured:
how an issue moves from detection through resolution and review, which
tools are involved at each stage, and how roles interact.

## High-Level Flow

```
   ┌─────────────┐
   │  Detection   │  Monitoring alert, customer report, internal report
   └──────┬───────┘
          ▼
   ┌─────────────┐
   │ Declaration  │  On-call triages, assigns severity, declares incident
   └──────┬───────┘
          ▼
   ┌─────────────┐
   │  Response    │  IC assigned, incident channel created, SMEs engaged
   └──────┬───────┘
          ▼
   ┌─────────────┐
   │ Mitigation / │  Stop the bleeding: rollback, failover, feature flag off
   │ Resolution   │
   └──────┬───────┘
          ▼
   ┌─────────────┐
   │ Communication│  Status page updates, stakeholder notifications
   └──────┬───────┘
          ▼
   ┌─────────────┐
   │Post-Incident │  Postmortem, action items, process improvements
   │   Review     │
   └─────────────┘
```

## Components

### Detection

Sources that can trigger an incident:
- Automated monitoring/alerting (metrics thresholds, error rate spikes, health checks)
- Customer support tickets escalated as potential incidents
- Internal reports from engineers or other teams

### Declaration

- Whoever detects the issue (on-call engineer, support, etc.) makes an
  initial severity call using the severity table in the README.
- Declaring an incident automatically:
  - Creates a dedicated Slack channel (e.g. `#incident-2026-07-28-checkout`)
  - Pages the on-call Incident Commander (for SEV1/SEV2)
  - Opens an incident record in the tracking tool

### Response

- **Incident Commander (IC)** is assigned (usually the on-call lead) and
  owns coordination — not necessarily the one fixing the issue.
- **SMEs** are pulled in based on the affected system.
- **Scribe** begins logging a timeline directly in the incident channel or
  incident record.
- **Communications Lead** is assigned for SEV1/SEV2 to handle external/
  internal updates so the IC and SMEs can focus on resolution.

### Mitigation & Resolution

Common mitigation paths, roughly in order of preference (fastest / lowest
risk first):
1. Feature flag / config toggle off
2. Rollback to last known-good deploy
3. Failover to a healthy region/replica
4. Scale up/out to relieve load
5. Direct code/infra fix (highest risk, used when above don't apply)

### Communication

- Status page updated within the target time for the severity (e.g. SEV1
  within 15 minutes of declaration).
- Internal stakeholders updated via a designated channel at a regular
  cadence (e.g. every 30 minutes for SEV1) until resolved.

### Post-Incident Review

- Required for all SEV1/SEV2 incidents, optional but encouraged for SEV3.
- Blameless postmortem using `templates/postmortem-template.md`.
- Produces concrete action items with owners and due dates, tracked to
  completion (not just written down and forgotten).

## Tooling Map

| Stage | Tool |
|-------|------|
| Detection | Monitoring/alerting platform |
| Declaration & paging | PagerDuty (or equivalent) |
| Response coordination | Slack incident channel |
| Status communication | Status page tool |
| Postmortem tracking | This repo / Confluence / Notion + ticketing system for action items |

## Design Principles

- **Clear single owner (IC) per incident** — avoids diffused responsibility.
- **Blameless culture** — postmortems focus on systemic causes, not individual fault.
- **Fast, honest communication** — under-communicating erodes trust faster than bad news.
- **Action items must have owners and deadlines**, or they don't happen.

## Open Questions / Future Work

> Fill in as the process matures, e.g.:
> - Automating incident channel creation and paging from alert triggers
> - Integrating postmortem action items directly with the ticketing system
> - Defining SEV thresholds per-service rather than org-wide
