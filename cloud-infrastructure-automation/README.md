# ITOps Infrastructure-as-Code

Infrastructure-as-Code (IaC) repository for provisioning and configuring
infrastructure using **Terraform** (provisioning) and **Ansible**
(configuration management).

## Overview

This repo defines the desired state of our infrastructure as code so changes
are version-controlled, reviewed, and repeatable. Terraform manages cloud
resources (networking, compute, storage, IAM, etc.); Ansible handles
post-provisioning configuration (packages, users, services, hardening).

## Repository Structure

```
.
├── terraform/
│   ├── modules/           # Reusable Terraform modules
│   ├── environments/
│   │   ├── dev/
│   │   ├── staging/
│   │   └── prod/
│   └── backend.tf         # Remote state configuration
├── ansible/
│   ├── inventories/
│   │   ├── dev/
│   │   ├── staging/
│   │   └── prod/
│   ├── playbooks/
│   ├── roles/
│   └── ansible.cfg
├── docs/
│   ├── ARCHITECTURE.md
│   └── RUNBOOK.md
├── .github/workflows/      # CI/CD pipelines (plan/apply, lint, tests)
└── README.md
```

## Prerequisites

| Tool | Version | Notes |
|------|---------|-------|
| Terraform | >= 1.7 | Use `tfenv` to match `.terraform-version` |
| Ansible | >= 2.16 | `pip install ansible` |
| Cloud CLI | latest | e.g. `aws`, `az`, or `gcloud`, authenticated |
| Git | any recent | |

## Getting Started

1. Clone the repo:
   ```bash
   git clone <repo-url>
   cd itops-infra
   ```
2. Authenticate with your cloud provider (e.g. `aws configure` / `az login`).
3. Initialize Terraform for the target environment:
   ```bash
   cd terraform/environments/dev
   terraform init
   ```
4. Review the plan before applying:
   ```bash
   terraform plan -out=tfplan
   terraform apply tfplan
   ```
5. Run Ansible against the provisioned inventory:
   ```bash
   cd ../../../ansible
   ansible-playbook -i inventories/dev playbooks/site.yml
   ```

## Environments

| Environment | Terraform workspace | Approval required |
|-------------|---------------------|--------------------|
| dev         | `dev`                | No                 |
| staging     | `staging`            | Yes (1 reviewer)   |
| prod        | `prod`               | Yes (2 reviewers)  |

## State Management

Terraform state is stored remotely (e.g. S3 + DynamoDB lock table, or
equivalent). **Never** commit `.tfstate` files or run `terraform apply`
against prod outside the CI/CD pipeline.

## CI/CD

- Pull requests trigger `terraform plan` and `ansible-lint` / `yamllint`.
- Merges to `main` trigger `terraform apply` for `dev` automatically.
- `staging` and `prod` applies require manual approval in the pipeline.

## Contributing

1. Create a feature branch: `git checkout -b feature/short-description`
2. Make changes, run `terraform fmt`, `terraform validate`, `ansible-lint`.
3. Open a PR — the plan output is posted automatically for review.
4. At least one approval required before merge.

## Further Documentation

- [Architecture](docs/ARCHITECTURE.md) — system design and component overview
- [Runbook](docs/RUNBOOK.md) — operational procedures and troubleshooting

## Support / Ownership

| Role | Owner |
|------|-------|
| Repo owner | ITOps Team |
| On-call | See PagerDuty rotation |
| Slack channel | #itops-infra |
