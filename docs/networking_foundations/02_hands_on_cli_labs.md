# Module 02: Hands-On Device CLI Labs

This module provides reproducible, step-by-step terminal sessions for connecting to, inspecting, configuring, and backing up our live emulated network devices.

---

## Lab 1: Cisco IOS-XE Core Router & Switch

### 1.1 Connect via SSH
```bash
ssh -p 2222 -o StrictHostKeyChecking=no cisco@127.0.0.1
```
- **Password:** `cisco123`
- **Expected Prompt:** `cisco-core-rtr01#`

### 1.2 Inspection Commands
```text
show version
show ip interface brief
show running-config
```

### 1.3 Configuration Tasks
```text
configure terminal
vlan 500
 name AUTOMATION_LAB
exit
ntp server 10.0.0.1 prefer
ntp server 10.0.0.2
exit
write memory
```

### 1.4 Verification
```text
show running-config | include ntp
```

---

## Lab 2: F5 BIG-IP Application Delivery Controller (LTM)

### 2.1 Connect via TMSH (SSH)
```bash
ssh -p 2223 -o StrictHostKeyChecking=no admin@127.0.0.1
```
- **Password:** `admin`
- **Expected Prompt:** `admin@(f5-bigip-ltm01)(cfg-sync Standalone)(Active)(/Common)(tmos)# `

### 2.2 Inspection Commands
```text
show sys version
show ltm virtual
show ltm pool
```

### 2.3 Manual UCS Backup Generation
```text
save sys ucs /var/local/ucs/manual_backup.ucs
```
*Output:*
```text
Saving active configuration...
/var/local/ucs/manual_backup.ucs is saved.
```

### 2.4 Modern Automation: iControl REST API
Run these commands from your local Mac terminal:
```bash
# Query BIG-IP Version
curl -s http://localhost:8443/mgmt/tm/sys/version | python3 -m json.tool

# Query Virtual Servers
curl -s http://localhost:8443/mgmt/tm/ltm/virtual | python3 -m json.tool

# Create a UCS Backup via API
curl -s -X POST http://localhost:8443/mgmt/tm/sys/ucs \
  -H "Content-Type: application/json" \
  -d '{"command": "save", "name": "rest_backup.ucs"}' | python3 -m json.tool
```

---

## Lab 3: Arista EOS Leaf-Spine Switch

### 3.1 Connect via SSH
```bash
ssh -p 2221 -o StrictHostKeyChecking=no admin@127.0.0.1
```
- **Password:** `admin`
- **Expected Prompt:** `arista-leaf01#`

### 3.2 Inspection Commands
```text
show version
show running-config
show ip route
```

### 3.3 Two-Phase Commit Session Lab
```text
configure session lab_change_01
ntp server 10.0.0.1 prefer
show session-config diff
commit
exit
```

---

## Lab 4: Palo Alto Networks Firewall (PAN-OS)

### 4.1 Connect via SSH
```bash
ssh -p 2224 -o StrictHostKeyChecking=no admin@127.0.0.1
```
- **Password:** `admin`
- **Expected Prompt:** `admin@paloalto-fw01> `

### 4.2 Inspection Commands
```text
show system info
show config running
```

### 4.3 Export Running XML Configuration via API
Run from your Mac terminal:
```bash
curl -s "http://localhost:9443/api/?type=export&category=configuration" | head -n 35
```
