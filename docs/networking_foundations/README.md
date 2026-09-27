# Enterprise Network Automation & Infrastructure Governance Master Guide

Welcome to the **Network Automation & Infrastructure Governance** knowledge base. This documentation repository is designed for network engineers and automation specialists leading large-scale multi-platform transformations across 1,000+ managed devices and isolated security enclaves (Citadels).

---

## 📚 Curriculum Structure & Modules

The guides are organized into logical modules under `docs/networking_foundations/`:

| Module | Document | Focus Area |
| :--- | :--- | :--- |
| **01** | [**Device Architectures & CLI Models**](01_device_architectures_and_cli.md) | Architectural deep dive into Cisco IOS-XE/NXOS, Arista EOS, F5 BIG-IP, Palo Alto PAN-OS, Aruba, Ciena, and Opengear. |
| **02** | [**Hands-On CLI Labs**](02_hands_on_cli_labs.md) | Practical command-by-command terminal sessions, manual inspection, configuration, and verification workflows. |
| **03** | [**Automation Mechanics & Quirks**](03_automation_mechanics_and_quirks.md) | Terminal length traps, prompt regex matching, transactional commits, and error handling. |
| **04** | [**Citadel & Isolated Landing Zones**](04_citadel_isolated_zones_architecture.md) | Zero-trust enclave architecture, in-zone runner polling patterns, and cross-zone barrier enforcement. |
| **05** | [**Multi-Cloud Backup Pipelines (S3 & Azure Blob)**](05_multicloud_backup_pipelines.md) | Dual-target backup strategies, AWS S3 Object Lock, Azure Blob Immutability, KMS encryption, and SHA-256 checks. |
| **06** | [**Disaster Recovery & Restore Runbooks**](06_disaster_recovery_and_restore_runbooks.md) | Full step-by-step restoration procedures for F5 UCS, Cisco clean boots, Arista sessions, and Palo Alto XML configs. |
| **07** | [**Threat Model Assessment (TMA)**](07_threat_model_assessment_tma.md) | STRIDE matrix for network CI/CD, least-privilege TACACS+ profiles, and InfoSec approval blueprints. |
| **08** | [**Tower (AWX) to GitLab Runners Migration**](08_ansible_tower_to_gitlab_runners_migration.md) | Transitioning from GUI-based Ansible Tower to Git-driven in-zone containerized GitLab CI/CD runners. |
| **09** | [**NetBox as Single Source of Truth (SSoT)**](09_netbox_ssot_and_dynamic_inventory.md) | Device modeling, IPAM, dynamic inventory plugins, and automated configuration context injection. |

---

## 🚀 Quick Reference: Local Lab Targets

For local testing, the following emulated network targets run in Docker:

- **Cisco IOS-XE / Nexus (SSH):** `ssh -p 2222 cisco@127.0.0.1` (Password: `cisco123`)
- **F5 BIG-IP (tmsh / SSH):** `ssh -p 2223 admin@127.0.0.1` (Password: `admin`)
- **F5 BIG-IP iControl REST:** `http://localhost:8443/mgmt/tm/sys/version`
- **Arista EOS (SSH):** `ssh -p 2221 admin@127.0.0.1` (Password: `admin`)
- **Palo Alto PAN-OS (XML API):** `http://localhost:9443/api/?type=export&category=configuration`
- **NetBox IPAM / SSoT:** `http://localhost:8000` (Token: `U47pFDkAfxbKPGCZZzuK1JHZqjhsKpHoiv3AYWib`)
- **AWX (Ansible Tower):** `http://localhost:8052` (Credentials: `admin` / `adminpassword`)
