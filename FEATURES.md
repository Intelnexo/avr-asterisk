# Features Técnicos - AVR-ASTERISK

**Última actualización:** 2025-01-XX  
**Versión:** 1.2.0  
**Mantenido por:** Andrés Zambrano B (andres@ianexo.net)

---

## Resumen Ejecutivo

AVR-ASTERISK es un servidor de telefonía IP basado en Asterisk 20.9.2, optimizado para entornos multi-tenant SaaS con soporte para cientos de llamadas concurrentes. Incluye configuraciones optimizadas para alto rendimiento, grabación de llamadas, routing avanzado y integración completa con WebRTC.

---

## Features Técnicos Principales

### 1. Asterisk 20.9.2
- **Versión:** Asterisk 20.9.2 (LTS)
- **Stack SIP:** PJSIP (moderno y eficiente)
- **Protocolos soportados:**
  - SIP (UDP/TCP/TLS)
  - WebRTC (WebSocket Secure)
  - RTP (Real-time Transport Protocol)
- **Codecs de audio:**
  - Opus (recomendado para WebRTC)
  - G.711 (ulaw/alaw)
  - G.722
  - G.729 (requiere licencia)

### 2. Arquitectura Multi-Tenant
- **Configuración por tenant:**
  - Archivos de configuración separados por `tenant_id`
  - Directorio `/etc/asterisk/tenants/{tenant_id}/`
  - Contextos SIP separados por tenant
- **Aislamiento de datos:**
  - Configuraciones independientes por tenant
  - Sin interferencia entre tenants
  - Escalabilidad horizontal

### 3. WebRTC Support
- **WebSocket Secure (WSS):**
  - Puerto: 8089 (configurable)
  - Path: `/ws` (configurable)
  - Certificados SSL/TLS requeridos
- **Endpoints PJSIP:**
  - Configuración automática por extensión
  - Soporte para múltiples extensiones por tenant
  - Registro automático de extensiones

### 4. Grabación de Llamadas
- **MixMonitor:**
  - Grabación eficiente de llamadas
  - Soporte para formatos WAV y MP3
  - Eventos AMI para inicio/fin de grabación
- **Configuración:**
  - Habilitación por trunk o extensión
  - Variables de entorno para UUID y metadatos
  - Integración con S3 para almacenamiento

### 5. Routing Avanzado
- **Outbound Routing:**
  - Routing por patrones de dígitos
  - Asociación de patrones con troncos específicos
  - Soporte para múltiples troncos por tenant
- **Inbound Routing:**
  - Routing por DID (Direct Inward Dialing)
  - Contextos personalizados por tenant
  - Redirección a extensiones o grupos

### 6. Colas de Llamadas
- **Asterisk Queues:**
  - Estrategias de distribución:
    - `ringall` - Suena a todos
    - `leastrecent` - Menos reciente
    - `fewestcalls` - Menos llamadas
    - `random` - Aleatorio
  - Configuración por grupo de extensiones
  - Timeout configurable por grupo
  - `maxlen` basado en canales concurrentes

### 7. Music on Hold (MOH)
- **Fuentes:**
  - URLs de S3 (HTTP mode)
  - Archivos locales (fallback)
  - Configuración por trunk
- **Clases personalizadas:**
  - Clase por tenant: `default-{tenant_id}`
  - Música personalizada por cliente

---

## Concurrencia y Performance

### 1. Capacidad de Llamadas Concurrentes

#### Configuración Optimizada
- **Máximo de llamadas:** 1000 (configurable)
- **Máximo de callers:** 1000 (configurable)
- **File descriptors:** 65536
- **Procesos:** 32768
- **Threads:** 1000

#### Estimaciones de Capacidad
- **Servidor básico (2 CPU, 4GB RAM):**
  - ~100 llamadas concurrentes
  - ~200 extensiones registradas
- **Servidor medio (4 CPU, 8GB RAM):**
  - ~300 llamadas concurrentes
  - ~500 extensiones registradas
- **Servidor alto rendimiento (8 CPU, 16GB RAM):**
  - ~500-1000 llamadas concurrentes
  - ~1000+ extensiones registradas

### 2. Optimizaciones del Sistema

#### Límites del Sistema
```ini
# /etc/security/limits.conf
* soft nofile 65536
* hard nofile 65536
* soft nproc 32768
* hard nproc 32768
```

#### Sysctls de Red
```bash
# Optimizaciones de red para alto volumen
net.core.somaxconn = 4096
net.ipv4.tcp_max_syn_backlog = 4096
net.ipv4.ip_local_port_range = 10000 65535
```

### 3. Configuración de Asterisk Optimizada

#### asterisk.conf.optimized
```ini
[options]
maxload = 0.9                    # Máximo 90% CPU
maxfiles = 65536                 # File descriptors
minmemfree = 64                  # 64 MB mínimo libre
maxcalls = 1000                  # Máximo de llamadas
maxcallers = 1000               # Máximo de callers
transmit_silence = yes          # Transmitir silencio
transmit_silence_hold = yes     # Transmitir silencio en hold
```

#### my_pjsip.conf.optimized
```ini
[global]
user_agent = AVR-Asterisk
endpoint_identifier_order = ip,username,anonymous
max_forwards = 70
contact_expiration_check_interval = 30
```

