# 3. Recursos, Presupuesto y Viabilidad Económica

Aunque el diseño técnico se valide mediante virtualización, llevar este proyecto al día a día de **Distributions-Líder** exige dimensionar los equipos físicos, el cableado y el trabajo humano necesarios para cubrir el edificio del Polígono PIBO (sótano, 1.ª y 2.ª planta) y dar servicio a sus 20 empleados y 2 administradores técnicos.

## 11.1. Descripción y justificación de los recursos necesarios

### Servidores físicos y respaldo eléctrico

* **Nodos de virtualización (Clúster Principal):** Dos servidores rack 2U con procesador AMD EPYC / Intel Xeon (24 núcleos), 128 GB de RAM DDR5 ECC, 2 SSD NVMe de 960 GB (RAID 1 para el hipervisor), 4 SSD NVMe U.3 de 3,84 TB (RAID 10 para máquinas virtuales y contenedores), fuente redundante y red dual 10GbE SFP+.
* **Almacenamiento local (NAS / Repositorio PXE):** Cabina NAS rack de 4 bahías con 4 discos Enterprise de 8 TB en RAID 5 (24 TB útiles) e interfaz 10GbE para copias locales rápidas, carpetas departamentales e imágenes ISO.
* **Sistema de Alimentación Ininterrumpida (SAI):** Unidad SAI Online de doble conversión en rack (3000 VA / 2700 W) en el cuarto de comunicaciones del sótano para proteger el equipamiento frente a cortes o picos de tensión.

**Justificación:** Herramientas como el SIEM **OpenSearch**, el honeypot **T-Pot**, la base de datos **MariaDB** y el clúster **Rancher** suman en conjunto unas 30 vCPUs y cerca de 90 GB de RAM junto con un uso intensivo de disco. Repartir la carga en dos nodos de 24 núcleos y 128 GB permite trabajar con total holgura y garantiza alta disponibilidad (HA): si uno de los servidores físicos falla, el otro puede mantener toda la empresa funcionando por sí solo. Además, el enlace de 10 Gbps con el NAS evita cuellos de botella cuando el taller de Montaje instala sistemas operativos por red (PXE) en varios ordenadores a la vez.

### Infraestructura de red y cableado estructurado

* **Cortafuegos perimetral dedicado (OPNsense Shield):** Appliance hardware en rack (6 puertos 2.5GbE y 2 SFP+ 10GbE) que gestiona la segmentación inter-VLAN, DHCP/DNS, el servidor VPN y el enlace con AWS Backup.
* **Conmutación por plantas:** Tres switches gestionables L2/L3 de 24 puertos GbE PoE+ y 4 puertos SFP+ 10G (uno en sótano, 1.ª planta y 2.ª planta), más un switch de 16 puertos GbE en la bancada de Montaje para conectar equipos en reparación y arranque PXE.
* **Cobertura Wi-Fi corporativa:** Tres puntos de acceso Wi-Fi 6 PoE empresariales (uno por nivel, dando prioridad a los lectores de inventario del almacén).
* **Armarios rack y cableado:** Un rack principal de 42U en el sótano y dos murales de 12U en las plantas superiores, unidos por fibra óptica multimodo OM4, junto con cableado Cat 6A LSZH, paneles de parcheo y 48 tomas dobles.
* **Conectividad WAN:** Fibra óptica empresarial simétrica de 1 Gbps con IP pública fija.

**Justificación:** Los 20 ordenadores de la plantilla y las 2 estaciones de los administradores se reutilizan por completo. Al tratarse de un edificio de tres niveles, colocar un switch por planta unido por fibra óptica ahorra tiradas innecesarias de cobre y mantiene una velocidad de 10 Gbps hacia el taller. Por su parte, dedicar un equipo físico exclusivo para **OPNsense** asegura que el filtrado de tráfico y el cifrado VPN no resten potencia a los servidores principales.

### Software, licencias y servicios cloud

* **Virtualización y sistemas base:** Hipervisor Proxmox VE con suscripción básica anual para los dos nodos, sobre distribuciones Linux (Debian / Ubuntu Server) sin coste de licencia.
* **Aplicaciones y seguridad (Open Source):** Despliegue sin coste en licencias de **OPNsense**, **Rancher**, **Nginx Proxy Manager**, **OpenLDAP**, **MariaDB**, **Zabbix**, **OpenSearch** y **T-Pot**.
* **Servicios Cloud y dominio:** Suscripción anual a **AWS S3** (alta redundancia) integrada con **AWS Backup** para respaldar unos 3 TB de facturación, inventario y configuraciones críticas, además del dominio web público.

**Justificación:** Apostar por software libre de nivel empresarial permite cubrir todos los requisitos técnicos sin atar a la empresa al pago anual de licencias por usuario o por núcleo. El gasto recurrente se concentra donde realmente aporta tranquilidad: el soporte del hipervisor y la nube de Amazon, garantizando que, ante cualquier imprevisto grave en la nave, el negocio pueda recuperar sus datos en minutos.

### Personal técnico de implantación

* **Ingeniero de Sistemas y Ciberseguridad (90 horas):** Configuración del clúster, reglas de OPNsense, despliegue de contenedores y bases de datos, automatización PXE, puesta en marcha del SIEM/monitoreo e integración con AWS.
* **Técnico Instalador de Telecomunicaciones (60 horas):** Montaje de armarios rack, tendido de fibra entre plantas, cableado de tomas y certificación de la red.

