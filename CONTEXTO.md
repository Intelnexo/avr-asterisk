# Contexto - AVR-ASTERISK

**Última actualización:** 2025-01-XX  
**Versión:** 1.2.1  
**Mantenido por:** Andrés Zambrano B (andres@ianexo.net)

---

## Propósito

Este documento mantiene el contexto completo del proyecto `avr-asterisk` para facilitar el análisis y continuación del trabajo en futuras sesiones.

---

## Resumen del Proyecto

### Descripción
`avr-asterisk` es un contenedor Docker optimizado de Asterisk 20.9.2 diseñado para soportar cientos de llamadas concurrentes en un entorno multi-tenant SaaS.

### Características Principales
- **Asterisk 20.9.2** con módulos optimizados
- **PJSIP** para manejo moderno de SIP
- **WebRTC** soportado vía WebSocket Secure (WSS)
- **Multi-tenant** mediante archivos de configuración separados por tenant
- **Optimizado para alto volumen** (cientos de llamadas concurrentes)
- **RTP ports:** 10000-20000
- **Timezone:** UTC

---

## Arquitectura

### Estructura de Configuración

```
/etc/asterisk/
├── asterisk.conf              # Configuración principal (puede ser optimizada)
├── pjsip.conf                 # Master PJSIP (incluye tenants)
├── extensions.conf             # Master dialplan (incluye tenants)
├── queues.conf                 # Master queues (incluye tenants)
├── musiconhold.conf           # Master MOH (incluye tenants)
└── tenants/                    # Configuraciones por tenant
    ├── {tenant_id}/
    │   ├── pjsip.conf          # PJSIP endpoints del tenant
    │   ├── extensions-pjsip.conf # PJSIP endpoints para extensiones WebRTC
    │   ├── extensions.conf      # Dialplan del tenant
    │   ├── outbound-routing.conf # Routing saliente por digit patterns
    │   ├── queues.conf          # Colas del tenant
    │   └── musiconhold.conf     # MOH del tenant
```

### Multi-tenancy

**Separación por archivos:**
- Cada tenant tiene su propio directorio `/etc/asterisk/tenants/{tenant_id}/`
- Los archivos master incluyen explícitamente cada tenant (Asterisk no soporta wildcards)
- Las configuraciones se generan dinámicamente desde `avr-ami`

**Formato de nombres:**
- Trunks: `tenant-{tenant_id}-trunk-{name}`
- Extensiones: `tenant-{tenant_id}-ext-{number}`
- Grupos: `tenant-{tenant_id}-group-{name}`
- Contextos: `from-internal-{tenant_id}`, `outbound-{tenant_id}`

---

## Historial de Cambios

### Versión 1.2.0 - Optimizaciones para Alto Volumen

**Cambios:**
1. **Configuraciones optimizadas:**
   - `examples/asterisk.conf.optimized` - Configuración principal optimizada
   - `examples/my_pjsip.conf.optimized` - Configuración PJSIP optimizada

2. **Dockerfile:**
   - Aumentados límites de sistema (nofile, nproc)
   - Habilitado `app_mixmonitor` para grabaciones
   - Habilitados formatos de audio (wav, mp3)
   - Creado directorio `/etc/asterisk/tenants` para multi-tenancy

3. **README.md:**
   - Versión revertida a 20.9.2 (compatibilidad con producción)
   - Corregidos RTP ports (10000-20000)
   - Corregido timezone (UTC)
   - Agregada sección de performance y escalabilidad

**Archivos modificados:**
- `Dockerfile`
- `README.md`
- `examples/asterisk.conf.optimized` (NUEVO)
- `examples/my_pjsip.conf.optimized` (NUEVO)
- `examples/README.md` (NUEVO)

**Decisiones técnicas:**
- RTP ports ampliados a 10000-20000 para soportar más llamadas concurrentes
- Timezone UTC para consistencia global
- Límites de sistema aumentados para alto volumen
- Configuraciones optimizadas como ejemplos (no se aplican automáticamente)

---

## Configuraciones Optimizadas

### asterisk.conf.optimized

**Características principales:**
- `maxload = 0.9` - Rechaza llamadas si CPU > 90%
- `maxfiles = 65536` - Máximo de archivos abiertos
- `maxcalls = 1000` - Máximo de llamadas concurrentes
- `maxproc = 32768` - Máximo de procesos
- `minmemfree = 64` - Mínimo de memoria libre (MB)

**Uso:**
```yaml
# En docker-compose.yml
volumes:
  - ../avr-asterisk/examples/asterisk.conf.optimized:/etc/asterisk/asterisk.conf:ro
```

### my_pjsip.conf.optimized

**Características principales:**
- `user_agent = AVR-Asterisk`
- `endpoint_identifier_order = ip,username,anonymous`
- `max_forwards = 70`
- `contact_expiration_check_interval = 30`

