# 2. Diseño de la Infraestructura

Para cubrir las tres plantas de **Distributions-Líder** (20 empleados y 2 administradores), la red se segmenta mediante un firewall central (**OPNsense**) que aísla el portal web, los departamentos, el monitoreo y la base de datos.

## 2.1. Diagrama de la infraestructura

![Diagrama de la infraestructura](img/diagrama.jpg)

## 2.2. Descripción de la infraestructura

* **Redes o zonas (VLANs):**
  * **VLAN 10 — DMZ:** Aloja y aísla el portal web público frente a la red interna.
  * **VLAN 20 — INTERNA:** Conecta a los empleados y administradores para compartir recursos y desplegar equipos.
  * **VLAN 30 — MONITOREO Y SEGURIDAD:** Red aislada para supervisar servidores, logs y alertas de seguridad.
  * **VLAN 40 — BASE DE DATOS:** Zona blindada sin acceso desde Internet para proteger los datos de la empresa.

* **Servidores y servicios principales:**
  * **Rancher y Nginx Proxy Manager (VLAN 10):** Orquestan los contenedores del portal web y lo publican por HTTPS.
  * **OpenLDAP y Servidor PXE / Scripts (VLAN 20):** LDAP gestiona usuarios y permisos por departamento; el servidor PXE/repositorio almacena ISOs y automatiza la instalación de sistemas por red en el taller.
  * **Zabbix, OpenSearch y T-Pot (VLAN 30):** Zabbix monitoriza la disponibilidad, OpenSearch centraliza los logs (SIEM) y T-Pot actúa como señuelo (*honeypot*) para detectar intrusiones.
  * **MariaDB (VLAN 40):** Base de datos general que respalda el portal web, el inventario y la facturación.

* **Estructura física real (reparto en 2 servidores físicos) y dispositivos de red:**
  Aunque en el diagrama figuran 8 servidores, físicamente **solo se compran 2 servidores en rack** (clúster **Proxmox VE** en el sótano junto a un **NAS**) donde se crean **8 máquinas virtuales** sin coste extra de licencias:
  * **Servidor Físico 1 (Negocio y Clientes):** Aloja **Rancher**, **Nginx**, **OpenLDAP** y **MariaDB**.
  * **Servidor Físico 2 (Taller y Seguridad):** Aloja el **Servidor PXE** del taller, **Zabbix**, **OpenSearch** y **T-Pot**, evitando que el análisis de logs o las descargas PXE frenen la web o la facturación. Si un servidor físico falla, sus 4 máquinas pasan automáticamente al otro (**Alta Disponibilidad**).
  * **OPNsense (Shield), Router WAN y Cloud:** OPNsense filtra el tráfico entre VLANs, da **DHCP y DNS**, gestiona la **VPN** y envía copias a **AWS S3 / AWS Backup**, saliendo a Internet por el Router WAN (`198.51.100.45`).

## 2.3. Plan de direccionamiento IPv4

| Red / Zona | Dirección de red | Máscara | Gateway | Uso principal |
| :--- | :--- | :--- | :--- | :--- |
| **VLAN 10 DMZ** | `192.168.10.0` | `255.255.255.0 (/24)` | `192.168.10.1` | Portal web público para clientes. |
| **VLAN 20 INTERNA** | `192.168.20.0` | `255.255.255.0 (/24)` | `192.168.20.1` | Usuarios, departamentos, LDAP, PXE y DHCP. |
| **VLAN 30 MONITOREO** | `192.168.30.0` | `255.255.255.0 (/24)` | `192.168.30.1` | Supervisión de red, SIEM y honeypot. |
| **VLAN 40 BD** | `192.168.40.0` | `255.255.255.0 (/24)` | `192.168.40.1` | Aislamiento de la base de datos general. |

| Dispositivo / Servidor | Red / Zona | Dirección IP / Máscara | Gateway | Función |
| :--- | :--- | :--- | :--- | :--- |
| **OPNsense (Shield)** | Todas / WAN | `192.168.x.1 /24` (LANs)<br>`198.51.100.45` (WAN) | Router WAN | Firewall central, DHCP, DNS, VPN y enlace AWS Backup. |
| **Rancher** | VLAN 10 | `192.168.10.20 /24` | `192.168.10.1` | Orquestador de contenedores del portal web. |
| **Nginx Proxy Manager** | VLAN 10 | `192.168.10.30 /24` | `192.168.10.1` | Proxy inverso expuesto a Internet. |
| **LDAP (Directorio)** | VLAN 20 | `192.168.20.20 /24` | `192.168.20.1` | Control de usuarios y permisos departamentales. |
| **Servidor Scripts / PXE** | VLAN 20 | `192.168.20.30 /24` | `192.168.20.1` | Repositorio de ISOs/manuales y despliegue por red (PXE). |
| **Estaciones de trabajo** | VLAN 20 | `192.168.20.100 – .200 /24` | `192.168.20.1` | Rango DHCP para empleados, administradores y taller. |
| **Zabbix** | VLAN 30 | `192.168.30.10 /24` | `192.168.30.1` | Monitoreo de disponibilidad y estado de la red. |
| **OpenSearch** | VLAN 30 | `192.168.30.20 /24` | `192.168.30.1` | Centralización y análisis de logs (SIEM). |
| **T-Pot** | VLAN 30 | `192.168.30.30 /24` | `192.168.30.1` | Honeypot para detección temprana de ataques. |
| **MariaDB** | VLAN 40 | `192.168.40.10 /24` | `192.168.40.1` | Base de datos general (portal web, inventario y facturas). |

**Comprobaciones básicas de red:** **(1)** `ping` de cada servidor a su gateway (`192.168.x.1`) sin conflictos de IP; **(2)** entrega automática de IP por **DHCP** (`.100–.200`) y resolución **DNS** en la VLAN 20; **(3)** aislamiento en OPNsense para que la DMZ solo acceda a MariaDB (`:3306`) y nunca a la VLAN 20 ni 30; **(4)** acceso externo exclusivo a Nginx (`.10.30`), conexión **VPN** y copias en **AWS S3**.