**Justificación:** Aunque los 2 administradores en plantilla llevarán el mantenimiento diario una vez entregada la red, levantar desde cero el cableado de tres plantas y toda la arquitectura de seguridad requiere apoyo externo durante unas tres semanas. De este modo, el equipo interno no desatiende el soporte cotidiano de la empresa durante la transición.

## 11.2. Presupuesto económico de implantación

A continuación se detalla el presupuesto estimado a precios actuales de mercado:

| Categoría | Concepto | Cant. | Precio Unit. (€) | Coste Total (€) |
| :--- | :--- | :---: | :---: | :---: |
| **Servidores** | Servidor Rack 2U 24C, 128 GB RAM ECC, NVMe RAID, 10GbE | 2 | 3.850,00 | 7.700,00 |
| **Almacenamiento** | Cabina NAS Rack 4 bahías (24 TB útiles RAID 5) 10GbE | 1 | 1.350,00 | 1.350,00 |
| **Protección** | SAI Online Doble Conversión Rack 3000 VA | 1 | 890,00 | 890,00 |
| **Red y Firewall** | Appliance Hardware Rack OPNsense (6x 2.5GbE + 2x SFP+) | 1 | 720,00 | 720,00 |
| **Red Planta** | Switch Gestionable L2/L3 24p GbE (PoE+) + 4 SFP+ 10G | 3 | 440,00 | 1.320,00 |
| **Red Taller** | Switch Gestionable 16p GbE para bancada PXE / Montaje | 1 | 190,00 | 190,00 |
| **Wi-Fi** | Punto de Acceso Wi-Fi 6 PoE Empresarial (Sótano, 1.ª y 2.ª planta) | 3 | 155,00 | 465,00 |
| **Racks y Cable** | Rack 42U (sótano), 2 Racks 12U (plantas), PDU y bandejas | 1 | 1.120,00 | 1.120,00 |
| **Racks y Cable** | Fibra OM4 entre plantas, cable Cat 6A LSZH y 48 tomas | 1 | 1.380,00 | 1.380,00 |
| **Software** | Suscripción anual Proxmox VE Community (por nodo físico) | 2 | 355,00 | 710,00 |
| **Cloud AWS** | Servicio anual AWS S3 Alta Redundancia + AWS Backup + Dominio | 1 | 840,00 | 840,00 |
| **Conectividad** | Fibra empresarial simétrica 1 Gbps con IP fija (anual) | 1 | 660,00 | 660,00 |
| **Personal** | Horas de Ingeniero de Sistemas y Ciberseguridad | 90 | 45,00 | 4.050,00 |
| **Personal** | Horas de Técnico Instalador de Telecomunicaciones | 60 | 30,00 | 1.800,00 |
| **Subtotal** | **Base Imponible (sin impuestos)** | — | — | **23.195,00** |
| **Impuestos** | **IVA aplicable (21 %)** | — | — | **4.870,95** |
| **Total** | **Presupuesto Total de Implantación** | — | — | **28.065,95** |

## 11.3. Financiación y viabilidad económica

### Propuesta de financiación

Para no restar liquidez a la compra habitual de stock informático y componentes, se plantea un modelo de **financiación mixta**:

1. **Renting tecnológico para el equipamiento electrónico (12.635,00 € + IVA):** Los servidores, el NAS, el SAI, el firewall, los switches y los puntos Wi-Fi se financian mediante *renting* a 36 meses. Supone una cuota cómoda de unos **405 €/mes** (menos de 19 € al mes por trabajador), incluye seguro frente a averías, facilita renovar el hardware a los tres años y es 100 % deducible en el Impuesto sobre Sociedades.
2. **Fondos propios para instalación y servicios anuales (10.560,00 € + IVA):** El cableado, los armarios rack, la mano de obra de implantación y el primer año de AWS, fibra y soporte se abonan con tesorería propia, evitando pagar intereses por servicios o trabajo técnico.
3. **Ayudas públicas y líneas bonificadas:** Al ser una pyme de 22 trabajadores ubicada en el Polígono PIBO (Bollullos de la Mitación), la empresa puede solicitar los incentivos de digitalización y ciberseguridad de la Junta de Andalucía (Agencia TRADE) o apoyarse en una línea ICO Empresas.

### Valoración de la viabilidad económica

Una inversión de **23.195,00 € más IVA** resulta muy equilibrada y rentable para el volumen de negocio de **Distributions-Líder** por cuatro motivos prácticos:

* **Ahorro real en licencias:** Montar esta misma arquitectura con software propietario (licencias de Windows Server Datacenter, 22 CALs de Active Directory, un SIEM comercial como Splunk y un firewall de pago con suscripción anual) dispararía el presupuesto por encima de los 45.000 €. El enfoque Open Source recorta la factura a la mitad sin perder prestaciones.
* **Aprovechamiento de lo que ya funciona:** Se conservan todos los puestos de trabajo actuales y no hace falta contratar un mantenimiento mensual externo, ya que los 2 administradores de la casa asumirán la gestión diaria.
* **Impacto directo en el taller y en las ventas:** El despliegue automático por red (PXE) ahorra horas de trabajo manual al preparar o reparar equipos de clientes, mientras que el nuevo portal web abre un escaparate de venta y consulta de incidencias operativo las 24 horas.
* **Blindaje frente a paradas del negocio:** En un taller por el que pasan ordenadores de terceros potencialmente infectados, aislar las redes con OPNsense, vigilar el tráfico con Zabbix, OpenSearch y T-Pot, y respaldar los datos críticos en **AWS S3** evita que un incidente de seguridad paralice la facturación de la empresa.
