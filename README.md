# 🛡️ Despliegue de Firewall Permitral con pfSense

Este manual, documenta el proceso técnico para la implementación de un firewall como psSense, desde la descarga y preparación de la imagen hasta la configuración de la arquitectura de red en VirtualBox


---

## 🛠️ Especificaciones Técnicas
Para asegurar que se puede replicar este laboratorio, se detallan las versiones y recursos que se han usado:
* **Hipervisor:** Oracle VirtualBox 7.x
* **Sistema Operativo:** pfSense Community Edition (v2.8.1+)
* **Arquitectura:** AMD64 (64-bit)
* **Recursos VM:** 4GB RAM | 1 vCPU | 30GB VDI (Dinámico)
* **Interfaces de Red:** WAN (NAT) | DMZ (Internal Network) |  LAN (Internal Network)

--- 

## 📂 Fase 00: Arquitectura y Diseño de Red
Antes de proceder con la implementación técnica, se define el esquema lógico de la infraestructura. El diseño se basa en una arquitectura de tres niveles (Tiered Architecture) para garantizar el aislamiento de servicios críticos.

![Esquema de Red](./00-esquema-de-red/01-esquema-de-red.png)

### Detalle de Segmentación:
* **Zona WAN (eth1):** Interfaz conectada al mundo exterior (NAT en VirtualBox) para salida a internet.
* **Zona DMZ (eth2 - 10.0.0.0/24):** Segmento destinado a servidores que ofrecen servicios externos, como el **Servidor Ubuntu**.
* **Red LAN (eth3 - 192.168.50.0/24):** Zona privada de alta confianza donde se ubica el **CLIENTE-LAN-1**.

---


## 📂 Fase 01: Obtención y Preparación de la Imagen
El proceso comienza con la adquisición de la imagen oficial, asegurando la integridad del software de seguridad perimetral a través de canales oficiales.

### Paso 1.1: Registro y Acceso al Portal
Se accede al portal oficial de **Netgate Store**. Aunque es una versión comunitaria, el flujo de trabajo requiere el registro del instalador para obtener los enlaces de descarga actualizados.

![Acceso a la web](01-descarga-iso/01-página%20web.png)

### Paso 1.2: Selección de la Arquitectura
Se configura el instalador seleccionando la arquitectura **AMD64 (64-bit)** y el tipo de instalador **DVD Image (ISO)**, garantizando la compatibilidad con el entorno de virtualización.

![Selección de imagen](01-descarga-iso/03-seleccion-iso-añadir-card.png)

### Paso 1.3: Proceso de Checkout y Descarga
Se completa el proceso de obtención a coste cero ($0.00). Una vez validado el pedido, se habilita la descarga del paquete comprimido.

![Descarga del instalador](01-descarga-iso/07-descargar.png)

### Paso 1.4: Preparación del Medio (Descompresión)
El archivo descargado se presenta en formato comprimido `.iso.gz`. Es imperativo realizar la extracción mediante herramientas como 7-Zip para obtener la imagen `.iso` final.

![Extracción de ISO](01-descarga-iso/10-extraer.png)


---

## 📂 Fase 02: Configuración de la Máquina Virtual (VirtualBox)
En esta etapa se define el hardware virtual necesario para soportar un firewall de alto rendimiento. La correcta asignación de recursos y la segmentación de redes son los pilares de este laboratorio.

### Paso 2.1: Resumen de Configuración de la VM
Se ha creado una instancia optimizada con los siguientes parámetros técnicos, asegurando que el instalador de pfSense tenga acceso a todos los segmentos de red requeridos:

* **Sistema Operativo:** FreeBSD (64-bit).
* **Memoria RAM:** 4096 MB (Asignación generosa para asegurar fluidez en el procesado de paquetes).
* **Almacenamiento:** Disco VDI de 30 GB con la ISO oficial montada.
* **Segmentación de Red (Tiered Architecture):**
    * **Adaptador 1:** WAN (Configurado como NAT para salida a internet).
    * **Adaptador 2:** DMZ (Red interna para servicios expuestos).
    * **Adaptador 3:** LAN (Red interna aislada para clientes locales).

