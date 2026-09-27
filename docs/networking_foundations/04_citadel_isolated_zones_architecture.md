# Module 04: Citadel & Isolated Landing Zones Architecture

Enterprise environments frequently segregate critical systems into isolated network enclaves known as **Citadels** or **Restricted Landing Zones** (e.g., PCI-DSS cardholder environments, core datacenter fabrics, SWIFT financial networks).

---

## 1. The Core Architectural Challenge

Traditional automation architectures use a centralized orchestrator (e.g., central AWX or GitLab instance) that attempts to SSH directly into every network device.

### Why Central Ingress Fails Security Reviews
1. **Perimeter Firewall Violations:** Connecting into a Citadel requires opening hundreds of inbound SSH (port 22) and API (port 443) firewall rules from the corporate network into the restricted zone.
2. **Single Point of Compromise:** If the central automation server is breached, the attacker gains direct management credentials to all 1,000+ devices across all security zones simultaneously.
3. **Regulatory Non-Compliance:** Regulations like PCI-DSS and FedRAMP forbid shared management planes that bridge isolated zones.

---

## 2. The Solution: In-Zone Runner Architecture

Instead of reaching *into* the Citadel, we deploy a **lightweight, containerized execution runner directly inside the Citadel**:

```
[ Central Control Plane ]
  - GitLab SCM
  - NetBox IPAM
  - HashiCorp Vault
        │
        ▲ (HTTPS / Port 443 Outbound Only)
        │
┌───────┴───────────────────────────────────────┐
│ CITADEL ISOLATED ENCLAVE                      │
│                                               │
│  [ In-Zone GitLab Runner ]                    │
│        │                                      │
│        ├── Local SSH (Port 22)                │
│        ▼                                      │
│  [ Core Routers / Firewalls / Load Balancers] │
└───────────────────────────────────────────────┘
```

### Key Principles of the In-Zone Runner Pattern
1. **Zero Cross-Zone Inbound Ports:** The Citadel firewall has **zero inbound rules** open for management.
2. **Outbound Polling over TLS:** The in-zone runner initiates an outbound HTTPS (port 443) long-poll connection to the central GitLab or AWX server to check for queued jobs.
3. **Local Execution Context:** Once assigned a task, the runner pulls the approved Git commit and executes the Ansible playbook locally against devices in its own subnet.
4. **Isolated Credentials:** The runner only possesses TACACS+ or local credentials valid for its specific Citadel zone.
