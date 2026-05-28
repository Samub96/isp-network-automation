# ISP Platform Services

Playbook para los servicios Linux del lab ISP sobre Ubuntu 24.04.

## Objetivo

Este conjunto de roles prepara la capa de servicios que acompana a la red:

- DHCP autoritativo para clientes y servicios internos.
- DNS con forwarders externos y resolucion de nombres del entorno.
- NTP para sincronizacion de servidores.
- FreeRADIUS para autenticacion PPPoE.
- Zabbix para monitoreo y alertamiento SNMP.
- Portal web como landing o pagina cautiva.

## Servicios y roles

| Host | Rol | Notas |
|------|-----|-------|
| `dhcp-01` | Kea DHCPv4 | Entrega IP, gateway y DNS a la red definida en variables |
| `dns-01` | Bind9 | Resolver local con forwarders externos |
| `ntp-01` | NTP | Sincroniza la hora de la infraestructura |
| `radius-01` | FreeRADIUS | Autenticacion PPPoE para los perfiles de servicio |
| `zabbix-01` | Zabbix | Servidor de monitoreo y frontend |
| `web-01` | Web portal | Landing o portal cautivo |

## Flujo operativo

El `site.yml` ejecuta los roles en este orden:

1. `common_linux` para preparar la base del sistema.
2. `kea_dhcp` para DHCP.
3. `bind9` para DNS.
4. `ntp` para tiempo.
5. `freeradius` para autenticacion PPPoE.
6. `zabbix_server` para monitoreo.
7. `web_portal` para la pagina web.

## Variables clave

Edita [group_vars/all.yml](group_vars/all.yml) para ajustar el comportamiento de los servicios.

- `platform_domain`: dominio local usado por DHCP y DNS.
- `ntp_servers`: servidores NTP de referencia para los Linux del laboratorio.
- `dhcp_subnet_cidr`, `dhcp_range_start`, `dhcp_range_end`, `dhcp_router`: subnet autoritativa del DHCP.
- `dhcp_dns_servers`: DNS entregados por DHCP a los clientes.
- `dns_forwarders`: resolutores externos.
- `radius_clients`: redes autorizadas a consultar FreeRADIUS.
- `radius_secret`: secreto compartido entre los equipos PPPoE y FreeRADIUS.
- `zabbix_version`, `zabbix_timezone`, `zabbix_db_password`: parametros del stack de monitoreo.

## Servicio PPPoE

El playbook esta pensado para un escenario con tres perfiles comerciales:

- 100 megas
- 200 megas
- 500 megas

La documentacion futura debe mapear estos planes a perfiles de FreeRADIUS, politicas QoS o shapers.

## DNS

El resolver debe cubrir dos funciones:

- Resolver nombres del entorno de red y servicios internos.
- Reenviar consultas externas a `8.8.8.8` y `8.8.4.4` por defecto.
- Mantener una zona local configurable con nombres de equipos y servicios en `dns_local_records`.

## DHCP

El servidor DHCP esta planteado como autoritativo para la subred definida en variables. La configuracion actual ya deja listos:

- pool de direcciones;
- gateway por defecto;
- DNS a entregar;
- dominio DHCP.

## Zabbix

El servidor de monitoreo queda listo para recibir:

- agentes Linux;
- chequeos SNMP;
- discovery de servicios;
- futuros templates por fabricante.

## Variables sugeridas para el manual

```yaml
radius_secret: "radiussecret"
dns_forwarders:
- 8.8.8.8
- 8.8.4.4
pppoe_service_profiles:
- name: pppoe-100m
  speed: 100M
- name: pppoe-200m
  speed: 200M
- name: pppoe-500m
  speed: 500M
qos_priority_policies:
  games: 1
  video: 2
  general: 3
```

## Validacion

```bash
cd playbook-platform
ansible-playbook -i inventory/lab.yml site.yml --syntax-check
ansible-playbook -i inventory/lab.yml site.yml --check
```
