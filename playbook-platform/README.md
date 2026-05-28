# ISP Platform Services

This playbook deploys the Linux services for the ISP lab on Ubuntu 24.04 (Noble).

## Service layout

- `dhcp-01` runs Kea DHCPv4.
- `dns-01` runs Bind9.
- `ntp-01` runs the NTP daemon with the regional pool servers.
- `radius-01` runs FreeRADIUS.
- `zabbix-01` runs the Zabbix server stack.
- `web-01` runs the captive portal web stack.

## Run

```bash
cd playbook-platform
ansible-playbook -i inventory/lab.yml site.yml --syntax-check
```
