# ISP Network Automation - GNS3 Lab

Repositorio de automatizacion para un laboratorio ISP en GNS3. El objetivo es dejar una base util para provisionar equipos de red y servicios de plataforma con Ansible, usando RouterOS en la capa de routing y un punto de abstraccion para la OLT que permita evolucionar hacia ocNOS sin rehacer la documentacion.

## Alcance

Este repo separa dos planos de operacion:

- `playbook-infra/` para equipos de red: routers core, edge, transporte, OLT y ONT.
- `playbook-platform/` para servicios Linux: DHCP, DNS, NTP, FreeRADIUS, Zabbix y portal web.

La idea es que este material sirva como base de un manual operativo, por lo que la configuracion importante esta expuesta como variables y agrupada por dominio de responsabilidad.

## Topologia Logica

| Zona | CIDR | Uso |
|------|------|-----|
| Servidores | 10.10.10.0/24 | DHCP, DNS, NTP, RADIUS, Zabbix y web |
| Core | 10.10.20.0/24 | Routers, loopbacks y BNG/PPPoE |
| Management | 10.10.30.0/24 | Borde, acceso de administracion y equipos de transporte |
| Radius Admin | 10.10.40.0/24 | Gestion de backhaul y acceso remoto de equipos |
| Clientes PPPoE | 100.64.0.0/18 | Pool de clientes / CGNAT |

## Flujo de provisionamiento

1. Se valida primero la sintaxis de ambos playbooks.
2. Luego se despliegan los equipos de infraestructura.
3. Finalmente se aplican los servicios de plataforma.

```bash
cd playbook-infra && ansible-playbook -i inventory/lab.yml site.yml --syntax-check
cd ../playbook-platform && ansible-playbook -i inventory/lab.yml site.yml --syntax-check
```

```bash
cd playbook-infra && ansible-playbook -i inventory/lab.yml site.yml -vv
cd ../playbook-platform && ansible-playbook -i inventory/lab.yml site.yml -vv
```

## Servicios de plataforma

| Servicio | Host | Rol |
|----------|------|-----|
| DHCP | `dhcp-01` | Kea DHCPv4 autoritativo para la red de servicios |
| DNS | `dns-01` | Bind9 como resolvedor y forwarder |
| NTP | `ntp-01` | Servidor de hora para los equipos de red y servidores |
| FreeRADIUS | `radius-01` | Autenticacion PPPoE |
| Zabbix | `zabbix-01` | Monitoreo y alertas SNMP |
| Web | `web-01` | Portal cautivo / landing de acceso |

## Variables clave

Las variables que mas interesan para el manual estan en estos archivos:

- [playbook-infra/group_vars/all.yml](playbook-infra/group_vars/all.yml)
- [playbook-infra/group_vars/olt.yml](playbook-infra/group_vars/olt.yml)
- [playbook-platform/group_vars/all.yml](playbook-platform/group_vars/all.yml)

Resumen de parametros que conviene revisar antes de pasar a produccion o a un entorno mas cercano al real:

- SNMP: comunidad, version y credenciales v3 si se habilitan.
- NTP: lista de servidores a los que deben apuntar routers, OLT y ONT.
- DNS: forwarders externos y, si se desea, zonas locales para equipos y servicios.
- FreeRADIUS: secreto compartido y perfiles PPPoE de 100, 200 y 500 megas.
- DHCP: rango autoritativo, router por defecto y DNS entregado a clientes.
- Zabbix: version y timezone.

## Requisitos

```bash
pip install ansible paramiko
ansible-galaxy collection install community.routeros ansible.posix
```

- MikroTik RouterOS con SSH habilitado.
- OLT con el backend definido para el playbook de infraestructura.
- Ubuntu 24.04 para los servidores de plataforma.
- Conectividad IP entre el controlador Ansible y todos los hosts.

## Referencia rapida

- [Guia de despliegue](DEPLOY.md)
- [Playbook de infraestructura](playbook-infra/README.md)
- [Playbook de plataforma](playbook-platform/README.md)

## Nota

El repo esta pensado como una base de automatizacion evolutiva: hoy documenta y provisiona el lab, y mas adelante puede ampliarse con plantillas, validaciones, roles adicionales y un manual de operacion completo.