### 4. RTP Port Range
- **Rango:** 10000-20000
- **Total de puertos:** 10000 puertos
- **Capacidad teórica:** ~5000 llamadas simultáneas (2 puertos por llamada)

### 5. Timezone
- **Configuración:** UTC (Universal Time Coordinated)
- **Consistencia:** Todas las fechas/horas en UTC
- **Logs:** Timestamps en UTC

---

## Seguridad

### 1. Autenticación y Autorización

#### PJSIP Endpoints
- **Autenticación SIP:**
  - Username/password por extensión
  - Realm por tenant
  - Validación de credenciales
- **Registro SIP:**
  - Expiración configurable (default: 300s)
  - Re-registro automático
  - Validación de contactos

#### AMI (Asterisk Manager Interface)
- **Autenticación:**
  - Username/password requeridos
  - Permisos por usuario
  - Acceso restringido por IP (opcional)
- **Puerto:** 5038 (configurable)
- **Protocolo:** TCP

### 2. Encriptación

#### TLS para SIP
- **Puerto:** 5061 (TLS)
- **Certificados:** SSL/TLS requeridos
- **Cifrado:** TLS 1.2+

#### WebSocket Secure (WSS)
- **Puerto:** 8089 (configurable)
- **Certificados:** SSL/TLS requeridos
- **Cifrado:** TLS 1.2+
- **Path:** `/ws` (configurable)

### 3. Protección de Red

#### Firewall
- **Puertos abiertos:**
  - 5060 (SIP UDP/TCP)
  - 5061 (SIP TLS)
  - 8089 (WSS)
  - 10000-20000 (RTP)
- **Restricciones:**
  - Solo IPs autorizadas para AMI
  - Rate limiting en firewall (opcional)

#### Rate Limiting
- **SIP:**
  - Límite de registros por IP
  - Límite de INVITEs por IP
  - Protección contra spam
- **RTP:**
  - Validación de paquetes RTP
  - Protección contra flooding

### 4. Logging y Auditoría

#### Logs Estructurados
- **Ubicación:** `/var/log/asterisk/`
- **Rotación:** Diaria
- **Retención:** 30 días (configurable)
- **Niveles:**
  - ERROR: Errores críticos
  - WARNING: Advertencias
  - NOTICE: Eventos importantes
  - DEBUG: Información de depuración

#### Eventos AMI
- **Eventos críticos:**
  - Registros de extensiones
  - Inicio/fin de llamadas
  - Errores de autenticación
  - Cambios de estado

### 5. Aislamiento Multi-Tenant

#### Separación de Configuraciones
- **Directorios:** `/etc/asterisk/tenants/{tenant_id}/`
- **Contextos:** `from-internal-{tenant_id}`, `outbound-{tenant_id}`
- **Endpoints:** `tenant-{tenant_id}-ext-{number}`
- **Sin interferencia:** Configuraciones completamente aisladas

#### Validación de Rutas
- **Llamadas internas:**
  - Solo extensiones del mismo tenant
  - Validación de contexto
- **Llamadas salientes:**
  - Solo troncos del mismo tenant
  - Validación de patrones de routing

---

## Stack Tecnológico

### Core
- **Asterisk:** 20.9.2
- **PJSIP:** Incluido en Asterisk
- **Base OS:** Debian/Ubuntu (Docker)

### Módulos Principales
- **app_mixmonitor:** Grabación de llamadas
- **format_wav:** Formato WAV
- **format_mp3:** Formato MP3
- **res_pjsip:** Stack SIP moderno
- **res_pjsip_transport_websocket:** WebSocket support

### Integraciones
- **AMI:** Asterisk Manager Interface
- **ARI:** Asterisk REST Interface (opcional)
- **S3:** Almacenamiento de grabaciones
- **MySQL:** Metadatos (vía avr-ami)

---

## Escalabilidad

### Horizontal
- **Múltiples instancias:**
  - Load balancing de SIP
  - Distribución de llamadas
  - Sincronización de configuraciones (vía avr-ami)
- **Consideraciones:**
  - Sticky sessions para WebRTC
  - Sincronización de estado
  - Compartir grabaciones (S3)

### Vertical
- **Recursos:**
  - CPU: Múltiples cores recomendados
  - RAM: 4GB mínimo, 16GB+ recomendado
  - Disco: SSD recomendado para logs
  - Red: 1Gbps+ para alto volumen

---

## Monitoreo y Métricas

### Métricas Clave
- **Llamadas activas:** `asterisk.active_calls`
- **Extensiones registradas:** `asterisk.registered_endpoints`
- **CPU usage:** `asterisk.cpu_usage`
- **Memoria:** `asterisk.memory_usage`
- **RTP streams:** `asterisk.rtp_streams`

### Health Checks
- **AMI:** Conexión a puerto 5038
- **PJSIP:** Estado de endpoints
- **RTP:** Disponibilidad de puertos

---

## Referencias

- **README.md** - Documentación principal
- **CONTEXTO.md** - Contexto completo del proyecto
- **TROUBLESHOOTING.md** - Guía de troubleshooting
- **CHANGELOG.md** - Historial de versiones

---

**Fin del Documento de Features**