![Resumen de Hardware Virtual](./02-configuracion-virtualbox/01-resumen-hardware.png)

---

## 📂 Fase 03: Instalación del Sistema (pfSense CE)
En esta fase se realiza la configuración lógica de las interfaces y el particionado del disco duro virtual bajo el sistema de archivos ZFS.

### 3.1: Configuración de Interfaces
El instalador reconoce las tarjetas de red virtuales (em0, em1, em2). Se procede con la vinculación física-lógica.
![Instalación Interfaces](./03-instalacion-sistema/03-network-installation.png)

### 3.2: Asignación WAN (em0)
Se designa la interfaz `em0` como WAN, configurada en modo DHCP para recibir conectividad del NAT de VirtualBox.
![Asignación WAN](./03-instalacion-sistema/04-asignacion-wan.png)
![Modo WAN DHCP](./03-instalacion-sistema/05-configuracion-modo-wan.png)

### 3.3: Asignación LAN (em2)
Siguiendo el diseño de red personalizado, se asigna la interfaz `em2` como LAN primaria.
![Asignación LAN](./03-instalacion-sistema/06-asignacion-lan.png)
![Confirmación Interfaces](./03-instalacion-sistema/08-confirmacion-interfaces.png)

### 3.4: Selección de Versión y Sistema de Archivos
Se selecciona la versión **Community Edition (CE)** y el sistema de archivos **ZFS** con esquema de particiones **GPT**.
![Selección CE](./03-instalacion-sistema/10-seleccion-version-ce.png)
![Opciones ZFS](./03-instalacion-sistema/11-opciones-instalacion-zfs.png)

### 3.5: Particionado y Selección de Disco
Se configura un pool ZFS tipo **Stripe** (sin redundancia). 
> **Nota Crítica:** Es obligatorio marcar el disco `ada0` con la tecla **Espacio** hasta ver el asterisco `[*]`.
![ZFS Stripe](./03-instalacion-sistema/12-zfs-stripe.png)
![Selección Disco ada0](./03-instalacion-sistema/13-seleccion-disco-zfs.png)

### 3.6: Ejecución y Reinicio
Confirmación final de escritura y proceso de instalación de paquetes.
![Confirmación Final](./03-instalacion-sistema/14-confirmacion-destruccion-datos.png)
![Reinicio Final](./03-instalacion-sistema/18-reboot-final.png)


--- 

## 📂 Fase 04: Finalización y Desmontaje de Medios
Fase crítica para romper el bucle de instalación y asegurar el arranque desde el disco duro virtual.

### 4.1: Desmontaje de la ISO en VirtualBox
Inmediatamente después de pulsar **Reboot**, se debe retirar la imagen ISO desde el menú del hipervisor para evitar que el instalador cargue nuevamente.
* **Ruta:** Dispositivos > Unidades Ópticas > Eliminar disco de la unidad virtual.
![Desmontaje ISO](./04-finalizacion-desmontaje/01-desmontaje-iso-virtualbox.png)

### 4.2: Primer Arranque del Kernel
Verificación de la secuencia de inicio de FreeBSD y carga de los servicios core de pfSense desde el pool ZFS.
![Secuencia de Inicio](./04-finalizacion-desmontaje/02-arranque-servicios.png)


--- 

## 📂 Fase 05: Configuración Operativa de Red (Consola)
En esta etapa final de consola, el firewall deja de tener una configuración estándar para adaptarse al esquema de red segmentado del laboratorio.

### 5.1: Acceso al Menú de Direccionamiento
Tras el primer arranque limpio, seleccionamos la **Opción 2 (Set interface(s) IP address)**. Este es el punto de entrada para definir manualmente las IPs de nuestra infraestructura.
![Selección Opción 2](./05-configuracion-inicial-consola/01-menu-consola-pfsense.png)

### 5.2: Configuración del Segmento LAN (192.168.50.1)
Se selecciona la interfaz **2 (LAN)**. Para garantizar el control total del segmento, se deniega la configuración por DHCP externo y se establece una IP estática.
![Selección Interfaz LAN](./05-configuracion-inicial-consola/03-seleccion-interfaz-lan.png)

