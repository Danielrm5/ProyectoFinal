# 3. Recursos, Presupuesto y Viabilidad Económica

Aunque el diseño lógico se valide mediante virtualización, implantar esta infraestructura en la operativa real de **Distributions-Líder** exige dimensionar con precisión los equipos físicos, el cableado estructurado, las licencias y el trabajo humano necesarios para cubrir los tres niveles del edificio en el Polígono PIBO (sótano, 1.ª y 2.ª planta) y dar servicio a sus **20 empleados y 2 administradores técnicos**.

---

## 11.1. Descripción y justificación de los recursos necesarios

### 1. Servidores físicos, almacenamiento y respaldo eléctrico

* **Nodos de virtualización (Clúster Principal en Alta Disponibilidad):** Dos servidores rack 2U con procesador AMD EPYC / Intel Xeon (24 núcleos / 48 hilos), **128 GB de RAM DDR5 ECC**, 2 SSD NVMe de 960 GB (RAID 1 dedicado al hipervisor Proxmox VE), 4 SSD NVMe U.3 Enterprise de 3,84 TB (RAID 10 con 7,68 TB útiles de alto rendimiento para máquinas virtuales y contenedores), fuente de alimentación redundante y tarjeta de red dual 10GbE SFP+.
* **Almacenamiento local compartido (Cabina NAS / Repositorio PXE):** Cabina NAS en formato rack de 4 bahías equipada con 4 discos Enterprise de 8 TB en **RAID 5 (24 TB útiles)** y doble interfaz 10GbE SFP+ para copias de seguridad locales rápidas, carpetas departamentales, scripts e imágenes ISO.
* **Sistema de Alimentación Ininterrumpida (SAI):** Unidad SAI Online de doble conversión en rack (**3000 VA / 2700 W**) instalada en el cuarto de comunicaciones del sótano para proteger los servidores, el NAS y la electrónica de red frente a cortes de suministro o picos de tensión en el polígono industrial.

#### Tabla de dimensionamiento de recursos por servidor virtual / servicio

Para justificar técnicamente la elección del clúster de 2 nodos (48 núcleos y 256 GB de RAM totales, con capacidad de asumir toda la carga en un solo nodo de 24 núcleos y 128 GB en caso de fallo de hardware), se detalla el reparto de recursos asignados a cada servicio del diseño:

| Servidor / Servicio | VLAN | IP asignada | vCPU | RAM (GB) | Disco NVMe (GB) | Justificación de recursos asignados |
| :--- | :---: | :--- | :---: | :---: | :---: | :--- |
| **Rancher (Clúster Web)** | VLAN 10 | `192.168.10.20` | 4 | 12 GB | 160 GB | Orquestación de contenedores del portal de pedidos y seguimiento de reparaciones. |
| **Nginx Proxy Manager** | VLAN 10 | `192.168.10.30` | 2 | 4 GB | 40 GB | Proxy inverso HTTP/HTTPS, terminación SSL/TLS y filtrado de peticiones entrantes. |
| **OpenLDAP (Directorio)** | VLAN 20 | `192.168.20.20` | 2 | 4 GB | 40 GB | Autenticación centralizada de los 22 trabajadores y control de grupos departamentales. |
| **Servidor Scripts / PXE** | VLAN 20 | `192.168.20.30` | 4 | 8 GB | 250 GB (+ NAS 24 TB) | Arranque en red TFTP/HTTP/NFS para despliegue simultáneo de SO en el taller de montaje. |
| **Zabbix (Monitoreo)** | VLAN 30 | `192.168.30.10` | 4 | 8 GB | 200 GB | Recolección continua de métricas SNMP/agentes de servidores, switches y enlaces. |
| **OpenSearch (SIEM / Logs)** | VLAN 30 | `192.168.30.20` | 6 | 24 GB | 800 GB | Indexación en memoria y correlación de logs de seguridad de toda la infraestructura. |
| **T-Pot (Honeypot)** | VLAN 30 | `192.168.30.30` | 4 | 16 GB | 250 GB | Ejecución simultánea de múltiples contenedores señuelo para detección de intrusos. |
| **MariaDB (Base de Datos)** | VLAN 40 | `192.168.40.10` | 4 | 16 GB | 500 GB | Caché InnoDB en RAM y almacenamiento NVMe RAID 10 para portal web, stock y facturación. |
| **Total asignado a VMs** | **Todas** | — | **30 vCPU** | **92 GB** | **2.240 GB (2,24 TB)** | **Funciona con holgura sobre 1 solo nodo (128 GB RAM) si el otro entra en mantenimiento (HA).** |

