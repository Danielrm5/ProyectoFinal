# 1. Presentación del Proyecto: Distributions-Líder

## 1.1. Presentación de la empresa

**Distributions-Líder** es una pequeña empresa dedicada a la distribución y reparación de equipos microinformáticos para pymes. Destaca por su flexibilidad: permite a cada cliente elegir el sistema operativo que prefiera y entrega los equipos con optimizaciones y configuraciones hechas a medida.

Se ubica en el **Polígono PIBO** (Bollullos de la Mitación), una zona empresarial muy bien comunicada con clientes y proveedores. Sus clientes principales son tiendas de informática que compran lotes para reventa, negocios de hostelería que necesitan equipos preconfigurados y medianas empresas que demandan servidores para digitalizar su gestión.

Sus instalaciones ocupan un edificio de dos plantas más un sótano destinado al almacén y al cuarto de comunicaciones. La finalidad de este proyecto es implantar una red segura y segmentada que conecte las tres plantas, automatice el montaje en el taller por red (PXE) y publique un portal web de pedidos e incidencias respaldado en la nube.

## 1.2. Organización y contexto de la empresa

La empresa cuenta con **20 empleados** repartidos en cuatro departamentos y **2 administradores técnicos** encargados de desplegar y mantener la infraestructura informática:

| Departamento | Ubicación | Nº de trabajadores | Labor principal y uso de la red |
| :--- | :--- | :---: | :--- |
| **Montaje y Reparaciones** | Primera planta | 8 | Reparación de equipos, descarga de ISOs/manuales e instalación por red (PXE). |
| **Administración Técnica** | Primera planta / CPD | 2 | Mantenimiento de servidores, red, seguridad, copias y soporte interno. |
| **Almacén y Logística** | Sótano | 4 | Control de stock de piezas y equipos con terminales y lectores Wi-Fi. |
| **Administración** | Segunda planta | 5 | Facturación, contabilidad y gestión de proveedores y clientes. |
| **Marketing** | Segunda planta | 3 | Gestión del catálogo web, pedidos online y campañas publicitarias. |

En el día a día los departamentos trabajan conectados: el taller (1.ª planta) y el almacén (sótano) comparten el inventario en tiempo real y descargan ISOs y manuales del repositorio local, mientras que en la 2.ª planta Administración y Marketing gestionan la facturación, los proveedores y las ventas. Además, la empresa necesita un **portal web** donde sus clientes empresariales puedan hacer pedidos o consultar el estado de sus reparaciones las 24 horas.

## 1.3. Análisis de las necesidades tecnológicas

| Necesidad técnica | Utilidad práctica en Distributions-Líder |
| :--- | :--- |
| **1. Separación de servicios web y red interna** | Aislar el portal web público (DMZ) para que no tenga acceso a la red del taller ni a las oficinas. |
| **2. Ciberseguridad y control del tráfico** | Proteger la facturación y evitar que equipos de clientes infectados en el taller propaguen virus. |
| **3. Servicios básicos de red (DNS y DHCP)** | Asignar IP automática a los puestos y a los PCs en reparación y resolver nombres internos. |
| **4. Gestión centralizada de usuarios y permisos** | Controlar con directorio (LDAP) qué recursos ve cada departamento (facturas, marketing o taller). |
| **5. Almacenamiento compartido** | Disponer de un repositorio central rápido para compartir ISOs, scripts, manuales y documentos. |
| **6. Aplicación web con base de datos** | Ofrecer a las pymes clientes un portal 24/7 para comprar equipos o ver el estado de sus averías. |
| **7. Acceso remoto seguro mediante VPN** | Permitir el teletrabajo seguro y la gestión remota de servidores a los 2 administradores. |
| **8. Copias de seguridad y recuperación** | Respaldar contabilidad, inventario y configuraciones frente a fallos de hardware o ataques. |
| **9. Monitorización de la infraestructura** | Supervisar el estado de los servidores y la disponibilidad de la web para evitar caídas. |
| **10. Automatización de tareas (PXE)** | Instalar sistemas operativos por red (PXE) en varios equipos a la vez, ahorrando horas en montaje. |
| **11. Servicios o recursos cloud** | Guardar una copia de seguridad externa en la nube (AWS) frente a cualquier siniestro en la nave. |
| **12. Mantenimiento y resolución de incidencias** | Garantizar que los 2 administradores mantengan la red estable sin frenar las entregas del taller. |
