# 1. Presentación del Proyecto: Distributions-Líder

## 1.1. Presentación de la empresa

**Distributions-Líder** es una pequeña empresa dedicada a la distribución y reparación de equipos microinformáticos. Lo que realmente la diferencia en su sector es su flexibilidad: permite a cada cliente elegir el sistema operativo que mejor se adapte a su negocio y entrega los equipos con optimizaciones y configuraciones hechas a medida.

La nave se encuentra en el **Polígono PIBO** (Bollullos de la Mitación), una ubicación muy bien comunicada y rodeada de otras pymes, lo que facilita el trato diario con clientes y proveedores. Su cartera de clientes abarca desde pequeñas tiendas de informática que compran lotes para reventa, hasta locales de hostelería que necesitan terminales listos para trabajar o empresas medianas que buscan renovar sus servidores y digitalizar su gestión.

El edificio cuenta con dos plantas para los departamentos técnicos y de oficina, además de un sótano destinado al almacén y al cuarto de comunicaciones. El objetivo de este proyecto es dotar a la empresa de una red moderna, segura y bien organizada que conecte las tres plantas, agilice el trabajo del taller y permita abrir un portal web de pedidos para los clientes.

## 1.2. Organización y contexto de la empresa

Actualmente la plantilla la forman **20 empleados** repartidos en cuatro departamentos, a los que se suman **2 administradores técnicos** encargados de poner en marcha y mantener toda la infraestructura informática:

| Departamento | Ubicación | Nº de trabajadores | Labor principal y uso de la red |
| :--- | :--- | :---: | :--- |
| **Montaje y Reparaciones** | Primera planta | 8 | Reparación de equipos, descarga de ISOs y manuales e instalación por red (PXE). |
| **Administración Técnica** | Primera planta / CPD | 2 | Mantenimiento de servidores, red, seguridad, copias y soporte interno. |
| **Almacén y Logística** | Sótano | 4 | Control de stock de piezas y equipos mediante terminales y lectores Wi-Fi. |
| **Administración** | Segunda planta | 5 | Facturación, contabilidad y trato con proveedores y clientes. |
| **Marketing** | Segunda planta | 3 | Gestión del catálogo web, pedidos online y campañas publicitarias. |

En el día a día los departamentos dependen unos de otros: el taller de la primera planta y el almacén del sótano consultan continuamente el inventario y utilizan el repositorio local para bajar imágenes de sistemas operativos (ISOs) y manuales de reparación. Por su parte, en la segunda planta, Administración y Marketing gestionan las compras, las facturas y la captación de clientes.

Además de que la comunicación interna sea rápida, la empresa necesita lanzar un **portal web** donde sus clientes puedan hacer pedidos, encargar servidores a medida o consultar en qué estado se encuentra la reparación de sus ordenadores.

## 1.3. Análisis de las necesidades tecnológicas

| Necesidad técnica | Utilidad práctica en Distributions-Líder |
| :--- | :--- |
| **1. Separación de servicios accesibles desde Internet y red interna** | Mantener el portal web de clientes en una zona aislada (DMZ) para que no tenga acceso a la red del taller ni a las oficinas. |
| **2. Ciberseguridad, protección de sistemas y control del tráfico** | Proteger los datos de facturación y evitar que los ordenadores de clientes infectados que entran a reparación afecten a la red. |
| **3. Servicios básicos de red (DNS y DHCP)** | Dar IP automática a los ordenadores de la plantilla y a los equipos que pasan a diario por la bancada del taller, además de resolver nombres internos. |
| **4. Gestión centralizada de usuarios, grupos, permisos y accesos** | Controlar mediante directorio (LDAP) qué puede ver cada empleado: facturación en Administración, campañas en Marketing o herramientas de montaje en el taller. |
| **5. Almacenamiento compartido** | Contar con un repositorio común y rápido donde guardar ISOs, scripts de optimización, manuales técnicos y documentos de trabajo. |
| **6. Aplicación web con base de datos** | Ofrecer a las pymes clientes una web abierta 24 horas para comprar equipos o revisar el estado de sus reparaciones. |
| **7. Acceso remoto seguro mediante VPN** | Permitir a los empleados teletrabajar de forma segura y a los 2 administradores conectarse a los servidores desde fuera si surge una urgencia. |
| **8. Copias de seguridad y recuperación de información** | Respaldar la contabilidad, el inventario y las configuraciones de clientes frente a averías de disco o posibles ataques. |
| **9. Monitorización de la infraestructura** | Vigilar que los servidores y la web de clientes estén siempre activos para detectar cualquier fallo antes de que afecte al trabajo. |
| **10. Automatización de tareas de administración** | Instalar sistemas operativos por red (PXE) de forma automática en varios ordenadores a la vez, ahorrando horas de trabajo manual en el taller. |
| **11. Servicios o recursos cloud** | Guardar una copia de seguridad externa en la nube (AWS) para que los datos críticos estén a salvo incluso si ocurre un incidente físico en la nave. |
| **12. Mantenimiento de servicios y resolución de incidencias** | Asegurar que los 2 administradores técnicos puedan mantener la red estable sin frenar el ritmo de entregas del taller ni el trabajo de oficina. |