### 2. Infraestructura de red y cableado estructurado

* **Cortafuegos perimetral dedicado (OPNsense Shield):** Appliance hardware en rack (procesador multinúcleo con aceleración AES-NI, 16 GB RAM, 6 puertos 2.5GbE y 2 puertos SFP+ 10GbE) que gestiona el enrutamiento inter-VLAN, DHCP, DNS, el servidor VPN y el enlace cifrado con AWS Backup.
* **Conmutación por plantas:** Tres switches gestionables L2/L3 de 24 puertos GbE PoE+ y 4 puertos SFP+ 10G (uno en el sótano, otro en la 1.ª planta y otro en la 2.ª planta), más un switch gestionable de 16 puertos GbE situado en la bancada del taller de Montaje para conectar los equipos de clientes en reparación y despliegue PXE.
* **Cobertura Wi-Fi corporativa:** Tres puntos de acceso Wi-Fi 6 PoE empresariales (uno por nivel, dando cobertura prioritaria a los lectores inalámbricos de inventario del almacén en el sótano).
* **Armarios rack y cableado estructurado:** Un armario rack principal de 42U en el cuarto de comunicaciones del sótano y dos armarios murales de 12U en las plantas 1.ª y 2.ª, interconectados mediante un troncal vertical de **fibra óptica multimodo OM4 (10 Gbps)**, junto con cableado horizontal **Cat 6A LSZH**, paneles de parcheo, regletas PDU y 48 tomas dobles RJ45.
* **Conectividad WAN:** Línea de fibra óptica empresarial simétrica de **1 Gbps con IP pública fija** (`198.51.100.45`).

**Justificación:** Los 20 ordenadores de los empleados y las 2 estaciones de los administradores técnicos ya existentes en la empresa se reutilizan íntegramente. Al tratarse de un edificio de tres plantas, instalar un switch por planta unido por fibra óptica OM4 de 10 Gbps evita cuellos de botella durante el despliegue masivo de imágenes ISO desde el NAS hacia el taller de la 1.ª planta. Dedicar un appliance físico independiente para **OPNsense** garantiza que el filtrado de paquetes y el cifrado VPN funcionen a velocidad de cable sin depender del hipervisor.

### 3. Software, licencias y servicios cloud

* **Virtualización y sistemas operativos base:** Hipervisor **Proxmox VE** con suscripción anual *Community* para los dos nodos físicos (acceso al repositorio Enterprise estable), ejecutando máquinas virtuales y contenedores sobre **Debian / Ubuntu Server LTS** sin coste de licencia.
* **Aplicaciones corporativas y de ciberseguridad (Open Source):** Despliegue sin coste de licencias de software de **OPNsense**, **Rancher**, **Nginx Proxy Manager**, **OpenLDAP**, **MariaDB**, **Zabbix**, **OpenSearch** y **T-Pot**.
* **Servicios Cloud y dominio web:** Suscripción anual a **AWS S3 (Alta Redundancia)** integrada con **AWS Backup** para almacenar de forma externa y cifrada hasta 3 TB de copias de seguridad (base de datos MariaDB, facturación, inventario y configuraciones), más el registro anual del dominio público corporativo.

**Justificación:** El uso de soluciones Open Source de estándar empresarial elimina por completo el pago recurrente de licencias por usuario (CALs) o por núcleo. El gasto anual de software se concentra exclusivamente en la estabilidad del hipervisor y en la copia de seguridad externa en **Amazon Web Services**, asegurando una recuperación rápida ante cualquier desastre físico en la nave.

### 4. Personal técnico de implantación

* **Ingeniero de Sistemas y Ciberseguridad (90 horas):** Instalación y puesta en alta disponibilidad del clúster Proxmox VE, configuración de VLANs y reglas de filtrado en OPNsense, despliegue de OpenLDAP, MariaDB, Rancher/Nginx, automatización del servidor PXE, configuración del stack SIEM/monitoreo (Zabbix, OpenSearch, T-Pot) e integración de backups con AWS S3.
* **Técnico Instalador de Telecomunicaciones (60 horas):** Montaje del rack de 42U y de los dos racks murales de 12U, tendido y fusión de la fibra óptica OM4 entre plantas, cableado Cat 6A de las 48 tomas dobles, instalación de los puntos Wi-Fi 6 PoE y certificación de la red.

