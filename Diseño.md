# 2. Diseño de la Infraestructura

Para dar respuesta a las necesidades de **Distributions-Líder** en su edificio de dos plantas y sótano (20 empleados y 2 administradores técnicos), se ha diseñado una red segmentada que separa el portal web público, el trabajo diario de los departamentos, la vigilancia de seguridad y la base de datos, respaldando toda la información crítica en la nube.

## 2.1. Diagrama de la infraestructura

El siguiente esquema muestra la topología lógica de la red, el direccionamiento IPv4 y las conexiones entre las cuatro VLANs, el firewall central **OPNsense**, Internet y **AWS Cloud**:

![Diagrama de la infraestructura](img/diagrama.jpg)

## 2.2. Descripción de la infraestructura

Toda la red se articula alrededor de un cortafuegos central (**OPNsense**) que divide el tráfico en cuatro zonas lógicas (VLANs) bien diferenciadas:

* **Redes o zonas (VLANs):**
  * **VLAN 10 — DMZ:** Aísla el portal web público frente al resto de la empresa para que una posible intrusión desde Internet no alcance los equipos internos.
  * **VLAN 20 — INTERNA:** Red de trabajo para los empleados y administradores donde se comparten carpetas, se autentican usuarios y funciona el despliegue por red del taller.
  * **VLAN 30 — MONITOREO Y SEGURIDAD:** Red dedicada exclusivamente a vigilar el estado de los servidores, analizar registros y detectar intentos de ataque.
  * **VLAN 40 — BASE DE DATOS:** Zona blindada y sin acceso directo desde el exterior para proteger los datos de facturación, inventario y pedidos web.

* **Servidores y servicios principales:**
  * **Rancher y Nginx Proxy Manager (VLAN 10):** Rancher gestiona los contenedores del portal web y Nginx actúa como proxy inverso para publicarlo hacia Internet de forma segura (HTTPS).
  * **OpenLDAP (VLAN 20):** Directorio centralizado para gestionar las cuentas de los trabajadores y los permisos de acceso de cada departamento.
  * **Servidor de Scripts / PXE y Estaciones de trabajo (VLAN 20):** Repositorio central con imágenes ISO, manuales y arranque por red (PXE) para automatizar la instalación de equipos en el taller, junto con los puestos de los empleados.
  * **Stack de Monitoreo — Zabbix, OpenSearch y T-Pot (VLAN 30):** Zabbix vigila que los servidores y switches estén operativos, OpenSearch centraliza los logs de seguridad (SIEM) y T-Pot funciona como *honeypot* (señuelo) para detectar intrusos a tiempo.
  * **MariaDB (VLAN 40):** Base de datos central que da servicio tanto al portal web de clientes como al inventario y la facturación interna.

* **Dispositivos de red, operativa y cloud:**
  * **OPNsense (Shield):** Cortafuegos principal y puerta de enlace de las cuatro VLANs. Filtra qué tráfico puede pasar entre redes, reparte IPs dinámicas (**DHCP**), resuelve nombres locales (**DNS**), gestiona el acceso **VPN** y conecta las copias de seguridad con AWS.
  * **Router WAN y Switch virtual:** El router WAN conecta el firewall a Internet (IP pública `198.51.100.45`), mientras que el switch virtual de la DMZ une Rancher y Nginx con la interfaz del cortafuegos.
  * **Distribución física y nube:** La red da cobertura al sótano (4 empleados de Almacén y cuarto de servidores), 1.ª planta (8 técnicos de Montaje y 2 administradores) y 2.ª planta (8 empleados de Administración y Marketing). Desde el exterior solo se puede entrar a la web (vía Nginx) o por la **VPN** de teletrabajo, mientras que **AWS S3 / AWS Backup** guarda fuera de la nave las copias de seguridad.

## 2.3. Plan de direccionamiento IPv4

Direccionamiento de las cuatro subredes principales (sin solapamientos y numeradas según su VLAN para facilitar su gestión):

| Red / Zona | Dirección de red | Máscara | Gateway | Uso principal |
| :--- | :--- | :--- | :--- | :--- |
| **VLAN 10 DMZ** | `192.168.10.0` | `255.255.255.0 (/24)` | `192.168.10.1` | Portal web público para clientes. |
| **VLAN 20 INTERNA** | `192.168.20.0` | `255.255.255.0 (/24)` | `192.168.20.1` | Usuarios, departamentos, LDAP, PXE y DHCP. |
| **VLAN 30 MONITOREO** | `192.168.30.0` | `255.255.255.0 (/24)` | `192.168.30.1` | Supervisión de red, SIEM y honeypot. |
| **VLAN 40 BD** | `192.168.40.0` | `255.255.255.0 (/24)` | `192.168.40.1` | Aislamiento de la base de datos general. |

Asignación de direcciones IP fijas para servidores y rango dinámico (DHCP) para equipos:

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

**Comprobaciones básicas de la configuración de red:**
* **Conectividad a puertas de enlace:** Comprobar mediante `ping` que todos los servidores llegan a su gateway en OPNsense (`192.168.10.1`, `.20.1`, `.30.1` y `.40.1`) sin conflictos de IP.
* **Funcionamiento de DHCP y DNS:** Verificar que al conectar un equipo en la VLAN 20 recibe IP automática dentro del rango `.100 – .200` y resuelve tanto nombres internos como páginas externas.
* **Aislamiento entre VLANs:** Confirmar que la DMZ (VLAN 10) solo puede consultar MariaDB (puerto `3306` en VLAN 40) y tiene bloqueado el paso hacia la red interna (VLAN 20) y la red de monitoreo (VLAN 30).
* **Acceso exterior, VPN y copias cloud:** Validar que desde Internet solo sea accesible el portal web a través de Nginx (`.10.30`), que el túnel VPN conecte correctamente y que OPNsense complete las copias hacia AWS S3.