Se aplica la IP **192.168.50.1** con máscara **/24**. Adicionalmente, se activa el **Servidor DHCP interno** (rango .20 - .100) y se mantiene el protocolo **HTTPS** para la WebGUI por seguridad.
![Configuración Estática y DHCP LAN](./05-configuracion-inicial-consola/05-configuracion-dhcp-lan.png)
![Verificación IP LAN](./05-configuracion-inicial-consola/06-verificacion-ip-lan-final.png)

### 5.3: Asignación y Configuración de la DMZ (OPT1)
Para habilitar el tercer adaptador de red (em1), se utiliza la **Opción 1 (Assign Interfaces)**. Se realiza el mapeo lógico vinculando **em1** como la interfaz **Optional 1**.
![Acceso Asignación](./05-configuracion-inicial-consola/07-acceso-asignacion-interfaces.png)
![Mapeo em1 a OPT1](./05-configuracion-inicial-consola/08-mapeo-interfaces-dmz.png)
![Confirmación Mapeo](./05-configuracion-inicial-consola/09-confirmacion-mapeo-tres-interfaces.png)

Una vez asignada, se utiliza nuevamente la **Opción 2** para otorgar a la **DMZ** la dirección **10.0.0.1/24** y activar su propio servidor DHCP.
![Selección Interfaz DMZ](./05-configuracion-inicial-consola/10-seleccion-interfaz-dmz.png)
![Configuración IP y DHCP DMZ](./05-configuracion-inicial-consola/11-configuracion-ip-dhcp-dmz.png)
![Éxito Configuración DMZ](./05-configuracion-inicial-consola/12-confirmacion-exito-dmz.png)

### 5.4: Estado Final de la Infraestructura
La configuración por consola concluye con éxito. El firewall presenta ahora sus tres zonas de red operativas y listas para la gestión mediante la interfaz web desde un cliente interno.
![Estado Final Consola](./05-configuracion-inicial-consola/13-estado-final-consola.png)


--- 

## 📂 Fase 06: Pruebas de Conectividad y Verificación
En esta fase se valida la correcta segmentación de la red y la operatividad de los servicios DHCP y Gateway desde los clientes finales, asegurando que el tráfico fluye según el diseño establecido.

### 6.1: Verificación de Direccionamiento IP (ipconfig)
Se inicia un cliente en la red LAN para comprobar que el servidor DHCP de pfSense ha asignado correctamente una dirección dentro del rango configurado (192.168.50.x). Se valida la puerta de enlace predeterminada.
![Verificación de IP](./06-pruebas-conectividad/01-ipconfig.png)

### 6.2: Prueba de Latencia a Gateway LAN
Se ejecuta un test de conectividad (`ping`) hacia la interfaz LAN del firewall (**192.168.50.1**). Esta prueba confirma que el enlace físico-lógico entre el cliente y el segmento de gestión es estable.
![Ping Gateway LAN](./06-pruebas-conectividad/02-ping-lan-gateway.png)

### 6.3: Validación de Salto a Gateway DMZ
Se realiza una prueba de alcance hacia la interfaz de la DMZ (**10.0.0.1**). El éxito de esta respuesta indica que el enrutamiento interno entre segmentos está activo y el kernel de pfSense procesa correctamente las peticiones ICMP.
![Ping Gateway DMZ](./06-pruebas-conectividad/03-ping-dmz-gateway.png)

### 6.4: Acceso a la Interfaz Gráfica (WebGUI)
Finalmente, se valida el acceso al panel de administración mediante el navegador del cliente. La carga del Dashboard confirma que los servicios de servidor web y seguridad de pfSense están listos para la configuración avanzada. (usuario admin y contraseña pfsense)
![Acceso WebGUI](./06-pruebas-conectividad/04-pfsense-webgui.png)

### 6.5: Dashboard de Estado del Sistema
Una vez autenticados, se accede al panel principal (Dashboard). En esta vista se verifica el estado de los servicios, el uso de recursos de hardware y que las tres interfaces (**WAN, LAN y DMZ**) se encuentran en estado "Up" con sus respectivas direcciones IP asignadas.
![Dashboard Principal](./06-pruebas-conectividad/05-dashboard-pfsense.png)