**Justificación:** Aunque los **2 administradores técnicos** en plantilla asumirán la administración diaria y el mantenimiento una vez entregada la infraestructura, ejecutar la obra de cableado en tres plantas y levantar desde cero toda la arquitectura de virtualización y seguridad requiere apoyo especializado externo durante tres semanas, evitando paralizar la actividad diaria de la empresa.

---

## 11.2. Presupuesto económico de implantación

A continuación se detalla el presupuesto completo a precios reales de mercado, desglosando cantidades, precios unitarios e importes totales de cada partida (equipamiento, cableado, software/cloud, conectividad y mano de obra):

| Categoría | Concepto | Cant. | Precio Unit. (€) | Coste Total (€) |
| :--- | :--- | :---: | :---: | :---: |
| **Servidores** | Servidor Rack 2U 24C, 128 GB RAM ECC, NVMe RAID 1+10, 10GbE | 2 | 3.850,00 | 7.700,00 |
| **Almacenamiento** | Cabina NAS Rack 4 bahías (24 TB útiles RAID 5) 10GbE | 1 | 1.350,00 | 1.350,00 |
| **Protección** | SAI Online Doble Conversión Rack 3000 VA / 2700 W | 1 | 890,00 | 890,00 |
| **Red y Firewall** | Appliance Hardware Rack OPNsense (6x 2.5GbE + 2x SFP+ 10G) | 1 | 720,00 | 720,00 |
| **Red Planta** | Switch Gestionable L2/L3 24p GbE (PoE+) + 4 SFP+ 10G | 3 | 440,00 | 1.320,00 |
| **Red Taller** | Switch Gestionable 16p GbE para bancada PXE / Montaje | 1 | 190,00 | 190,00 |
| **Wi-Fi** | Punto de Acceso Wi-Fi 6 PoE Empresarial (Sótano, 1.ª y 2.ª planta) | 3 | 155,00 | 465,00 |
| **Racks y Cable** | Rack 42U (sótano), 2 Racks murales 12U (plantas), PDU y bandejas | 1 | 1.120,00 | 1.120,00 |
| **Racks y Cable** | Fibra OM4 entre plantas, cable Cat 6A LSZH y 48 tomas dobles | 1 | 1.380,00 | 1.380,00 |
| **Software** | Suscripción anual Proxmox VE Community (por nodo físico) | 2 | 355,00 | 710,00 |
| **Cloud AWS** | Servicio anual AWS S3 Alta Redundancia + AWS Backup + Dominio | 1 | 840,00 | 840,00 |
| **Conectividad** | Fibra empresarial simétrica 1 Gbps con IP fija (cuota anual) | 1 | 660,00 | 660,00 |
| **Personal** | Horas de Ingeniero de Sistemas y Ciberseguridad | 90 | 45,00 | 4.050,00 |
| **Personal** | Horas de Técnico Instalador de Telecomunicaciones | 60 | 30,00 | 1.800,00 |
| **Subtotal** | **Base Imponible (sin impuestos)** | — | — | **23.195,00** |
| **Impuestos** | **IVA aplicable (21 %)** | — | — | **4.870,95** |
| **Total** | **Presupuesto Total de Implantación (IVA incluido)** | — | — | **28.065,95** |

### Resumen por tipo de gasto (Inversión inicial vs. Coste operativo anual)

* **Equipamiento electrónico financiable (Servidores, NAS, SAI, Firewall, Switches y Wi-Fi):** `12.635,00 €` + IVA.
* **Infraestructura pasiva y mano de obra de implantación (Racks, cableado e ingeniería):** `8.350,00 €` + IVA (pago único en el despliegue).
* **Costes recurrentes anuales (Proxmox VE, AWS S3/Backup y Fibra 1 Gbps):** `2.210,00 € / año` + IVA (`184,17 € / mes`).

---

## 11.3. Financiación y viabilidad económica

### 1. Propuesta de financiación concreta

Para preservar la liquidez de **Distributions-Líder** destinada a la compra habitual de stock informático y componentes para clientes, el importe presupuestado (**23.195,00 € + IVA**) se articula mediante un modelo de **financiación mixta** ajustado al céntimo con las partidas de la tabla anterior:

