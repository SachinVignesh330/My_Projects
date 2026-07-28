# Architecture

## Purpose

This document describes how the infrastructure defined in this repository
fits together: the cloud resources Terraform provisions, the configuration
Ansible applies on top, and how the two hand off to each other.

## High-Level Diagram

```
                ┌─────────────────────────┐
                │        Git Repo          │
                │  terraform/  ansible/    │
                └────────────┬─────────────┘
                             │ CI/CD pipeline
                             ▼
                ┌─────────────────────────┐
                │   Terraform (provision)  │
                │  VPC, subnets, compute,  │
                │  storage, IAM, DNS       │
                └────────────┬─────────────┘
                             │ dynamic inventory
                             ▼
                ┌─────────────────────────┐
                │   Ansible (configure)    │
                │  OS hardening, packages, │
                │  services, users, agents │
                └────────────┬─────────────┘
                             ▼
                ┌─────────────────────────┐
                │   Running Infrastructure │
                │  (dev / staging / prod)  │
                └─────────────────────────┘
```

## Components

### Terraform Layer

| Layer | Responsibility |
|-------|-----------------|
| `terraform/modules/` | Reusable, versioned building blocks (network, compute, database, IAM) |
| `terraform/environments/<env>/` | Environment-specific composition of modules and variable values |
| Remote backend | Stores `.tfstate` centrally; provides locking to prevent concurrent applies |

Design principles:
- **Modules are environment-agnostic** — all environment differences live in
  `environments/<env>/*.tfvars`, not in the modules themselves.
- **One state file per environment** to limit blast radius of a bad apply.
- **Least-privilege IAM** — each environment's CI role can only act on its
  own environment's resources.

### Ansible Layer

| Layer | Responsibility |
|-------|-----------------|
| `inventories/<env>/` | Host groupings per environment, often generated dynamically from Terraform outputs or cloud APIs |
| `roles/` | Single-purpose, reusable configuration units (e.g. `common`, `monitoring-agent`, `webserver`) |
| `playbooks/` | Compose roles into a run for a given host group (e.g. `site.yml`) |

Design principles:
- Roles are idempotent — safe to re-run without changing already-correct state.
- Secrets are pulled from a vault/secrets manager at runtime, never committed.
- Playbooks are tagged so partial runs (e.g. `--tags monitoring`) are possible.

### Handoff Between Terraform and Ansible

Terraform outputs (IP addresses, hostnames, resource IDs) feed the Ansible
inventory, either via:
- a dynamic inventory script/plugin reading Terraform state, or
- a generated static inventory file written by the CI pipeline after `apply`.

## Environments

Each environment (`dev`, `staging`, `prod`) is fully isolated:
- Separate Terraform state and workspace
- Separate cloud account/subscription or strict resource tagging boundary
- Separate Ansible inventory and vault-encrypted secrets

## CI/CD Flow

1. PR opened → `terraform plan` (all envs affected) + lint checks run, plan
   posted as a PR comment.
2. Merge to `main` → auto-apply to `dev`, Ansible run against `dev` inventory.
3. Promotion to `staging`/`prod` → manual approval gate, then apply + configure.

## Security Considerations

- State files and vault-encrypted secrets are never stored in plaintext in git.
- IAM roles used by CI are scoped per-environment and reviewed quarterly.
- All changes are peer-reviewed via PR before reaching staging/prod.

## Open Questions / Future Work

> Fill in as the architecture evolves, e.g.:
> - Multi-region failover strategy
> - Secrets manager migration
> - Policy-as-code (e.g. OPA/Sentinel) for Terraform guardrails