**Uso:**
```yaml
# En docker-compose.yml
volumes:
  - ../avr-asterisk/examples/my_pjsip.conf.optimized:/etc/asterisk/my_pjsip.conf:ro
```

**Nota:** Este archivo debe ser incluido en el `pjsip.conf` master si se usa.

---

## Integración con avr-ami

### Flujo de Configuración

1. **avr-ami genera configuraciones:**
   - `generatePJSIPConfig()` - Genera `pjsip.conf` para trunks
   - `generateExtensionPJSIPConfig()` - Genera `extensions-pjsip.conf` para extensiones WebRTC
   - `generateExtensionsConfig()` - Genera `extensions.conf` para dialplan
   - `generateOutboundRoutingConfig()` - Genera `outbound-routing.conf` para routing saliente
   - `generateQueuesConfig()` - Genera `queues.conf` para colas
   - `generateMusicOnHoldConfig()` - Genera `musiconhold.conf` para MOH

2. **avr-ami actualiza archivos master:**
   - `updateMasterIncludes()` - Agrega `#include` explícitos para cada tenant

3. **avr-ami recarga Asterisk:**
   - `reloadAsteriskConfig()` - Ejecuta `module reload` vía AMI

### Sincronización

- **MinIO S3:** Configuraciones se suben a MinIO para sincronización entre contenedores
- **Al iniciar:** Si `SYNC_FROM_MINIO_ON_STARTUP=true`, descarga configuraciones de MinIO

---

## Variables de Entorno

```bash
# RTP ports
RTP_START=10000
RTP_END=20000

# Timezone
TZ=UTC

# Asterisk AMI (para avr-ami)
ASTERISK_HOST=asterisk
ASTERISK_PORT=5038
ASTERISK_USERNAME=...
ASTERISK_PASSWORD=...

# WebSocket para WebRTC
WEBSOCKET_PORT=8089
WEBSOCKET_PATH=/ws
```

---

## Puertos y Servicios

| Puerto | Protocolo | Servicio | Descripción |
|--------|-----------|----------|-------------|
| 5060 | UDP/TCP | SIP | Registro y señalización SIP |
| 5061 | TLS | SIP | SIP sobre TLS |
| 8089 | WSS | WebSocket | WebRTC vía WebSocket Secure |
| 10000-20000 | UDP | RTP | Medios de audio/video |

---

## Comandos Útiles

### Verificar Estado

```bash
# Conectarse a Asterisk CLI
docker exec -it avr-asterisk asterisk -rvvv

# Ver endpoints PJSIP
pjsip show endpoints

# Ver colas
queue show

# Ver transports
pjsip show transports

# Ver dialplan
dialplan show from-internal-{tenant_id}
```

### Verificar Configuraciones

```bash
# Ver configuración de un tenant
ls -la /etc/asterisk/tenants/{tenant_id}/

# Ver archivos master
cat /etc/asterisk/pjsip.conf | grep include
cat /etc/asterisk/extensions.conf | grep include
```

### Recargar Configuración

```bash
# Desde Asterisk CLI
module reload

# Desde avr-ami (vía AMI)
# Se ejecuta automáticamente después de generar configuraciones
```

---

## Problemas Conocidos

### Asterisk no soporta wildcards en #include

**Problema:** Asterisk no soporta `#include "tenants/*/pjsip.conf"`

**Solución:** `avr-ami` genera includes explícitos para cada tenant:
```ini
#include "tenants/{tenant_id}/pjsip.conf"
#include "tenants/{tenant_id}/extensions-pjsip.conf"
```

### Configuraciones optimizadas no se aplican automáticamente

**Problema:** Los archivos en `examples/` son solo ejemplos

**Solución:** Deben montarse explícitamente en `docker-compose.yml` o copiarse manualmente

---

## Próximos Pasos

1. **Testing:**
   - Probar con cientos de llamadas concurrentes
   - Verificar que las optimizaciones funcionan correctamente
   - Validar multi-tenancy con múltiples tenants

2. **Optimizaciones adicionales:**
   - Ajustar límites según carga real
   - Optimizar configuración de RTP según red
   - Considerar clustering para mayor escalabilidad

3. **Monitoreo:**
   - Integrar métricas de Prometheus
   - Alertas para alto uso de CPU/memoria
   - Monitoreo de llamadas concurrentes

---

## Referencias

- **README.md** - Documentación principal del proyecto
- **CHANGELOG.md** - Historial de versiones
- **examples/README.md** - Guía de uso de configuraciones optimizadas
- **CONTEXTO_PROYECTOS.md** (raíz) - Contexto general de todos los proyectos

---

**Fin del Documento de Contexto**

