# ISP Network Automation - GNS3 Lab

Automatizaci�n de configuraci�n para red ISP simulada en GNS3 usando Ansible.

##  Topolog�a de Red

| ID | Dispositivo | Función / Rol | Dirección IP (Gestión) | Observaciones de Configuración |
| --- | --- | --- | --- | --- |
| SRV-01 | Radius | Autenticación PPPoE | 10.10.10.2 | Base de datos de usuarios y perfiles. |
| SRV-02 | Ansible | Automatización (Provisionamiento) | 10.10.10.3 | Gestión de configuraciones mediante Playbooks. |
| SRV-03 | Zabbix | Observabilidad (Monitoreo) | 10.10.10.4 | Dashboard de estado y alertas SNMP. |
| SRV-04 | LibreQoS | Calidad de Servicio (Shaping) | 10.10.10.5 | Control de ancho de banda y latencia. |
| SRV-05a | DHCP | Servicios básicos de red | 10.10.10.6 | Soporte de hosting interno. |
| SRV-05b | DNS | Servicios básicos de red | 10.10.10.7 | Soporte de resolución interno. |
| SRV-05c | WEB | Servicios básicos de red | 10.10.10.8 | Soporte de resolución y hosting interno. |
| SRV-05d | NTP | Servicios básicos de red | 10.10.10.9 | Soporte de sincronización de hora. |
| RTR-01 | FW-EDGE-1 | Firewall de Borde | 10.10.30.1 | Seguridad perimetral y salida a Internet (NAT). |
| RTR-02 | RTR-EDGE-1 | BNG / PPPoE Server | 10.10.20.1 | Concentrador de túneles y ruteo dinámico. |
| RTR-03 | CR-CORE-1 | Core Router Principal | 10.10.20.2 | Nodo OSPF - Distribución de tráfico. |
| RTR-04 | CR-CORE-2 | Core Router Secundario | 10.10.20.3 | Redundancia de Core (Malla completa). |
| SW-01 | SW-SRV-1 | Switch de Agregación | 10.10.10.10 | Conexión de granja de servidores (VLAN 10). |
| OLT-01 | OLT-ACC-1 | Acceso Fibra Óptica | 10.10.40.10 | Gestión de ONTs (client3 y client4). |
| RAD-01 | MIMOSA C5x | Backhaul Inalámbrico | 10.10.40.20 | Enlace punto a punto entre torres. |
| AP-01 | AP-ACC-1 | Acceso Inalámbrico | 10.10.40.30 | Sectorial para clientes inalámbricos. |

La automatización de este repositorio se centra en los equipos de red MikroTik; los servicios de la primera fila quedan documentados en inventario para referencia y futuras integraciones.

##  Caracter�sticas

- **Core Routers**: Configuraci�n con OSPF para enrutamiento din�mico
- **Distribuidores**: Soporte para VLANs de servicios
- **OLT/ONT**: Configuraci�n de terminales �pticas y de usuario
- **Modular**: Roles espec�ficos para cada tipo de dispositivo
- **Escalable**: F�cil de a�adir nuevos hosts

##  Estructura del Proyecto

```
.
 site.yml                 # Playbook principal
 inventory/
    lab.yml             # Inventario de hosts
 group_vars/
    all.yml             # Variables globales
    core.yml            # Variables routers core
    distribuidores.yml  # Variables distribuidores
    transporte.yml      # Variables transporte
    olt.yml             # Variables OLT
    ont.yml             # Variables ONT
 host_vars/              # (opcional) Variables por host
 roles/
     interfaces/         # Configuraci�n de interfaces
     ospf_config/        # Configuraci�n OSPF
     olt_config/         # Configuraci�n OLT
     ont_config/         # Configuraci�n ONT
```

##  Uso

### 1. Instalar dependencias
```bash
pip install ansible paramiko
ansible-galaxy collection install community.routeros
```

### 2. Configurar credenciales (IMPORTANTE)
Edita `group_vars/all.yml` con las credenciales reales:
```yaml
ansible_user: api  # Usar en producci�n
ansible_password: tu_contrase�a_real
```

### 3. Ejecutar playbook completo
```bash
ansible-playbook -i inventory/lab.yml site.yml
```

### 4. Ejecutar para grupo espec�fico
```bash
# Solo routers core
ansible-playbook -i inventory/lab.yml site.yml -l core

# Solo OLT
ansible-playbook -i inventory/lab.yml site.yml -l olt
```

### 5. Modo seco (verificar sin aplicar)
```bash
ansible-playbook -i inventory/lab.yml site.yml --check
```

##  Seguridad - Usar Vault

Para no guardar credenciales en texto plano:

```bash
# Crear archivo encriptado
ansible-vault create group_vars/vault.yml

# Editar archivo encriptado
ansible-vault edit group_vars/vault.yml

# Exportar variables desde vault en all.yml
# include_vars: "{{ playbook_dir }}/group_vars/vault.yml"

# Ejecutar con vault
ansible-playbook -i inventory/lab.yml site.yml --ask-vault-pass
```

##  Variables Principales

### Core
- `ospf_enabled`: true
- `ospf_router_id_base`: "10.10.255"
- `bgp_asn`: 65000

### Distribuidores
- `ospf_area`: "0.0.0.1"
- `bgp_asn`: 65001
- `access_vlans`: Lista de VLANs

### OLT
- `olt_ports`: 24
- `onu_profiles`: Perfiles de ONUs

### ONT
- `wan_vlan`: 100
- `management_vlan`: 101

##  Comandos �tiles

```bash
# Verificar conectividad
ansible -i inventory/lab.yml all -m ping

# Listar hosts
ansible-inventory -i inventory/lab.yml --list

# Ejecutar comando ad-hoc
ansible -i inventory/lab.yml all -m community.routeros.command \
  -a "commands=['/system identity print']"

# Ver variables de un host
ansible -i inventory/lab.yml fw-edge-1 -m debug -a "var=hostvars[inventory_hostname]"
```

##  Pr�ximos pasos de mejora

- [ ] BGP para conectividad inter-AS
- [ ] QoS policies para servicios
- [ ] Backup autom�tico de configuraciones
- [ ] Monitoreo con Prometheus/Grafana
- [ ] Failover y redundancia
- [ ] Templates Jinja2 personalizados

##  Requisitos

- GNS3 con laboratorio configurado
- Ansible 2.9+
- Collection: `community.routeros`
- MikroTik con SSH habilitado

##  Notas

- Si MikroTik est� en GNS3 localmente, usa `127.0.0.1` con puertos diferentes
- Aseg�rate que los dispositivos tengan IPs en la red de management
- El usuario debe tener permisos de administrador

---
Last Updated: 2026-03-02
