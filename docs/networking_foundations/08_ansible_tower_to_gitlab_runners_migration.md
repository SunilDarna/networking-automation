# Module 08: Ansible Tower (AWX) to GitLab Runners Migration

While Ansible Tower / AWX provides a graphical console and RBAC, enterprise infrastructure governance initiatives are transitioning toward **native, in-zone GitLab CI/CD Runners** to eliminate GUI drift and enable pure GitOps.

---

## 1. Architectural Comparison

| Dimension | Ansible Tower / AWX | GitLab CI/CD In-Zone Runners |
| :--- | :--- | :--- |
| **Execution Plane** | Persistent container cluster (Web, Task, Redis, Postgres) | Ephemeral container spawned per job |
| **Interface** | Web UI and REST API | Git commits, Merge Requests, and `.gitlab-ci.yml` |
| **Runner Overhead** | Heavy (requires full Kubernetes or multi-VM stack) | Extremely lightweight (single Go binary runner) |
| **Citadel Fit** | Difficult to isolate across multiple closed enclaves | Ideal: runner lives inside enclave and polls outward |
| **Secrets Storage** | Stored encrypted in Tower database | Fetched dynamically from HashiCorp Vault at runtime |

---

## 2. Production `.gitlab-ci.yml` Pipeline

```yaml
stages:
  - lint
  - pre-flight-check
  - deploy-citadel
  - backup-audit

variables:
  ANSIBLE_FORCE_COLOR: "true"
  INVENTORY: "inventory/netbox_inventory.yml"

# Stage 1: Static Code Quality & Syntax Linting
lint_playbooks:
  stage: lint
  image: registry.enterprise.local/network/ee-network:latest
  script:
    - ansible-lint playbooks/*.yml
    - yamllint inventory/ playbooks/
  tags:
    - shared-runner

# Stage 2: Dry-Run Validation (Executed by In-Zone Runner)
dry_run_citadel:
  stage: pre-flight-check
  image: registry.enterprise.local/network/ee-network:latest
  script:
    - ansible-playbook -i $INVENTORY playbooks/network_compliance.yml --check --diff
  tags:
    - runner-citadel-zone1

# Stage 3: Real In-Zone Deployment (Requires Approval)
deploy_network:
  stage: deploy-citadel
  image: registry.enterprise.local/network/ee-network:latest
  script:
    - ansible-playbook -i $INVENTORY playbooks/network_compliance.yml
  environment:
    name: production-citadel
  when: manual  # Manual trigger button in GitLab UI
  tags:
    - runner-citadel-zone1
  rules:
    - if: '$CI_COMMIT_BRANCH == "main"'
```
