# Changelog - AVR-ASTERISK

Todos los cambios notables de este proyecto serán documentados en este archivo.

El formato está basado en [Keep a Changelog](https://keepachangelog.com/es-ES/1.0.0/),
y este proyecto adhiere a [Semantic Versioning](https://semver.org/lang/es/).

## [1.2.0] - 2025-01-XX

### Resumen
Optimizaciones para soportar alto volumen de llamadas (cientos concurrentes) y compatibilidad completa con sistema multi-tenant.

**Autor inicial:** AVR (Agent Voice Response)  
**Actualizaciones:** Andrés Zambrano B (andres@ianexo.net)

---

### Added - Agregado

#### Configuraciones Optimizadas
- ✨ `examples/asterisk.conf.optimized` - Configuración principal optimizada para alto volumen
  - Límites de recursos: `maxfiles = 65536`, `maxcalls = 1000`, `maxproc = 32768`
  - Control de CPU: `maxload = 0.9` (rechaza llamadas si CPU > 90%)
  - Control de memoria: `minmemfree = 64` (MB mínimo libre)
  - Optimizaciones de rendimiento para alta concurrencia
- ✨ `examples/my_pjsip.conf.optimized` - Configuración global de PJSIP optimizada
  - Configuración de user agent
  - Orden de identificación de endpoints
  - Configuración de forwards y timeouts
  - Optimizaciones de rendimiento para PJSIP
- ✨ `examples/README.md` - Documentación sobre cómo usar las configuraciones optimizadas

#### Dockerfile
- ✨ Directorio `/etc/asterisk/tenants` creado para configuraciones multi-tenant
- ✨ Aumento de límites del sistema:
  - `nofile = 65536` (archivos abiertos)
  - `nproc = 32768` (procesos)
- ✨ Includes automáticos en archivos maestros:
  - `#include "my_extensions.conf"` en `extensions.conf`
  - `#include "my_pjsip.conf"` en `pjsip.conf`
  - `#include "my_manager.conf"` en `manager.conf`
  - `#include "my_queues.conf"` en `queues.conf`
  - `#include "my_ari.conf"` en `ari.conf`

#### README
- ✨ Sección "Performance and Scalability" con:
  - Estimaciones de capacidad (100-1000+ llamadas)
  - Optimizaciones del sistema (sysctls)
  - Configuración de Docker Compose para recursos
  - Optimizaciones de configuración de Asterisk
- ✨ Tabla de puertos por defecto
- ✨ Tabla de variables de entorno
- ✨ Sección de troubleshooting

---

### Changed - Cambiado

#### Dockerfile
- 🔄 RTP port range corregido: 10000-20000 (antes estaba limitado a 10000-10050)
- 🔄 Timezone corregido: UTC (antes estaba documentado como Europe/Rome)
- 🔄 Habilitación de AMI Manager Interface
- 🔄 Habilitación de Prometheus metrics
- 🔄 Habilitación de HTTP/HTTPS API

#### README
- 🔄 Versión de Asterisk revertida a: 20.9.2 (compatibilidad con producción)
- 🔄 RTP port range actualizado: 10000-20000
- 🔄 Timezone actualizado: UTC
- 🔄 Documentación expandida con secciones detalladas

---

### Fixed - Corregido

- 🐛 RTP port range limitado incorrectamente - Corregido a 10000-20000
- 🐛 Timezone documentado incorrectamente - Corregido a UTC
- 🐛 Falta de directorio para configuraciones multi-tenant - Agregado `/etc/asterisk/tenants`

---

### Security - Seguridad

- 🔒 Límites del sistema aumentados para prevenir DoS
- 🔒 Configuraciones optimizadas incluyen ajustes de seguridad

---

### Documentation - Documentación

- 📚 README actualizado con información completa
- 📚 Ejemplos de configuraciones optimizadas documentados
- 📚 Guía de uso de configuraciones optimizadas en Docker Compose

---

## [1.1.0] - Versión Anterior

### Added
- Configuración inicial de Asterisk
- Dockerfile básico
- README inicial

---

## Notas de Uso

### Configuraciones Optimizadas

Las configuraciones optimizadas son **opcionales** pero **recomendadas** para:
- Alto volumen de llamadas (100+ concurrentes)
- Optimización de recursos del sistema
- Mejor rendimiento general

**No se configuran por cliente**, son configuraciones **globales** del sistema que aplican a todos los tenants automáticamente.

### Compatibilidad Multi-Tenant

Las optimizaciones son **100% compatibles** con el sistema multi-tenant:
- `asterisk.conf` es independiente y no se modifica por el sistema multi-tenant
- `my_pjsip.conf` se incluye **antes** de las configuraciones por tenant
- Las configuraciones por tenant se agregan después sin conflictos

### Orden de Inclusión

```
[global] (desde my_pjsip.conf)
#include "my_pjsip.conf" (optimizaciones globales)
#include "tenants/{tenant_id}/pjsip.conf" (configuraciones por tenant)
```

---

## Referencias

- [Keep a Changelog](https://keepachangelog.com/es-ES/1.0.0/)
- [Semantic Versioning](https://semver.org/lang/es/)
- [Asterisk Documentation](https://docs.asterisk.org/)

