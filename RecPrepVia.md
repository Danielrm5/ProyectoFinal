## 11. Recursos y viabilidad económica

El presente apartado detalla el dimensionamiento físico, humano y financiero requerido para llevar a producción la infraestructura proyectada para la empresa **Distributions-Líder**, dedicada a la distribución y reparación de equipos informáticos para pymes y ubicada en el Parque Industrial y de Servicios del Aljarafe (Polígono PIBO, en Bollullos de la Mitación, Sevilla). Aunque la validación técnica se realiza mediante virtualización, el presupuesto responde a una instalación física real distribuida en el edificio de la compañía (sótano, planta baja y primera planta) para dar servicio a sus 20 empleados (repartidos en Montaje y Reparaciones, Almacén y Logística, Administración, y Marketing) y a sus 2 administradores técnicos.

### 11.1. Descripción y justificación de los recursos necesarios

#### Servidores físicos y respaldo eléctrico

* **Nodos de virtualización (Clúster Principal)**: Dos servidores en formato rack 2U equipados cada uno con procesador AMD EPYC / Intel Xeon de 24 núcleos, 128 GB de memoria RAM DDR5 ECC registrada, dos discos SSD NVMe Enterprise de 960 GB en RAID 1 para el hipervisor, cuatro discos SSD NVMe U.3 de 3,84 TB en RAID 10 para alojar las máquinas virtuales y contenedores, fuente de alimentación redundante y tarjeta de red dual 10GbE SFP+.
* **Almacenamiento local y servidor de despliegue (NAS / Repositorio PXE)**: Una cabina NAS en formato rack de 4 bahías equipada con 4 discos duros Enterprise de 8 TB en RAID 5 (24 TB útiles) e interfaz 10GbE, destinada a almacenar copias de seguridad locales rápidas, imágenes ISO para el despliegue por red (PXE) del departamento de Montaje y Reparaciones, y carpetas compartidas de facturación e inventario.
* **Sistema de Alimentación Ininterrumpida (SAI)**: Una unidad SAI Online de doble conversión en rack de 3000 VA / 2700 W ubicada en el cuarto de comunicaciones del sótano para estabilizar la corriente y proteger los servidores frente a cortes eléctricos.

**Justificación**: La arquitectura diseñada contempla una alta densidad de servicios simultáneos distribuidos en cuatro subredes (VLAN 10 DMZ con Rancher y Nginx Proxy Manager; VLAN 20 Interna con LDAP y automatización; VLAN 30 con el stack pesado de Zabbix, OpenSearch y T-Pot; y VLAN 40 con MariaDB). En particular, el indexador SIEM **OpenSearch** (192.168.30.20), el honeypot **T-Pot** (192.168.30.30) y el clúster **Rancher** (192.168.10.20) demandan un elevado volumen de memoria RAM, cómputo e IOPS de disco. Contar con dos nodos de 128 GB de RAM permite repartir esta carga de forma holgada y garantizar alta disponibilidad (HA) si un servidor falla. Por su parte, la conectividad a 10 Gbps entre los nodos y el NAS evita cuellos de botella cuando el taller de Montaje instala sistemas operativos en múltiples equipos simultáneamente por red (PXE).

#### Infraestructura de red y cableado estructurado

* **Cortafuegos perimetral dedicado (OPNsense Shield)**: Un appliance hardware en formato rack con 6 puertos 2.5GbE y 2 puertos 10GbE SFP+ para ejecutar **OPNsense (Shield)**, actuando como núcleo de segmentación entre las puertas de enlace de cada red (`192.168.10.1`, `192.168.20.1`, `192.168.30.1` y `192.168.40.1`), servidor VPN seguro e interfaz hacia AWS Backup.
* **Conmutación distribuida por plantas**: Tres switches gestionables L2/L3 de 24 puertos Gigabit Ethernet y 4 puertos SFP+ de 10 Gbps (uno por planta: sótano, planta baja y primera planta) para propagar las VLANs 10, 20, 30 y 40, más un switch adicional de 16 puertos Gigabit en el banco de trabajo del departamento de Montaje y Reparaciones para la conexión masiva de equipos en pruebas y arranque PXE.
* **Cobertura inalámbrica corporativa**: Tres puntos de acceso Wi-Fi 6 empresariales alimentados por PoE (uno por planta, reforzando la zona de Almacén y Logística para el uso de lectores de inventario y terminales móviles).
* **Armarios rack y cableado estructurado**: Un armario rack principal de 42U en el sótano (CPD) y dos armarios murales de 12U para las plantas superiores, unidos mediante enlaces troncales verticales (*backbone*) de fibra óptica multimodo OM4, junto con cableado horizontal Cat 6A libre de halógenos (LSZH), paneles de parcheo y 48 tomas de red dobles (para los 22 puestos fijos y las bancadas de reparación).
* **Conectividad WAN**: Línea de fibra óptica empresarial simétrica de 1 Gbps con IP pública estática (`198.51.100.45`) contratada con un operador con cobertura en el Polígono PIBO.

