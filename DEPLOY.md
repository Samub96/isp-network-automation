# Deployment Guide

This document is the operational reference for deploying and tuning the ISP automation stack in GNS3 or a similar lab.

## 1. Playbooks

### Infrastructure

Scope:

- RouterOS routers for core, transport and edge roles.
- OLT and ONT provisioning through the infrastructure roles.
- SNMP and time synchronization parameters exposed through variables.

Runbook:

```bash
cd playbook-infra
ansible-playbook -i inventory/lab.yml site.yml --syntax-check
ansible-playbook -i inventory/lab.yml site.yml --check --diff
ansible-playbook -i inventory/lab.yml site.yml
```

### Platform

Scope:

- Kea DHCPv4 for the authoritative client network.
- Bind9 as recursive resolver and forwarder.
- NTP for server-side time sync.
- FreeRADIUS for PPPoE authentication.
- Zabbix as the monitoring stack.
- Web portal for landing or captive portal use.

Runbook:

```bash
cd playbook-platform
ansible-playbook -i inventory/lab.yml site.yml --syntax-check
ansible-playbook -i inventory/lab.yml site.yml --check
ansible-playbook -i inventory/lab.yml site.yml
```

## 2. Suggested execution order

1. Validate inventories.
2. Check syntax on both playbooks.
3. Apply infrastructure.
4. Apply platform services.
5. Verify that NTP, DNS, DHCP, PPPoE and Zabbix are reachable from the network.

```bash
ansible-inventory -i playbook-infra/inventory/lab.yml --graph
ansible-inventory -i playbook-platform/inventory/lab.yml --graph
```

## 3. Network and service map

| Segment | CIDR | Notes |
|---------|------|-------|
| Servers | 10.10.10.0/24 | DNS, NTP, DHCP, RADIUS, Zabbix, Web |
| Core | 10.10.20.0/24 | Routers and loopbacks |
| Management | 10.10.30.0/24 | Borde, acceso y administracion |
| Radius Admin | 10.10.40.0/24 | Backhaul and access management |
| PPPoE clients | 100.64.0.0/18 | Customer address pool |

## 4. Configuration points

### Infrastructure variables

Edit [playbook-infra/group_vars/all.yml](playbook-infra/group_vars/all.yml) for global values:

- `ntp_servers`: NTP target for routers, OLT and ONT.
- `snmp_enabled`: global SNMP toggle.
- `snmp_version`: version to prefer, normally `2c` while v3 is prepared.
- `snmp_community`: default SNMP community, currently `monitoring@isp`.
- `snmp_v3_enabled`: marker for a future migration to SNMPv3.
- `snmp_v3_user`, `snmp_v3_auth_password`, `snmp_v3_priv_password`: example credentials for v3.

Edit [playbook-infra/group_vars/olt.yml](playbook-infra/group_vars/olt.yml) for OLT specific values:

- `olt_ports`: number of PON or logical ports documented in the lab.
- `onu_profiles`: list of ONU service profiles.
- `olt_vendor`: backend hint for the manual, currently documented for ocNOS target usage.

### Platform variables

Edit [playbook-platform/group_vars/all.yml](playbook-platform/group_vars/all.yml) for service behavior:

- `platform_domain`: local DNS domain used by DHCP and service naming.
- `ntp_servers`: upstream NTP sources for the Linux servers.
- `dhcp_subnet_cidr`, `dhcp_range_start`, `dhcp_range_end`, `dhcp_router`: Kea authoritative scope.
- `dhcp_dns_servers`: DNS servers offered to clients.
- `dns_zone_name` and `dns_local_records`: internal zone for equipment and service names.
- `dns_forwarders`: external resolvers.
- `radius_clients`: CIDRs allowed to talk to FreeRADIUS.
- `radius_secret`: shared secret for RADIUS clients.
- `pppoe_service_profiles`: intended tiers for 100, 200 and 500 megas.
- `qos_priority_policies`: future LibreQoS or shaping priority map for games and video.
- `zabbix_version`, `zabbix_timezone`, `zabbix_db_password`: monitoring stack controls.

## 5. Recommended defaults

### SNMP

Use SNMPv2c as a simple baseline and keep the community in a variable:

```yaml
snmp_enabled: true
snmp_version: "2c"
snmp_community: "monitoring@isp"
snmp_v3_enabled: false
snmp_v3_user: "monitoring"
snmp_v3_auth_password: "password"
snmp_v3_priv_password: "password"
```

### DNS forwarders

Point the resolver to public forwarders that are easy to replace later:

```yaml
dns_forwarders:
  - 8.8.8.8
  - 8.8.4.4
```

### PPPoE service tiers

The intended commercial catalog is:

- 100 megas
- 200 megas
- 500 megas

Those tiers should later map to RADIUS profiles, shapers or QoS policies.

## 6. Post-deploy checks

```bash
ansible -i playbook-infra/inventory/lab.yml all -m ping
ansible -i playbook-platform/inventory/lab.yml all -m ping
ansible -i playbook-platform/inventory/lab.yml zabbix -m service -a "name=zabbix-server state=started"
```

## 7. Notes for the future manual

- Document local DNS records for network equipment and services.
- Add the exact SNMPv3 flow when the OLT or routers are ready for it.
- Expand the FreeRADIUS user and profile database.
- Formalize the LibreQoS policy tree for gaming and video priority.
- Add Zabbix templates, triggers and discovery rules per vendor.
