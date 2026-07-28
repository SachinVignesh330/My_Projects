# Runbook

Operational procedures for day-to-day and incident work on this
infrastructure. Keep this doc updated whenever a procedure changes.

## 1. Standard Change: Deploying an Infrastructure Update

1. Create a branch and make the Terraform/Ansible change.
2. Open a PR — CI posts the `terraform plan` output automatically.
3. Review the plan carefully, especially any `destroy` actions.
4. Get required approvals (1 for staging, 2 for prod).
5. Merge → pipeline applies to `dev` automatically.
6. Promote to `staging`, verify, then promote to `prod` via the pipeline's
   manual approval step.
7. Run the relevant Ansible playbook against the updated inventory if
   configuration (not just infrastructure) changed.

## 2. Emergency Rollback

1. Identify the last known-good commit (check pipeline history / releases).
2. `git revert` the offending commit rather than force-pushing.
3. Push and let the pipeline re-plan and re-apply the previous state.
4. If Terraform state is corrupted or out of sync:
   ```bash
   terraform state list
   terraform state show <resource>
   terraform plan   # confirm expected diff before applying
   ```
5. If a bad Ansible run leaves hosts misconfigured, re-run the playbook —
   roles should be idempotent and self-correct on the next run.

## 3. Common Issues & Troubleshooting

### Terraform state lock stuck
```bash
terraform force-unlock <LOCK_ID>
```
Only do this after confirming no other apply is genuinely in progress.

### Terraform plan shows unexpected changes (drift)
- Someone likely made a manual change in the console/CLI.
- Run `terraform plan` to see the diff, discuss with the team before
  deciding whether to import the manual change or revert it.

### Ansible playbook fails partway through
```bash
ansible-playbook -i inventories/<env> playbooks/site.yml --limit <failed_host> --start-at-task="<task name>"
```
Use `--check --diff` first to preview changes safely.

### New host not appearing in Ansible inventory
- Confirm Terraform apply completed and outputs are populated.
- Regenerate/refresh the dynamic inventory:
  ```bash
  ansible-inventory -i inventories/<env> --list
  ```

## 4. Access & Secrets

- Cloud credentials: obtained via SSO / assumed role, never long-lived keys
  committed to the repo.
- Ansible Vault password: retrieved from the team's secrets manager, not
  stored on disk.
- Rotate any credential that may have been exposed immediately, and notify
  the ITOps channel.

## 5. On-Call Escalation

| Severity | Response time | Action |
|----------|---------------|--------|
| Sev1 – prod outage | Immediate | Page on-call via PagerDuty, open incident channel |
| Sev2 – degraded service | < 30 min | Notify #itops-infra, begin investigation |
| Sev3 – non-urgent | Next business day | Log as a ticket |

## 6. Post-Incident

1. Restore service first; root-cause after.
2. Write a postmortem within 48 hours (template: link here).
3. File follow-up tickets for any IaC changes needed to prevent recurrence.
4. Update this runbook if the incident revealed a missing procedure.