**Justificación**: Tal como establece el planteamiento del proyecto, los 20 equipos de los empleados y las 2 estaciones de los administradores técnicos (`192.168.20.30`) ya existen y se reutilizan íntegramente. La división del edificio en dos plantas y sótano exige descentralizar la conmutación con un switch por nivel unido por fibra óptica para evitar tiradas de cobre excesivas y garantizar velocidad de 10 Gbps desde el taller de reparaciones hasta los servidores de imágenes ISO y bases de datos. Aislar físicamente el cortafuegos OPNsense en un appliance dedicado asegura que el tráfico entre las VLANs y el cifrado de la VPN no penalicen el procesador de los servidores de aplicaciones.

#### Software, licencias y servicios cloud

* **Plataforma de virtualización y sistemas base**: Hipervisor Proxmox VE con suscripción básica anual para los dos nodos físicos, ejecutando distribuciones Linux de código abierto (Debian / Ubuntu Server) sin coste de licencia.
* **Pila de aplicaciones y ciberseguridad (Open Source)**: Despliegue sin coste de licenciamiento de **OPNsense** (cortafuegos y VPN), **Rancher** (`192.168.10.20`) para orquestación de contenedores del portal web de ventas y seguimiento, **Nginx Proxy Manager** (`192.168.10.30`), **OpenLDAP** (`192.168.20.20`), **MariaDB** (`192.168.40.10`), **Zabbix** (`192.168.30.10`), **OpenSearch** (`192.168.30.20`) y **T-Pot** (`192.168.30.30`).
* **Servicios Cloud (Amazon Web Services) y Dominio**: Suscripción anual a **AWS S3 con alta redundancia** e integración mediante **AWS Backup** para el respaldo externo automatizado y cifrado de las bases de datos de facturación, inventarios y configuraciones críticas (estimado en 3 TB de almacenamiento y transferencia), más el registro anual del dominio público para el portal de clientes.

### 11.2. Presupuesto económico de implantación

A continuación se detalla el presupuesto estimado con precios de mercado actuales para la puesta en marcha de la infraestructura en el edificio de Distributions-Líder:

| Categoría | Concepto | Cant. | Precio Unit. (€) | Coste Total (€) |
| :--- | :--- | :---: | :---: | :---: |
| **Servidores** | Servidor Rack 2U 24C, 128 GB RAM ECC, NVMe RAID, 10GbE | 2 | 3.850,00 | 7.700,00 |
| **Almacenamiento** | Cabina NAS Rack 4 bahías (32 TB brutos / 24 TB RAID 5) 10GbE | 1 | 1.350,00 | 1.350,00 |
| **Protección** | SAI Online Doble Conversión Rack 3000 VA | 1 | 890,00 | 890,00 |
| **Red y Firewall** | Appliance Hardware Rack OPNsense (6x 2.5GbE + 2x SFP+) | 1 | 720,00 | 720,00 |
| **Red Planta** | Switch Gestionable L2/L3 24p GbE (PoE+) + 4 SFP+ 10G | 3 | 440,00 | 1.320,00 |
| **Red Taller** | Switch Gestionable 16p GbE para bancada PXE / Montaje | 1 | 190,00 | 190,00 |
| **Wi-Fi** | Punto de Acceso Wi-Fi 6 PoE Empresarial (Sótano y 2 plantas) | 3 | 155,00 | 465,00 |
| **Racks y Cable** | Rack 42U (sótano), 2 Racks 12U (plantas), PDU y bandejas | 1 | 1.120,00 | 1.120,00 |
| **Racks y Cable** | Enlace fibra OM4 entre plantas, cable Cat 6A LSZH y 48 tomas | 1 | 1.380,00 | 1.380,00 |
| **Software** | Suscripción anual Proxmox VE Community (por nodo físico) | 2 | 355,00 | 710,00 |
| **Cloud AWS** | Servicio anual AWS S3 Alta Redundancia + AWS Backup + Dominio | 1 | 840,00 | 840,00 |
| **Conectividad** | Fibra empresarial simétrica 1 Gbps con IP fija (anual) | 1 | 660,00 | 660,00 |
| **Personal** | Horas de Ingeniero de Sistemas y Ciberseguridad | 90 | 45,00 | 4.050,00 |
| **Personal** | Horas de Técnico Instalador de Telecomunicaciones | 60 | 30,00 | 1.800,00 |
| **Subtotal** | **Base Imponible (sin impuestos)** | — | — | **23.195,00** |
| **Impuestos** | **IVA aplicable (21 %)** | — | — | **4.870,95** |
| **Total** | **Presupuesto Total de Implantación** | — | — | **28.065,95** |


