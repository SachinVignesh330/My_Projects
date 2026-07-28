# Runbook

Step-by-step procedures for handling an incident from detection through
closure. Use this during an active incident — it's meant to be followed in
real time, not just read once.

## 1. Detecting & Declaring an Incident

1. An alert fires, or someone reports an issue.
2. Do a quick sanity check: is this real, and is it actually impacting users?
3. Assign an initial severity (see README severity table). When unsure,
   **default to the higher severity** — you can downgrade later, but a slow
   start on a real SEV1 is costly.
4. Declare the incident:
   - Page the on-call Incident Commander (SEV1/SEV2), or
   - Open a ticket and self-assign (SEV3/SEV4).
5. Confirm an incident channel is created (e.g. `#incident-YYYY-MM-DD-shortname`).

## 2. Assembling the Response

1. IC joins the channel, confirms they're taking command, pins a message
   stating: severity, systems affected, IC name, start time.
2. IC pulls in SMEs based on the affected system(s).
3. For SEV1/SEV2, IC assigns a **Communications Lead** and a **Scribe**.
4. Scribe starts a running timeline directly in the channel, e.g.:
   ```
   14:02 - Alert fired: checkout error rate > 5%
   14:05 - IC (Jane) takes command
   14:07 - SME (Raj) confirms DB connection pool exhaustion
   14:12 - Mitigation: scaling DB connection pool
   ```

## 3. Mitigating the Issue

Work through options roughly in this order (fastest / lowest-risk first):
1. **Feature flag off** — disable the problematic feature.
2. **Rollback** — revert to last known-good deploy.
   ```bash
   # Example — adapt to your deploy tooling
   ./deploy.sh rollback --service checkout --to <last-good-tag>
   ```
3. **Failover** — shift traffic to a healthy region/replica.
4. **Scale** — add capacity if the issue is load-related.
5. **Direct fix** — only if none of the above apply; requires extra care
   and, ideally, a second reviewer even under time pressure.

At each step, the Scribe logs what was tried and the result.

## 4. Communicating

| Severity | Status page update | Internal update cadence |
|----------|--------------------|--------------------------|
| SEV1 | Within 15 min of declaration | Every 30 min until resolved |
| SEV2 | Within 30 min | Every hour until resolved |
| SEV3 | Optional | On resolution |
| SEV4 | Not required | Not required |

Status update template:
```
[INVESTIGATING/IDENTIFIED/MONITORING/RESOLVED]
We are aware of an issue affecting <system>. <Brief, factual impact
description.> We are actively working on it. Next update by <time>.
```

## 5. Declaring Resolution

1. Confirm the fix is holding (monitor key metrics for at least 15–30 min
   post-mitigation before calling it resolved).
2. Post a final status update (internal and external).
3. IC formally closes the incident in the channel and tracking tool,
   noting the resolution time.
4. Schedule the postmortem (required for SEV1/SEV2) within 48 hours.

## 6. Post-Incident Review

1. Use `templates/postmortem-template.md`.
2. Cover: timeline, root cause(s), what went well, what didn't, action items.
3. Keep it blameless — focus on systems and process, not individuals.
4. Assign an owner and due date to every action item.
5. Track action items to completion in the ticketing system, not just in
   the postmortem doc.

## 7. Common Pitfalls to Avoid

- **Under-declaring severity** to avoid "causing a fuss" — default higher,
  downgrade later.
- **IC also trying to fix the issue themselves** — keep coordination and
  hands-on-keyboard work separate where possible.
- **Silence during a long incident** — even "no update yet, still
  investigating" is better than nothing.
- **Skipping the postmortem** because the fix was quick — recurring small
  issues are often signs of a bigger systemic problem.

## Escalation Contacts

| Role | Contact |
|------|---------|
| Primary on-call | PagerDuty rotation |
| Secondary on-call | PagerDuty rotation |
| Engineering leadership (SEV1 only) | See internal contact list |
