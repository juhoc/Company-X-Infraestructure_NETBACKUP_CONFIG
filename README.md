# Configuración Automatizada de Clientes NetBackup (Ansible AWX)

Este repositorio contiene un Playbook maestro y un Rol de Ansible diseñado para ser ejecutado desde **Ansible Automation Platform (AWX/Tower)**. Su objetivo principal es automatizar de forma masiva y estandarizada la configuración de infraestructura para clientes de NetBackup a nivel de sistema operativo y aplicativo.

## 🎯 Objetivo del Playbook

La necesidad de este desarrollo surge para automatizar la modificación crítica de archivos del cliente NetBackup y del sistema operativo (OS) durante procesos de migración o alta de servidores, asegurando que la comunicación entre el cliente y los servidores maestros/medios fluya de manera correcta a través de las redes de respaldo (IPs Robóticas).

En específico, el rol automatiza las siguientes tareas:
1. **Modificación del archivo `/etc/hosts`**: Agrega de forma estandarizada las IPs y nombres de los *Media Servers* (ej. `vxnbucfs`, `cpdnbumedia3-le`, `lvprpbckrescc01-cfs`) e inyecta la IP Robótica (segmentos `172.10`, `172.11`, `172.13` o fallback) asignando un sufijo estandarizado al nombre del cliente (`hostname_nbu`).
2. **Modificación del archivo de configuración `bp.conf`**: Se declaran los *Media Servers* principales para autorizarlos, y se estandariza el parámetro `CLIENT_NAME` aplicando el mismo sufijo.
3. **Validación de Cambios**: Ejecuta comandos nativos (`bpclntcmd`) para validar resolución y comunicación directa contra el *Master Server*.
4. **Reporte Consolidado HTML**: Genera de forma automática un Dashboard gerencial y técnico en formato HTML.

## 🚀 Beneficios del Desarrollo

- **Estandarización Absoluta:** Garantiza que todos los clientes posean la misma sintaxis de configuración, sufijos de cliente, y apuntes de *Media Servers*, reduciendo las discrepancias en el entorno.
- **Reducción a Cero del Error Humano:** Al eliminar la intervención manual de edición de archivos con editores como `vi` o `nano`, se evita la corrupción del archivo `/etc/hosts` o el crítico `bp.conf`.
- **A Prueba de Fallos (Robustez):** Incorpora validaciones inteligentes (manejo de listas vacías) que detectan automáticamente si un servidor cuenta con una IP de red de respaldos. Si no la tiene, realiza un *fallback* seguro usando la IP principal sin romper la automatización.
- **Auditoría y Trazabilidad:** 
  - Crea de manera automática **backups con *timestamps*** (fecha y época) de todos los archivos (`/etc/hosts` y `bp.conf`) antes de efectuar cualquier modificación.
  - Al término del ciclo, concentra los resultados de todos los nodos en un **Reporte HTML Consolidado** que detalla el estatus de conexión por servidor, simplificando la certificación de las migraciones.

## ⚙️ Arquitectura del Flujo (Rol `role_config_netbackup`)

El flujo se divide en tareas secuenciales (fases `s0` a `s19`):

- **Fases Iniciales (`s0-s6`):** Validación de versiones instaladas, chequeo de demonios (PBX), e identificación dinámica de las interfaces de red (IP Robótica) para tomar decisiones.
- **Fases de Modificación (`s7-s11`):** Generación de respaldos `.bak`, limpieza de registros obsoletos y orquestación de la nueva configuración mediante módulos idempotentes (`blockinfile` y `lineinfile`).
- **Fases de Validación (`s12-s18`):** Pruebas de fuego de NetBackup. Solicitud y validación de tokens de seguridad y resolución hacia el Master (`bpclntcmd -self`, `bpclntcmd -pn`).
- **Fase de Reporteo (`s19`):** Renderizado de templates Jinja2 (`consolidated_cfs_report.html.j2`) y publicación de resultados.

---

> *Desarrollo diseñado para entornos corporativos gestionados mediante flujos CI/CD o ejecución bajo demanda en plataformas Ansible AWX / Ansible Tower.*
