# Module 01: Multi-Vendor Device Architectures & CLI Models

Enterprise network environments are rarely homogeneous. A robust automation framework must understand the underlying operating system paradigms, memory models, and CLI structures of diverse vendors.

---

## 1. Cisco Systems (IOS-XE & NX-OS)

### 1.1 Architecture & Operating Modes
- **Cisco IOS-XE:** Runs a Linux-based kernel (typically CentOS/Yocto under the hood) hosting the IOS daemon as a control-plane process.
- **Cisco NX-OS:** Purpose-built for datacenter switches (Nexus series), featuring modular, fault-tolerant process restartability and native XML/JSON APIs (NX-API).

### 1.2 CLI Hierarchy
1. **User EXEC (`>`)**: Limited inspection (`ping`, `traceroute`, `show version`).
2. **Privileged EXEC (`#`)**: Activated via `enable`. Allows full diagnostic views, running config dumps, and debug commands.
3. **Global Configuration (`(config)#`)**: Activated via `configure terminal`.

### 1.3 Memory Model (Running vs Startup)
Cisco uses **direct command execution**:
- Typing `vlan 100` in `(config)#` immediately updates the active data plane (`running-config` in RAM).
- Changes must be explicitly committed to non-volatile storage (`startup-config` in NVRAM) via `write memory` or `copy running-config startup-config`.

---

## 2. Arista Networks (EOS)

### 2.1 Architecture
- **Unmodified Linux Base:** EOS runs directly on Fedora Linux. Administrators can type `bash` from privileged EXEC to drop into a standard Linux shell.
- **Sysdb Central State Engine:** All switch state (routing tables, interface counters, MAC tables) is stored in an in-memory database (`Sysdb`), completely separating state from control plane protocol agents.

### 2.2 Atomic Configuration Sessions
Arista does not mandate immediate configuration application. Instead, it supports **transactional configuration sessions**:
```text
arista-leaf01# configure session change_vlan
arista-leaf01(config-s-change_vlan)# vlan 100
arista-leaf01(config-s-change_vlan)# show session-config diff
+ vlan 100
arista-leaf01(config-s-change_vlan)# commit
```
If an error occurs, the engineer or automated playbook runs `abort`, leaving production traffic untouched.

### 2.3 Programmability (eAPI)
EOS exposes **eAPI**, a JSON-RPC interface listening on ports 80/443. Commands like `show version` return structured JSON objects instead of unstructured raw text.

---

## 3. F5 Networks BIG-IP (TMOS)

### 3.1 Architecture
- **TMOS (Traffic Management Operating System):** Real-time kernel specialized for Layer 4–Layer 7 application delivery, SSL acceleration, and packet inspection.
- **TMM (Traffic Management Microkernel):** Directly binds to physical network interfaces, bypassing the Linux control plane for maximum throughput.

### 3.2 Management Interfaces
- **TMSH (Traffic Management Shell):** Hierarchical CLI for F5 administrators:
  ```text
  admin@(f5-bigip)(Active)(/Common)(tmos)# show sys version
  admin@(f5-bigip)(Active)(/Common)(tmos)# list ltm virtual
  ```
- **iControl REST API:** RESTful JSON endpoints listening on HTTPS 443/8443 (`/mgmt/tm/...`).

### 3.3 Administrative Partitions & UCS Backups
- **Partitions:** Logical silos (e.g. `/Common`, `/Production`) that isolate security policies and virtual servers.
- **UCS (User Configuration Set):** A comprehensive backup tarball containing:
  - Big-IP base configurations (`bigip_base.conf`)
  - LTM/AFM/APM objects (`bigip.conf`)
  - Private SSL keys & X.509 certificates
  - Hardware licenses and user credentials

---

## 4. Palo Alto Networks (PAN-OS & Panorama)

### 4.1 Architecture
- **Single-Pass Parallel Processing (SP3):** Hardware architecture that performs networking, threat identification, and policy lookup in a single pass.
- **Panorama:** Central management platform orchestrating configuration templates and device groups across thousands of firewalls.

### 4.2 Candidate vs Running Configuration
Modifying rules or address objects edits only the **Candidate Configuration** in RAM. The firewall continues enforcing the active **Running Configuration** until a two-phase commit is performed:
```text
admin@pa-fw01# set address web-srv ip-netmask 10.0.0.10/32
admin@pa-fw01# commit
```

### 4.3 XML Architecture
Under the hood, PAN-OS stores its entire running and candidate configurations as XML DOM trees. Automation can query, export, and overwrite sections of the configuration using the XML API (`/api/?type=export&category=configuration`).

---

## 5. Other Enterprise Platforms in Scope

| Vendor / Platform | Device Types | Management Protocol | Automation Role |
| :--- | :--- | :--- | :--- |
| **Aruba (AOS-CX)** | Campus Core & Access Switches | REST API v10.04+ & SSH | Campus VLAN and 802.1X policy enforcement |
| **Ciena** | Carrier Ethernet & NIDs | SSH / TL1 | WAN demarcation, MPLS handoffs |
| **Extreme Networks** | ExtremeXOS / VSP Switches | SSH / Python CLI | Datacenter and enterprise switching |
| **Opengear** | Console Server Managers | SSH (Port 22) | Out-of-band serial recovery (Day 0 console access) |
| **APC / Raritan** | Switched Smart PDUs | SNMPv3 / SSH | Rack power cycling and environmental telemetry |