**Justificación**: El diseño aprovecha herramientas de software libre de nivel empresarial para cubrir los 12 requisitos tecnológicos sin incurrir en costes de licencias por usuario (CAL) o por núcleo. La única partida recurrente de servicios digitales se destina a la nube de Amazon (**AWS S3 / AWS Backup**) y al soporte estable del hipervisor, garantizando que, ante un incendio, robo o fallo catastrófico en la nave del Polígono PIBO, Distributions-Líder pueda restaurar sus facturas, inventarios y portal web desde AWS en cuestión de minutos.

#### Personal técnico de implantación

* **Ingeniero de Sistemas y Ciberseguridad (90 horas)**: Encargado de configurar el clúster de virtualización, implementar las reglas de segmentación en OPNsense, desplegar Rancher, Nginx Proxy Manager, MariaDB y LDAP, automatizar el servidor PXE y los scripts de administración, y poner en marcha el SIEM OpenSearch, Zabbix, T-Pot y la conexión con AWS S3.
* **Técnico Instalador de Telecomunicaciones (60 horas)**: Responsable del montaje de los tres armarios rack en el sótano y las dos plantas, tendido de fibra vertical entre plantas, cableado de las tomas de los cuatro departamentos y del taller de montaje, y certificación de la red.

**Justificación**: Aunque la plantilla cuenta con 2 administradores técnicos que asumirán el mantenimiento diario una vez entregada la infraestructura, la envergadura del despliegue inicial (cableado de tres niveles y configuración desde cero de cuatro VLANs con SIEM, honeypot y orquestador de contenedores) requiere apoyo externo especializado durante tres semanas para no paralizar el soporte técnico ordinario de la empresa.


### 11.3. Financiación y viabilidad económica

#### Propuesta de financiación

Para que **Distributions-Líder** acometa esta modernización tecnológica manteniendo intacta su capacidad de compra de stock informático y componentes, se plantea un esquema de **financiación combinada**:

1. **Renting tecnológico para el equipamiento electrónico (12.635,00 € + IVA)**: Los dos servidores principales, la cabina NAS, el SAI, el appliance OPNsense, los cuatro switches y los tres puntos de acceso se financian mediante un contrato de *renting* tecnológico a 36 meses. Esto supone una cuota mensual fija aproximada de **405 €/mes**, que incluye seguro a todo riesgo frente a averías eléctricas, permite renovar el equipamiento al tercer año y resulta 100 % deducible como gasto en el Impuesto sobre Sociedades.
2. **Fondos propios para adecuación del edificio, puesta en marcha y servicios anuales (10.560,00 € + IVA)**: La instalación de los armarios y el cableado entre el sótano y las dos plantas, los honorarios de implantación y el primer año de AWS S3, fibra y soporte se abonan con la tesorería corriente de la empresa, evitando pagar intereses bancarios por mano de obra o servicios en la nube de pago por uso.
3. **Subvenciones y líneas de crédito industrial**: Por su ubicación en el Polígono PIBO (Bollullos de la Mitación) y su condición de pyme de 22 trabajadores, la empresa puede acogerse a los incentivos de digitalización y ciberseguridad de la Junta de Andalucía (Agencia TRADE) o financiar el pago inicial mediante una línea ICO Empresas a tipo bonificado.

#### Valoración de la viabilidad económica

El coste total de **23.195,00 € de base imponible (28.065,95 € con IVA)** es altamente razonable, rentable y asumible para una empresa del volumen de **Distributions-Líder** (22 trabajadores en plantilla y actividad comercial B2B para otras pymes) por cuatro razones clave:

* **Ahorro masivo en licencias privativas**: Desplegar soluciones comerciales equivalentes para un SIEM (como Splunk o QRadar en lugar de OpenSearch), monitorización, orquestación de contenedores (Rancher), directorio de usuarios (Active Directory con 22 CALs y licencias Windows Server Datacenter) y firewall de pago elevaría el presupuesto por encima de los 45.000 €. La arquitectura Open Source planteada reduce el coste a la mitad sin sacrificar prestaciones.
* **Reutilización del parque informático existente**: No es necesario adquirir los 20 ordenadores de los departamentos ni las 2 estaciones de trabajo de los administradores técnicos, ni tampoco pagar a una consultora externa por el mantenimiento mensual posterior, ya que los 2 administradores en plantilla gestionarán la red una vez implantada.
* **Aumento directo de la productividad en el taller y las ventas**: La automatización del despliegue por red (PXE) multiplica la velocidad a la que el departamento de Montaje y Reparaciones prepara y repara ordenadores para clientes, mientras que el portal web alojado en la DMZ (VLAN 10 con Rancher y Nginx Proxy Manager) abre un canal de ventas y seguimiento 24/7 que incrementa la facturación de la empresa.
* **Protección frente a paradas críticas y ciberataques**: Al trabajar con equipos en reparación potencialmente infectados en el taller, la separación estricta mediante OPNsense, la vigilancia activa con Zabbix, OpenSearch y el honeypot T-Pot (VLAN 30), y el respaldo externo en **AWS S3** blindan la facturación y la base de datos general (`192.168.40.10`), asegurando la continuidad del negocio desde el primer día.
