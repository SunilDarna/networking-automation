# Module 07: Threat Model Assessment (TMA) for Network CI/CD

Before automated pipelines are granted access to core routing and firewalls, enterprise InfoSec teams require a formal **Threat Model Assessment (TMA)** to demonstrate that the automation framework will not become a conduit for catastrophic compromise.

---

## 1. STRIDE Threat Analysis for Network Automation

| STRIDE Category | Specific Network Automation Threat | Technical Defense & Mitigation |
| :--- | :--- | :--- |
| **Spoofing** | Rogue runner attempts to impersonate an authorized in-zone runner to push malicious network changes. | Mutual TLS (mTLS) with short-lived X.509 runner tokens. Runner registration pinned to specific subnet CIDRs and hardware UUIDs. |
| **Tampering** | Malicious actor modifies an Ansible playbook in transit or alters a backup stored in S3/Azure Blob. | GPG-signed Git commits on protected branches. Cloud storage configured with WORM Immutability / S3 Object Lock. |
| **Repudiation** | An engineer deploys an unauthorized BGP route and denies executing the job. | Immutable audit logs streamed to central SIEM (Splunk). Job IDs cryptographically linked to Git commit SHAs and approving peer signatures. |
| **Information Disclosure** | Device credentials (e.g. enable secrets, F5 admin passwords) leak into plain text pipeline logs. | HashiCorp Vault dynamic credential leasing. `no_log: true` enforced on all sensitive Ansible tasks. Log masking filters on runners. |
| **Denial of Service** | Concurrent pipeline runs flood switch control planes, causing CPU spikes and OSPF/BGP flapping. | Strict concurrency limits (`serial: "10%"`), rate limiting, and dedicated Out-of-Band (OOB) management VRF isolation. |
| **Elevation of Privilege** | An automated job running under read-only audit permissions escapes into privileged config mode. | TACACS+ command authorization per runner role. AAA command accounting. Service accounts restricted to authorized commands only. |

---

## 2. InfoSec Approval Gate Checklist

To achieve formal sign-off from corporate cybersecurity:
1. **Zero Cross-Zone Management Ingress:** Confirm in-zone runners poll outward over TLS; no open inbound ports across the perimeter.
2. **Mandatory Two-Person Rule:** Protected Git branches require a minimum of two senior peer approvals before merging to `main`.
3. **Automated Pre-Flight Guardrails:** Batfish/yamllint checks run automatically on every Merge Request to detect invalid syntax, open `0.0.0.0/0` routes, or deprecated ciphers before deployment.
