# 3. Recursos, Presupuesto y Viabilidad Económica

Llevar este diseño a la práctica en el edificio de **Distributions-Líder** (sótano, 1.ª y 2.ª planta, con 20 empleados y 2 administradores) requiere dimensionar el hardware, el cableado y el trabajo de implantación a precios reales.

## 11.1. Descripción y justificación de los recursos necesarios

* **Servidores, almacenamiento y protección (Sótano):**
  * **2 Nodos de virtualización (Clúster Proxmox HA):** Dos servidores rack 2U (24 núcleos, 128 GB RAM ECC, 2 NVMe 960 GB RAID 1 para sistema, 4 NVMe 3,84 TB RAID 10 para VMs, fuente redundante y red 10GbE SFP+).
  * **Cabina NAS (24 TB útiles RAID 5, 10GbE):** Almacenamiento local rápido para copias, carpetas e ISOs PXE.
  * **SAI Online Rack (3000 VA) y Climatización Split Inverter:** Protegen el cuarto de servidores frente a cortes eléctricos y evitan el sobrecalentamiento 24/7.
  * *Justificación:* Las 8 máquinas virtuales (repartidas 4 en el Nodo 1 para web/LDAP/MariaDB y 4 en el Nodo 2 para PXE/Zabbix/OpenSearch/T-Pot) suman ~30 vCPUs y ~90 GB de RAM. Con dos servidores de 128 GB hay holgura total y alta disponibilidad (si uno cae, el otro asume todo). El enlace 10GbE al NAS permite instalar varios PCs por PXE sin cuellos de botella.

* **Red, cableado estructurado, software y personal:**
  * **Firewall y Switches:** Appliance físico **OPNsense** en rack (6x 2.5GbE + 2x SFP+ 10G), **3 switches L2/L3 24p PoE+ con 4 SFP+ 10G** (uno por planta), **1 switch 16p GbE** para la bancada del taller, **3 puntos Wi-Fi 6 PoE** y un lote de **8 transceptores SFP+ fibra y 6 cables DAC 10G**.
  * **Racks, cableado y WAN:** Rack 42U en sótano y dos murales de 12U en plantas con paneles de parcheo, fibra óptica OM4 entre plantas, cable Cat 6A LSZH, canalización, 48 tomas dobles, latiguillos RJ45 y fibra simétrica de 1 Gbps con IP fija.
  * **Software y Personal:** Se reutilizan los 22 PCs actuales y se emplea software Open Source sin coste de licencias (OPNsense, Proxmox, Linux, Rancher, Nginx, OpenLDAP, MariaDB, Zabbix, OpenSearch y T-Pot), pagando solo soporte básico de Proxmox y el respaldo en **AWS S3 / AWS Backup**. Para el despliegue inicial de tres semanas se contratan **90 h de Ingeniero de Sistemas** y **60 h de Técnico Instalador**, dejando después el mantenimiento diario a los 2 administradores en plantilla.

## 11.2. Presupuesto económico de implantación

| Categoría | Concepto | Cant. | Precio Unit. (€) | Coste Total (€) |
| :--- | :--- | :---: | :---: | :---: |
| **Servidores** | Servidor Rack 2U 24C, 128 GB RAM ECC, NVMe RAID, 10GbE | 2 | 3.850,00 | 7.700,00 |
| **Almacenamiento** | Cabina NAS Rack 4 bahías (24 TB útiles RAID 5) 10GbE | 1 | 1.350,00 | 1.350,00 |
| **Protección** | SAI Online Doble Conversión Rack 3000 VA | 1 | 890,00 | 890,00 |
| **Red y Firewall** | Appliance Hardware Rack OPNsense (6x 2.5GbE + 2x SFP+) | 1 | 720,00 | 720,00 |
| **Red Planta** | Switch Gestionable L2/L3 24p GbE (PoE+) + 4 SFP+ 10G | 3 | 440,00 | 1.320,00 |
| **Red Taller** | Switch Gestionable 16p GbE para bancada PXE / Montaje | 1 | 190,00 | 190,00 |
| **Wi-Fi** | Punto de Acceso Wi-Fi 6 PoE Empresarial (Sótano, 1.ª y 2.ª planta) | 3 | 155,00 | 465,00 |
| **Enlaces 10G** | Lote de 8 transceptores SFP+ 10GBase-SR (fibra) y 6 cables DAC 10G | 1 | 420,00 | 420,00 |
| **Climatización** | Aire acondicionado Split Inverter dedicado para cuarto CPD (sótano) | 1 | 790,00 | 790,00 |
| **Racks y Parcheo** | Rack 42U, 2 Racks 12U, PDU, paneles de parcheo Cat 6A y bandejas fibra | 1 | 1.280,00 | 1.280,00 |
| **Cableado y Tomas** | Fibra OM4, cable Cat 6A LSZH, canalización, 48 tomas y latiguillos RJ45 | 1 | 1.620,00 | 1.620,00 |
| **Software** | Suscripción anual Proxmox VE Community (por nodo físico) | 2 | 355,00 | 710,00 |
| **Cloud AWS** | Servicio anual AWS S3 Alta Redundancia + AWS Backup + Dominio | 1 | 840,00 | 840,00 |
| **Conectividad** | Fibra empresarial simétrica 1 Gbps con IP fija (anual) | 1 | 660,00 | 660,00 |
| **Personal** | Horas de Ingeniero de Sistemas y Ciberseguridad | 90 | 45,00 | 4.050,00 |
| **Personal** | Horas de Técnico Instalador de Telecomunicaciones | 60 | 30,00 | 1.800,00 |
| **Subtotal** | **Base Imponible (sin impuestos)** | — | — | **24.805,00** |
| **Impuestos** | **IVA aplicable (21 %)** | — | — | **5.209,05** |
| **Total** | **Presupuesto Total de Implantación** | — | — | **30.014,05** |

## 11.3. Financiación y viabilidad económica

* **Propuesta de financiación mixta:**
  1. **Renting tecnológico a 36 meses para el equipamiento electrónico (13.055,00 € + IVA):** Cubre servidores, NAS, SAI, firewall, switches, Wi-Fi y enlaces 10G por una cuota de **418 €/mes** (19 €/mes por trabajador), con seguro incluido y 100 % deducible en el Impuesto sobre Sociedades.
  2. **Fondos propios para instalación y servicios anuales (11.750,00 € + IVA):** Cubre climatización, racks, cableado, mano de obra y el primer año de AWS, fibra y soporte sin pagar intereses bancarios.
  3. **Ayudas públicas:** Solicitud de incentivos de digitalización y ciberseguridad para pymes de la Junta de Andalucía (Agencia TRADE) o línea ICO Empresas.

* **Valoración de la viabilidad económica:** La inversión de **24.805,00 € + IVA** es totalmente asumible y rentable para la empresa: **(1)** ahorra más de 20.000 € en licencias frente a alternativas propietarias (Windows Server, CALs o Splunk); **(2)** reutiliza los 22 ordenadores existentes y evita pagar mantenimiento externo gracias a los 2 administradores propios; **(3)** el despliegue PXE ahorra horas diarias en el taller y el portal web abre ventas 24/7; y **(4)** el aislamiento por VLANs y las copias en **AWS S3** evitan que un virus en el taller paralice la facturación.
