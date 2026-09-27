# Module 05: Multi-Cloud Backup Pipelines (AWS S3 & Azure Blob)

A mission-critical network automation framework must guarantee that all 1,000+ device configurations are backed up daily, cryptographically verified, and stored in tamper-proof, immutable cloud storage.

---

## 1. Storage Target Architecture: Dual-Cloud Strategy

| Cloud Feature | Amazon Web Services (AWS S3) | Microsoft Azure (Blob Storage) |
| :--- | :--- | :--- |
| **Immutability (WORM)** | S3 Object Lock (Compliance Mode) | Azure Immutable Blob (Time-Based Policy) |
| **Encryption** | AWS KMS Customer Managed Keys (SSE-KMS) | Azure Key Vault Customer Managed Keys (CMK) |
| **Authentication** | IAM Roles for Service Accounts (IRSA) / OIDC | Azure Managed Identity / Workload Identity |
| **Lifecycle Tiers** | S3 Standard → S3 Glacier Flexible (90d) | Hot Tier → Cool Tier (30d) → Archive (90d) |

---

## 2. Integrity & Cryptographic Validation

Before uploading any backup file to the cloud, the runner must compute and record a **SHA-256 checksum**:
1. It creates the backup archive locally on the runner.
2. It computes `sha256sum backup_file.ucs`.
3. It uploads the backup file to S3/Azure Blob with the SHA-256 hash attached as object metadata.
4. During any disaster recovery operation, the download script recalculates the SHA-256 hash to ensure the backup was not corrupted or tampered with in transit.

---

## 3. Production Ansible Multi-Cloud Backup Playbook

```yaml
---
- name: Multi-Vendor Network Backup & Cloud Storage Upload
  hosts: all_network
  gather_facts: no
  vars:
    cloud_storage_target: "aws"  # Set to "aws" or "azure"
    backup_timestamp: "{{ lookup('pipe', 'date +%Y%m%d_%H%M%S') }}"
    local_archive_dir: "/tmp/network_backups"

  tasks:
    - name: Ensure Local Staging Directory Exists
      ansible.builtin.file:
        path: "{{ local_archive_dir }}"
        state: directory
        mode: '0755'

    - name: Calculate Cryptographic SHA-256 Checksum
      ansible.builtin.stat:
        path: "{{ backup_file_path }}"
        checksum_algorithm: sha256
      register: backup_stat

    - name: Upload to AWS S3 with Object Lock & KMS Encryption
      when: cloud_storage_target == "aws"
      amazon.aws.s3_object:
        bucket: "enterprise-network-backups-immutable"
        object: "{{ inventory_hostname }}/{{ backup_file_name }}"
        src: "{{ backup_file_path }}"
        mode: put
        encrypt: yes
        encryption_kms_key_id: "arn:aws:kms:us-east-1:123456789012:key/network-backup-key"
        metadata:
          sha256: "{{ backup_stat.stat.checksum }}"
          device: "{{ inventory_hostname }}"

    - name: Upload to Azure Blob Storage with Immutability Policy
      when: cloud_storage_target == "azure"
      azure.azcollection.azure_rm_storageblob:
        resource_group: "rg-network-governance-prod"
        storage_account_name: "stnetbackupimmutprod"
        container_name: "device-backups"
        blob: "{{ inventory_hostname }}/{{ backup_file_name }}"
        src: "{{ backup_file_path }}"
        metadata:
          sha256: "{{ backup_stat.stat.checksum }}"
          device: "{{ inventory_hostname }}"
```
