# Module 09: NetBox as Single Source of Truth (SSoT) & Dynamic Inventory

NetBox eliminates static spreadsheets and serves as the authoritative source for IPAM, device rack locations, cabling, and automation variables.

---

## 1. The Single Source of Truth Principle

In a GitOps network governance framework:
- If an engineer needs to add a VLAN or re-address a switch interface, they **first update NetBox**.
- The automation pipeline queries NetBox dynamically, detects the intended state, and pushes the configuration to the physical devices.
- Manual changes performed directly on the switch CLI without updating NetBox are flagged as **configuration drift** during scheduled audit runs.

---

## 2. Dynamic Inventory Integration (`nb_inventory`)

Instead of maintaining static IP lists in `hosts.yml`, Ansible queries NetBox's REST API at runtime using the `netbox.netbox.nb_inventory` plugin:

```yaml
plugin: netbox.netbox.nb_inventory
api_endpoint: "http://host.k3d.internal:8000"
token: "U47pFDkAfxbKPGCZZzuK1JHZqjhsKpHoiv3AYWib"
validate_certs: false
config_context: true

# Automatically group devices by NetBox metadata
group_by:
  - sites
  - roles
  - manufacturers
  - device_types

# Construct hostvars dynamically
compose:
  ansible_host: primary_ip4.address | default(name) | regex_replace('/.*', '')
  ansible_network_os: >-
    {% if 'cisco' in device_type.manufacturer.slug %}
      cisco.ios.ios
    {% elif 'arista' in device_type.manufacturer.slug %}
      arista.eos.eos
    {% elif 'f5' in device_type.manufacturer.slug %}
      f5networks.f5_modules.bigip
    {% else %}
      generic
    {% endif %}
```

---

## 3. Config Contexts for Data Inheritance

NetBox **Config Contexts** allow you to assign hierarchical JSON data to devices based on their site, platform, or role:
- **Global Context:** Defines corporate DNS (`1.1.1.1`, `8.8.8.8`) and NTP servers (`10.0.0.1`, `10.0.0.2`).
- **Site Context:** Defines local syslog receivers and default gateway subnets.
- **Device Context:** Defines local BGP AS numbers and interface IP allocations.

Ansible automatically merges these contexts into `hostvars`, allowing playbooks to use clean, declarative Jinja2 templates without hardcoded values.
