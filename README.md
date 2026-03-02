# ISP Network Automation - GNS3 Lab

Automatización de configuración para red ISP simulada en GNS3 usando Ansible.

##  Topología de Red

```
         NAT1 (CCR2116-12G-3)
            |
    +-------+-------+
    |               |
 Torre-1       Torre-2
(CCR2116-12G-1) (CCR2116-12G-2)
    |               |
    +-------+-------+
            |
       Transport
    (CCR2116-12G-4)
            |
          OLT
        (CS328-24P)
            |
          ONT
        (CHR7.1/1)
```

##  Características

- **Core Routers**: Configuración con OSPF para enrutamiento dinámico
- **Distribuidores**: Soporte para VLANs de servicios
- **OLT/ONT**: Configuración de terminales ópticas y de usuario
- **Modular**: Roles específicos para cada tipo de dispositivo
- **Escalable**: Fácil de añadir nuevos hosts

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
     interfaces/         # Configuración de interfaces
     ospf_config/        # Configuración OSPF
     olt_config/         # Configuración OLT
     ont_config/         # Configuración ONT
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
ansible_user: api  # Usar en producción
ansible_password: tu_contraseña_real
```

### 3. Ejecutar playbook completo
```bash
ansible-playbook -i inventory/lab.yml site.yml
```

### 4. Ejecutar para grupo específico
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
- `ospf_router_id_base`: "10.255.255"
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

##  Comandos útiles

```bash
# Verificar conectividad
ansible -i inventory/lab.yml all -m ping

# Listar hosts
ansible-inventory -i inventory/lab.yml --list

# Ejecutar comando ad-hoc
ansible -i inventory/lab.yml all -m community.routeros.command \
  -a "commands=['/system identity print']"

# Ver variables de un host
ansible -i inventory/lab.yml nat1 -m debug -a "var=hostvars[inventory_hostname]"
```

##  Próximos pasos de mejora

- [ ] BGP para conectividad inter-AS
- [ ] QoS policies para servicios
- [ ] Backup automático de configuraciones
- [ ] Monitoreo con Prometheus/Grafana
- [ ] Failover y redundancia
- [ ] Templates Jinja2 personalizados

##  Requisitos

- GNS3 con laboratorio configurado
- Ansible 2.9+
- Collection: `community.routeros`
- MikroTik con SSH habilitado

##  Notas

- Si MikroTik está en GNS3 localmente, usa `127.0.0.1` con puertos diferentes
- Asegúrate que los dispositivos tengan IPs en la red de management
- El usuario debe tener permisos de administrador

---
Last Updated: 2026-03-02
