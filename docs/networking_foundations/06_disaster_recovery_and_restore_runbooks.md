# Module 06: Disaster Recovery (DR) & Configuration Restore Runbooks

Backups are only as good as the verified recovery procedures behind them. This runbook details step-by-step restoration workflows for all primary network platforms.

---

## 1. F5 BIG-IP UCS Restoration

When restoring an F5 BIG-IP from a `.ucs` archive:
- **Case A: Restoring onto the Same Hardware:** Standard restore.
- **Case B: Restoring onto an RMA Replacement Chassis:** The replacement hardware has a different chassis serial number. **You MUST use the `no-license` flag**; otherwise, the restore overwrites the replacement unit's license and breaks system startup!

### Restoration Procedure
```bash
# 1. Transfer UCS archive to device
scp backup_20260925.ucs admin@192.168.10.10:/var/local/ucs/

# 2. SSH into target unit
ssh admin@192.168.10.10

# 3. Execute restore with no-license flag
tmsh load sys ucs backup_20260925.ucs no-license

# 4. Verify system daemons and MCPD state
tmsh show sys mcp-state
tmsh show ltm pool
tmsh show ltm virtual
```

---

## 2. Cisco IOS-XE / NX-OS Clean State Recovery

### Cisco Nexus (NX-OS) Rollback
NX-OS supports native configuration checkpoints:
```text
# Non-disruptive rollback to clean baseline
rollback running-config file bootflash:backup_baseline.cfg
```

### Cisco Catalyst / Router Clean Reload
```text
# Copy pristine baseline to startup-config and reload
copy sftp://backup-srv/cisco-core-01.cfg startup-config
reload
```

---

## 3. Palo Alto Firewall XML State Restore

Because PAN-OS stores configuration as XML DOM trees, disaster recovery uses the XML API:
```bash
# 1. Import pristine configuration XML
curl -k -F "file=@paloalto_backup.xml" \
  "https://192.168.10.30/api/?type=import&category=configuration&key=API_KEY"

# 2. Load imported XML into Candidate Configuration
curl -k "https://192.168.10.30/api/?type=op&cmd=<load><config><from>paloalto_backup.xml</from></config></load>&key=API_KEY"

# 3. Commit Candidate Configuration to hardware data plane
curl -k "https://192.168.10.30/api/?type=commit&cmd=<commit></commit>&key=API_KEY"
```
