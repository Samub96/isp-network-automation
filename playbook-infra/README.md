# ISP Network Automation - Infra

Playbook de infraestructura para equipos de red del lab ISP.

## Objetivo

Este playbook centraliza la configuracion de routers de borde, core y transporte, ademas de OLT y ONT. La orientacion actual es RouterOS para la mayoria de los equipos y un punto de abstraccion para la OLT que permita evolucionar hacia ocNOS sin reescribir la documentacion completa.

## Alcance funcional

- Identidad del dispositivo.
- Direccionamiento e interfaces basicas.
- OSPF para la capa de core y distribucion.
- Provisionamiento basico de OLT y ONT.
- SNMP y sincronizacion NTP como valores de operacion del lab.

## Estructura

```text
playbook-infra/
   site.yml
   inventory/lab.yml
   group_vars/
      all.yml
      core.yml
      distribuidores.yml
      transporte.yml
      olt.yml
      ont.yml
   roles/
      interfaces/
      ospf_config/
      olt_config/
      ont_config/
```

## Inventario logico

| Grupo | Rol | Ejemplos |
|-------|-----|----------|
| `core` | Routers principales | BNG/PPPoE, core primario, core secundario |
| `transporte` | Borde y transporte | Firewall de borde |
| `distribuidores` | Agregacion | Switch de servicios |
| `olt` | Acceso optico | OLT de acceso |
| `ont` | CPE/cliente | ONT individuales |

## Orden de aplicacion

El `site.yml` ejecuta los roles por grupo:

1. `interfaces` y `ospf_config` para `core`, `distribuidores` y `transporte`.
2. `olt_config` para `olt`.
3. `ont_config` para `ont`.

## Variables importantes

### Globales

Edita [group_vars/all.yml](group_vars/all.yml) para cambiar el comportamiento base.

- `ntp_servers`: servidores NTP que deben consumir los equipos de red.
- `snmp_enabled`: activa o desactiva SNMP.
- `snmp_version`: version preferida; el valor actual por defecto es `2c`.
- `snmp_community`: comunidad SNMP de monitoreo.
- `snmp_v3_enabled`: marcador para una futura migracion a SNMPv3.
- `snmp_v3_user`, `snmp_v3_auth_password`, `snmp_v3_priv_password`: credenciales de ejemplo para v3.

### OLT

Edita [group_vars/olt.yml](group_vars/olt.yml) para ajustar la capa de acceso optico.

- `olt_ports`: cantidad de puertos PON o logicos documentados.
- `onu_profiles`: lista de perfiles de ONU.
- `snmp_enabled`: habilitacion de SNMP en la OLT.

### ONT

Edita [group_vars/ont.yml](group_vars/ont.yml) para el comportamiento por cliente.

- `wan_vlan`: VLAN WAN del cliente.
- `management_vlan`: VLAN de administracion.
- `allow_reset`: permite o bloquea el reset desde el equipo.

## SNMP y NTP

La configuracion base queda preparada para:

- SNMPv2c con comunidad variable.
- SNMPv3 como extension futura si el equipo lo soporta.
- Sincronizacion NTP apuntando al servidor definido en `ntp_servers`.

Valores sugeridos para el manual:

```yaml
snmp_enabled: true
snmp_version: "2c"
snmp_community: "monitoring@isp"
snmp_v3_enabled: false
snmp_v3_user: "monitoring"
snmp_v3_auth_password: "password"
snmp_v3_priv_password: "password"
ntp_servers:
   - "10.10.10.9"
```

## Uso

### Instalar dependencias

```bash
pip install ansible paramiko
ansible-galaxy collection install community.routeros
```

### Validar sintaxis

```bash
ansible-playbook -i inventory/lab.yml site.yml --syntax-check
```

### Ejecutar un grupo puntual

```bash
ansible-playbook -i inventory/lab.yml site.yml -l core
ansible-playbook -i inventory/lab.yml site.yml -l olt
```

### Ejecutar sin cambios

```bash
ansible-playbook -i inventory/lab.yml site.yml --check --diff
```

## Recomendaciones operativas

- Usa Vault para secretos reales.
- Mantén los nombres de host sincronizados con el inventario.
- Revisa que el servidor NTP del lab este accesible desde todos los equipos de red.
- Si la OLT pasa a ocNOS, conserva estas variables y cambia solo la capa de comandos del rol.

## Comandos utiles

```bash
ansible -i inventory/lab.yml all -m ping
ansible-inventory -i inventory/lab.yml --list
ansible -i inventory/lab.yml all -m community.routeros.command -a "commands=['/system identity print']"
```
