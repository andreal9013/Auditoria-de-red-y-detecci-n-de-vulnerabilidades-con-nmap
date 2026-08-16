# Auditoría de Red y Detección de Vulnerabilidades con Nmap

## 1. Resumen Ejecutivo
Este proyecto documenta la fase de descubrimiento, mapeo de puertos y evaluación automatizada de vulnerabilidades sobre un servidor de prueba de la red (`scanme.nmap.org`). Se evaluó la superficie de ataque expuesta utilizando **Nmap** y su motor de scripts (**NSE**), identificando servicios obsoletos (OpenSSH, Apache), vulnerabilidades de Denegación de Servicio (Slowloris), falta de controles anti-CSRF e indexación pública de directorios web.

---

## 2. Entorno y Herramientas
* **Herramienta de Escaneo:** Nmap v7.991 (Consola Windows / CMD)
* **Motor de Scripts:** Nmap Scripting Engine (NSE)
* **Objetivo Evaluado:** `scanme.nmap.org` (`45.33.32.156`)
* **Sistema Operativo Detectado:** Linux Kernel (Ubuntu)
* **Latencia Registrada:** 0.12s
* **Tiempo Total de Ejecución (NSE Scan):** 331.44 segundos (~5.5 minutos)

---

## 3. Metodología de Escaneo y Fases Ejecutadas

### Fase 1: Mapeo Inicial y Descubrimiento de Servicios (`-sV -Pn`)
Identificación de puertos TCP abiertos y detección exacta de versiones de software ejecutable sin depender de pings ICMP previos.

**Comando ejecutado:**
```bash
nmap -sV -Pn scanme.nmap.org
Resultados de la Fase 1:

Puertos Filtrados: 26 puertos TCP bloqueados/filtrados por firewall intermedio.

Puertos Cerrados: 971 puertos TCP.

Puertos Abiertos Identificados:

22/tcp - SSH (OpenSSH 6.6.1p1 Ubuntu 2ubuntu2.13)

80/tcp - HTTP (Apache httpd 2.4.7 Ubuntu)

9929/tcp - Nping-echo (Nping echo)
Fase 2: Auditoría Automatizada de Vulnerabilidades (--script vuln)
Consulta automatizada a bases de datos de vulnerabilidades conocidas (CVEs) e inspección de configuraciones en los servicios web y de administración expuestos.

Comando ejecutado:

Bash
nmap -sV --script vuln -p 22,80 scanme.nmap.org
Fase 3: Evaluación de Vectores Específicos y Fingerprinting Final
Inspección profunda realizada por el motor de scripts sobre el servicio web y el servicio SSH para validar fallos de denegación de servicio, configuraciones de seguridad y vectores de inyección.

Hallazgos de la Fase 3:

Prueba DoS Slowloris: Confirmado estado LIKELY VULNERABLE a ataques de retención de conexiones parciales HTTP.

Evaluación XSS: Ejecución del script http-dombased-xss sin detección de vulnerabilidades basadas en DOM.

Confirmación de Huella Digital: Validación final del servidor bajo la cabecera Apache/2.4.7 (Ubuntu) y sistema operativo Linux Kernel.
4. Matriz de Hallazgos y Evaluación de Riesgos
Puerto / Servicio,Hallazgo / Vulnerabilidad,Identificador / CVE,Clasificación de Riesgo,Impacto Técnico
22/tcp (SSH),Software desactualizado,Múltiples CVEs (CVSS 9.8 - 10.0),Crítico,Riesgo de Ejecución Remota de Código (RCE) y enumeración de usuarios en OpenSSH 6.6.1p1.
80/tcp (HTTP),Vulnerabilidad DoS Slowloris,CVE-2007-6750,Alto,Agotamiento de hilos del servidor mediante el mantenimiento de peticiones HTTP parciales.
80/tcp (HTTP),Formularios Ausentes de Token Anti-CSRF,http-csrf (/search/),Medio,Vulnerabilidad a Cross-Site Request Forgery en formularios de búsqueda.
80/tcp (HTTP),Exposición de Directorios (Directory Listing),http-enum (/images/),Bajo / Medio,Exposición pública e indexación de archivos y carpetas internas en el servidor web.
80/tcp (HTTP),Evaluación XSS basada en DOM,http-dombased-xss,Informativo (Seguro),No se detectaron vectores de ataque XSS basados en DOM.
5.Recomendaciones de Mitigación (Hardening)
Gestión de Parches y Actualizaciones (Patch Management):

Actualizar OpenSSH a la versión estable más reciente y deshabilitar métodos de autenticación obsoletos o basados en contraseña débil.

Actualizar Apache HTTP Server de la versión 2.4.7 a la última versión disponible del fabricante.

Mitigación contra Denegación de Servicio (Slowloris):

Configurar e implementar el módulo mod_reqtimeout en Apache para establecer límites estrictos de tiempo de espera en la recepción de encabezados y cuerpos de peticiones HTTP.

Bastionado Web (Web Hardening):

Desactivar la indexación automática de directorios mediante la directiva Options -Indexes en la configuración de Apache.

Implementar tokens anti-CSRF únicos y aleatorios por sesión en todos los formularios interactivos de la aplicación.
