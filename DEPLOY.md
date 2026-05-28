# Deployment Guide

This guide explains how to validate and deploy the ISP automation stack across your GNS3 lab.

## Structure

The automation is organized into two playbooks:

### 1. **Infrastructure** (`playbook-infra/`)
Manages MikroTik devices: routers (core, transport, distributors), OLT/ONT fiber terminals and OSPF routing.

**Hosts:** MikroTik RouterOS devices  
**Transport:** SSH with `community.routeros` collection  
**Inventory:** `playbook-infra/inventory/lab.yml`  
**Run:**
```bash
cd playbook-infra
ansible-playbook -i inventory/lab.yml site.yml --syntax-check    # Validate only
ansible-playbook -i inventory/lab.yml site.yml --check --diff   # Dry-run
ansible-playbook -i inventory/lab.yml site.yml                  # Deploy
```

### 2. **Platform Services** (`playbook-platform/`)
Deploys Ubuntu 24.04 service stack: DHCP (Kea), DNS (Bind9), NTP, FreeRADIUS, Zabbix, and the captive portal web interface.

**Hosts:** Ubuntu Linux servers  
**Transport:** SSH with sudo  
**Inventory:** `playbook-platform/inventory/lab.yml`  
**Run:**
```bash
cd playbook-platform
ansible-playbook -i inventory/lab.yml site.yml --syntax-check   # Validate only
ansible-playbook -i inventory/lab.yml site.yml --check         # Dry-run
ansible-playbook -i inventory/lab.yml site.yml                 # Deploy
```

## Network Context

```
Servidores:       10.10.10.0/24     (RADIUS, DNS, Zabbix, DHCP, NTP, Web)
Core:             10.10.20.0/24     (Routers, OSPF loopbacks)
Management:       10.10.30.0/24     (Firewall, switches)
Radius Admin:     10.10.40.0/24     (Mimosa backhaul)
Clientes PPPoE:   100.64.0.0/18     (CGNAT pool, ~16k addresses)
```

## Service Details

| Service | Host | Subnet | Config |
|---------|------|--------|--------|
| DHCP (Kea) | dhcp-01 | 10.10.10.6 | `group_vars/all.yml`: `dhcp_subnet_cidr`, `dhcp_range_start`, `dhcp_range_end` |
| DNS (Bind9) | dns-01 | 10.10.10.7 | `group_vars/all.yml`: `dns_forwarders` |
| NTP | ntp-01 | 10.10.10.9 | `group_vars/all.yml`: `ntp_servers` (Colombian + fallback) |
| RADIUS | radius-01 | 10.10.10.2 | `group_vars/all.yml`: `radius_clients` (networks allowed) |
| Zabbix | zabbix-01 | 10.10.10.4 | Ubuntu 24.04, MariaDB, PHP-FPM via Apache |
| Web Portal | web-01 | 10.10.10.8 | Captive portal index at `/var/www/html/index.html` |

## Typical Workflow

### Phase 1: Validate without touching devices
```bash
# Test playbook syntax
cd playbook-infra && ansible-playbook -i inventory/lab.yml site.yml --syntax-check
cd playbook-platform && ansible-playbook -i inventory/lab.yml site.yml --syntax-check

# Check inventory loads
ansible-inventory -i playbook-infra/inventory/lab.yml --graph
ansible-inventory -i playbook-platform/inventory/lab.yml --graph
```

### Phase 2: Check connectivity before deployment
```bash
# Ping infrastructure hosts
ansible -i playbook-infra/inventory/lab.yml all -m ping

# Ping platform hosts
ansible -i playbook-platform/inventory/lab.yml all -m ping
```

### Phase 3: Dry-run (check mode)
```bash
cd playbook-infra
ansible-playbook -i inventory/lab.yml site.yml --check --diff

cd ../playbook-platform
ansible-playbook -i inventory/lab.yml site.yml --check
```

### Phase 4: Deploy infrastructure first
```bash
cd playbook-infra
ansible-playbook -i inventory/lab.yml site.yml -v
```

### Phase 5: Deploy platform services
```bash
cd playbook-platform
ansible-playbook -i inventory/lab.yml site.yml -v
```

## Prerequisites

### For Infrastructure
```bash
pip install ansible paramiko
ansible-galaxy collection install community.routeros
```

### For Platform
```bash
pip install ansible paramiko
ansible-galaxy collection install ansible.posix
```

## Customization

### Change NTP Servers
Edit `playbook-platform/group_vars/all.yml`:
```yaml
ntp_servers:
  - 0.co.pool.ntp.org
  - 1.south-america.pool.ntp.org
  # Add your custom servers here
```

### Change DHCP Range
Edit `playbook-platform/group_vars/all.yml`:
```yaml
dhcp_subnet_cidr: 10.10.30.0/24
dhcp_range_start: 10.10.30.100
dhcp_range_end: 10.10.30.200
dhcp_router: 10.10.30.1
```

### Change DNS Forwarders
Edit `playbook-platform/group_vars/all.yml`:
```yaml
dns_forwarders:
  - 1.1.1.1
  - 8.8.8.8
```

### Change RADIUS Clients (Allowed Networks)
Edit `playbook-platform/group_vars/all.yml`:
```yaml
radius_clients:
  - name: core
    ip: 10.10.20.0/24
  - name: management
    ip: 10.10.30.0/24
  - name: mimosa-admin
    ip: 10.10.40.0/24
```

## Next Steps

- Add FreeRADIUS user database configuration (`scenarios/radius_users.sql`)
- Configure Zabbix agents on monitored hosts
- Customize the captive portal HTML/CSS
- Set up Zabbix monitoring rules and triggers
- Document OSPF parameters (area 0, router IDs)
