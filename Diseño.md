# 1. Necesidad

La empresa **Distributions-Líder**, ubicada en el polígono PIBO y dedicada a la distribución y reparación de equipos informáticos para pymes, requiere una infraestructura de red segura para su edificio de dos plantas y sótano. Con 20 empleados distribuidos en cuatro departamentos (Montaje y Reparaciones, Almacén y Logística, Administración y Marketing) y 2 administradores técnicos, necesita compartir recursos internos de forma fluida (inventarios, facturas, ISOs), automatizar despliegues por red (PXE) y ofrecer un portal web externo para clientes respaldado mediante copias de seguridad en la nube.

# 2. Trabajo práctico

## 2.1. Propuesta de infraestructura

La infraestructura se segmenta mediante un firewall central (**OPNsense**) en cuatro zonas lógicas:

*   **Redes o zonas (VLANs):**
    *   **VLAN 10 DMZ:** Aloja y aísla el portal web público frente a la red interna.
    *   **VLAN 20 INTERNA:** Conecta a los empleados y administradores para compartir recursos corporativos.
    *   **VLAN 30 MONITOREO Y SEGURIDAD:** Red aislada para supervisar la disponibilidad y seguridad de los servidores.
    *   **VLAN 40 BASE DE DATOS:** Aislamiento estricto para proteger los datos corporativos y del portal web.

*   **Servidores y servicios principales:**
    *   **Rancher y Nginx Proxy Manager (VLAN 10):** Orquestan contenedores
    *   **Estaciones de trabajo y Scripts de automatización (VLAN 20):** Equipos de usuario y repositorio central para ISOs, manuales y despliegue de sistemas operativos por red (PXE).
    *   **Stack de Monitoreo - Zabbix, OpenSearch y T-Pot (VLAN 30):** Supervisa el estado de la red, centraliza el análisis de logs (SIEM) y detecta ataques mediante señuelos (*honeypot*).
    *   **MariaDB (VLAN 40):** Base de datos general que respalda el portal web, la facturación y el inventario.

*   **Dispositivos de red:**
    *   **OPNsense (Shield):** Firewall perimetral e inter-VLAN que filtra el tráfico, asigna IPs dinámicas (DHCP), gestiona el túnel VPN y conecta la interfaz de backup hacia AWS.
    *   **Router WAN:** Enlace principal entre el firewall OPNsense e Internet.
    *   **Switch virtual:** Conmutador lógico en la DMZ para interconectar Rancher y Nginx Proxy Manager.

*   **Operativa y accesibilidad:**
    *   **Ubicación física:** Sótano (4 empleados de Almacén), 1.ª planta (8 empleados de Montaje) y 2.ª planta (8 empleados de Administración y Marketing), junto con 2 administradores técnicos operando en la VLAN 20.
    *   **Accesibilidad externa y VPN:** Desde Internet solo son accesibles el portal web (vía Nginx) y la VPN segura en OPNsense para teletrabajo y administración remota.
    *   **Servicios Cloud (AWS S3):** Almacenamiento externo de alta redundancia conectado a OPNsense para copias de seguridad.

## 2.2. Diagrama de la infraestructura

El siguiente esquema representa la topología lógica, el direccionamiento y las conexiones entre las cuatro VLANs, el firewall OPNsense, Internet y AWS Cloud:

![Imagen Diagrama](/img/diagrama.jpg)

## 2.3. Plan de direccionamiento IPv4

Direccionamiento base de las redes definidas:

| Red / Zona | Dirección de red | Máscara | Gateway | Observaciones |
| :--- | :--- | :--- | :--- | :--- |
| **VLAN 10 DMZ** | 192.168.10.0 | 255.255.255.0 (/24) | 192.168.10.1 | Portal web público para clientes. |
| **VLAN 20 INTERNA** | 192.168.20.0 | 255.255.255.0 (/24) | 192.168.20.1 | Gestión de usuarios y departamentos. |
| **VLAN 30 MONITOREO** | 192.168.30.0 | 255.255.255.0 (/24) | 192.168.30.1 | Supervisión y seguridad de red. |
| **VLAN 40 BD** | 192.168.40.0 | 255.255.255.0 (/24) | 192.168.40.1 | Aislamiento de base de datos general. |

Dispositivos de infraestructura y servidores con dirección IP fija:

| Dispositivo / Servidor | Red / Zona | Dirección IP / Máscara | Gateway | Observaciones |
| :--- | :--- | :--- | :--- | :--- |
| **OPNsense (Shield)** | Todas | 192.168.x.1 /24 | 198.51.100.45 | Firewall central, DHCP, VPN y enlace AWS Backup. |
| **Rancher** | VLAN 10 | 192.168.10.20 /24 | 192.168.10.1 | Orquestador del portal web. |
| **Nginx Proxy Manager** | VLAN 10 | 192.168.10.30 /24 | 192.168.10.1 | Proxy inverso expuesto a Internet. |
| **LDAP (Directorio)** | VLAN 20 | 192.168.20.20 /24 | 192.168.20.1 | Control de permisos departamentales. |
| **Estaciones / Scripts** | VLAN 20 | 192.168.20.30 /24 | 192.168.20.1 | Equipos de usuario y automatización (PXE / ISOs). |
| **Zabbix** | VLAN 30 | 192.168.30.10 /24 | 192.168.30.1 | Monitoreo de estado de la red. |
| **OpenSearch** | VLAN 30 | 192.168.30.20 /24 | 192.168.30.1 | Centralización y análisis de logs (SIEM). |
| **T-Pot** | VLAN 30 | 192.168.30.30 /24 | 192.168.30.1 | Honeypot para detección de ataques. |
| **MariaDB** | VLAN 40 | 192.168.40.10 /24 | 192.168.40.1 | Base de datos general (portal y corporativa). |