1. **Renting tecnológico a 36 meses para el equipamiento electrónico (`12.635,00 € + IVA`):**
   * Cubre las siete primeras partidas del presupuesto: los 2 servidores rack (`7.700 €`), la cabina NAS (`1.350 €`), el SAI (`890 €`), el appliance OPNsense (`720 €`), los 4 switches (`1.510 €`) y los 3 puntos Wi-Fi 6 (`465 €`).
   * Supone una cuota fija muy asumible de **`405,00 € / mes` (+ IVA)** durante 3 años, incluye seguro a todo riesgo frente a averías de hardware, evita la descapitalización inicial, facilita la renovación tecnológica al finalizar el contrato y es **100 % deducible como gasto fiscal** en el Impuesto sobre Sociedades (además de deducir el IVA soportado trimestralmente).
2. **Fondos propios de tesorería para instalación pasiva y primer año de servicios (`10.560,00 € + IVA`):**
   * Cubre las partidas no sujetas a renting: armarios rack y cableado estructurado (`2.500 €`), mano de obra de ingeniería e instalación (`5.850 €`) y el primer año completo de suscripción Proxmox, AWS S3/Backup y fibra óptica (`2.210 €`). Abonar estas partidas con recursos propios evita pagar intereses financieros sobre horas de trabajo o servicios anuales.
3. **Subvenciones y líneas bonificadas aplicables:**
   * Al ser una pyme de 22 trabajadores radicada en el Polígono Industrial PIBO (Bollullos de la Mitación, Sevilla), la empresa cumple los requisitos para acogerse a los incentivos de transformación digital y ciberseguridad industrial de la **Junta de Andalucía (Agencia TRADE)** o financiar la parte de instalación mediante una línea **ICO Empresas y Emprendedores**.

### 2. Valoración razonada de la viabilidad económica

La inversión total de **23.195,00 € (base imponible)** es plenamente viable, proporcionada y rentable para una empresa de distribución y reparación informática de **22 trabajadores** por cinco razones económicas y operativas:

1. **Esfuerzo financiero mensual mínimo por trabajador:** Con el modelo mixto propuesto, el desembolso inicial de tesorería se reduce a `10.560 €` y el coste fijo mensual durante los tres primeros años es de apenas **`405 €/mes`** de renting (más `184 €/mes` a partir del segundo año en servicios cloud/fibra), lo que equivale a **menos de 27 € al mes por trabajador**, una cifra mínima dentro de la estructura de costes de una pyme con 22 empleados.
2. **Ahorro superior al 50 % frente a software propietario:** Implementar esta misma arquitectura con licencias comerciales equivalentes (Windows Server Datacenter en 48 núcleos, 22 CALs de Active Directory, un SIEM propietario como Splunk y un firewall comercial con suscripción UTM anual) elevaría el presupuesto por encima de los **45.000 €** iniciales y más de **6.000 €/año** en renovaciones. La arquitectura Open Source elegida ahorra más de **22.000 €** sin renunciar a prestaciones empresariales.
3. **Reutilización de activos y soporte interno propio:** Se aprovechan al 100 % los 22 puestos de trabajo actuales y no es necesario contratar una cuota mensual de mantenimiento externo, ya que los **2 administradores técnicos** en plantilla gestionan directamente la plataforma.
4. **Retorno de la inversión (ROI) directo en el taller y en las ventas:**
   * En la 1.ª planta (8 técnicos de Montaje y Reparaciones), el despliegue automatizado por red (**PXE**) a 10 Gbps reduce el tiempo de instalación y configuración de cada lote de ordenadores de horas a minutos, aumentando la capacidad diaria de facturación del taller.
   * El nuevo **portal web en la DMZ** abre un canal de pedidos B2B y consulta de reparaciones operativo las 24 horas, captando ventas de tiendas y pymes clientes sin sobrecargar la atención telefónica.
5. **Protección frente a pérdidas por paradas de negocio:** En un taller por el que pasan diariamente equipos de clientes potencialmente infectados, un incidente de *ransomware* que alcanzase la base de datos de facturación o inventario tendría un coste superior al de toda la infraestructura. La segmentación en 4 VLANs con **OPNsense**, la vigilancia con **Zabbix, OpenSearch y T-Pot** y el respaldo externo en **AWS S3** blindan la continuidad del negocio.
