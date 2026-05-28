# ISP Network Automation - GNS3 Lab

Complete infrastructure and services automation for a simulated ISP network using Ansible, MikroTik, and Ubuntu 24.04.

## Structure

```
.
├── README.md                # This file
├── DEPLOY.md               # Deployment guide and customization reference
├── playbook-infra/         # MikroTik infrastructure (routers, OLT/ONT)
│   ├── site.yml
│   ├── inventory/lab.yml
│   ├── group_vars/
│   ├── roles/
│   │   ├── interfaces/
│   │   ├── ospf_config/
│   │   ├── olt_config/
│   │   └── ont_config/
│   └── README.md
└── playbook-platform/      # Ubuntu 24.04 services (DHCP, DNS, NTP, Zabbix, etc.)
    ├── site.yml
    ├── inventory/lab.yml
    ├── group_vars/
    ├── roles/
    │   ├── common_linux/
    │   ├── kea_dhcp/
    │   ├── bind9/
    │   ├── ntp/
    │   ├── freeradius/
    │   ├── zabbix_server/
    │   └── web_portal/
    └── README.md
```

## Quick Validation

Validate both playbooks without applying any changes:

```bash
# Validate syntax
cd playbook-infra && ansible-playbook -i inventory/lab.yml site.yml --syntax-check
cd ../playbook-platform && ansible-playbook -i inventory/lab.yml site.yml --syntax-check

# Verify inventory parsing
ansible-inventory -i playbook-infra/inventory/lab.yml --graph
ansible-inventory -i playbook-platform/inventory/lab.yml --graph
```

## Network Topology

| Zone | CIDR | Usage |
|------|------|-------|
| **Servidores** | 10.10.10.0/24 | RADIUS, DNS, Zabbix, DHCP, NTP |
| **Core** | 10.10.20.0/24 | Routers, OSPF loopbacks |
| **Management** | 10.10.30.0/24 | Firewall, switches, edge equipment |
| **Radius Admin** | 10.10.40.0/24 | Mimosa C5x backhaul management |
| **Clientes PPPoE** | 100.64.0.0/18 | 16,382 IPs for CGNAT pool |

## Deployment Workflow

1. **Validate infrastructure playbook:**
   ```bash
   cd playbook-infra && ansible-playbook -i inventory/lab.yml site.yml --syntax-check
   ```

2. **Validate platform playbook:**
   ```bash
   cd playbook-platform && ansible-playbook -i inventory/lab.yml site.yml --syntax-check
   ```

3. **Deploy infrastructure (runs in GNS3 against MikroTik device IPs defined in inventory):**
   ```bash
   cd playbook-infra && ansible-playbook -i inventory/lab.yml site.yml -vv
   ```

4. **Deploy platform services (runs in GNS3 against Ubuntu server IPs defined in inventory):**
   ```bash
   cd playbook-platform && ansible-playbook -i inventory/lab.yml site.yml -vv
   ```

See [DEPLOY.md](DEPLOY.md) for detailed deployment guide, customization options, and troubleshooting.

## Service Summary

| Service | Container | Version | Config | Port |
|---------|-----------|---------|--------|------|
| **DHCP** | kea-01 | Kea 2.x | Kea DHCPv4 | 67/udp |
| **DNS** | dns-01 | Bind9 | Recursive forwarder | 53/udp |
| **NTP** | ntp-01 | ntpd | Colombian + global pools | 123/udp |
| **RADIUS** | radius-01 | FreeRADIUS 3 | PPPoE auth | 1812/udp |
| **Zabbix** | zabbix-01 | 7.0 + MariaDB | Monitoring | 10051 TCP, 10050 UDP |
| **Web** | web-01 | Apache2 | Captive portal | 80/tcp |

## Prerequisites

### Ansible Setup
```bash
pip install ansible paramiko
ansible-galaxy collection install community.routeros ansible.posix
```

### GNS3 Lab Requirements
- MikroTik RouterOS devices with SSH enabled
- Ubuntu 24.04 (Noble) VMs with SSH and sudo configured
- Network connectivity between all hosts and your Ansible controller
- IPs matching those defined in `inventory/lab.yml` files

## Key Files

- **[playbook-infra/README.md](playbook-infra/README.md)** – Infrastructure deployment details
- **[playbook-platform/README.md](playbook-platform/README.md)** – Platform services deployment details
- **[DEPLOY.md](DEPLOY.md)** – Full deployment and customization guide
- **[playbook-infra/group_vars/all.yml](playbook-infra/group_vars/all.yml)** – Network and infrastructure variables
- **[playbook-platform/group_vars/all.yml](playbook-platform/group_vars/all.yml)** – Service and platform variables

## License & Notes

This project automates a complete ISP lab for GNS3. All scripts and configurations are designed for testing and internal lab use only.
