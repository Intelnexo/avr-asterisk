# Estructura Funcional - AVR-ASTERISK

**Última actualización:** 2025-01-20  
**Versión:** 1.2.1  
**Mantenido por:** Andrés Zambrano B (andres@ianexo.net)

---

## 📋 Índice

1. [Resumen Ejecutivo](#resumen-ejecutivo)
2. [Arquitectura y Configuración](#arquitectura-y-configuración)
3. [Módulos y Features Habilitados](#módulos-y-features-habilitados)
4. [Interfaces y APIs](#interfaces-y-apis)
5. [Configuración de Archivos](#configuración-de-archivos)
6. [Integración con ARI](#integración-con-ari)
7. [Variables de Entorno](#variables-de-entorno)
8. [Puertos y Servicios](#puertos-y-servicios)

---

## 🎯 Resumen Ejecutivo

**AVR-ASTERISK** es un contenedor Docker optimizado de **Asterisk 20.9.2** diseñado para soportar cientos de llamadas concurrentes en un entorno multi-tenant SaaS. Incluye configuraciones optimizadas para alto rendimiento, grabación de llamadas, routing avanzado y integración completa con WebRTC.

### Características Principales
- ✅ **Asterisk 20.9.2** (LTS) compilado desde fuente
- ✅ **PJSIP** - Stack SIP moderno y eficiente
- ✅ **WebRTC** - Soporte vía WebSocket Secure (WSS)
- ✅ **AudioSocket** - Channel driver para streaming de audio en tiempo real
- ✅ **Multi-tenant** - Configuraciones separadas por tenant
- ✅ **Optimizado para alto volumen** - Cientos de llamadas concurrentes
- ✅ **RTP Ports:** 10000-20000 (10,000 puertos UDP)
- ✅ **Timezone:** UTC

---

## 🏗️ Arquitectura y Configuración

### Estructura de Directorios

```
/etc/asterisk/
├── asterisk.conf              # Configuración principal
├── pjsip.conf                 # Master PJSIP (incluye tenants)
├── extensions.conf            # Master dialplan (incluye tenants)
├── queues.conf                # Master queues (incluye tenants)
├── musiconhold.conf           # Master MOH (incluye tenants)
├── manager.conf               # AMI configuration
├── ari.conf                   # ARI configuration
├── http.conf                  # HTTP/HTTPS API configuration
├── rtp.conf                   # RTP settings (ports 10000-20000)
├── prometheus.conf            # Prometheus metrics
└── tenants/                   # Configuraciones por tenant
    ├── {tenant_id}/
    │   ├── pjsip.conf          # PJSIP endpoints del tenant
    │   ├── extensions-pjsip.conf # PJSIP endpoints para extensiones WebRTC
    │   ├── extensions.conf    # Dialplan del tenant
    │   ├── outbound-routing.conf # Routing saliente por digit patterns
    │   ├── queues.conf         # Colas del tenant
    │   └── musiconhold.conf    # MOH del tenant
```

### Multi-tenancy

**Separación por archivos:**
- Cada tenant tiene su propio directorio `/etc/asterisk/tenants/{tenant_id}/`
- Los archivos master incluyen explícitamente cada tenant
- Contextos SIP separados por tenant: `from-internal-{tenant_id}`, `outbound-{tenant_id}`
- Aislamiento completo de configuraciones

---

## 🔌 Módulos y Features Habilitados

### Módulos Core de Asterisk

#### 1. **res_pjsip.so** - Stack SIP Moderno
- **Propósito:** Manejo de señalización SIP
- **Features:**
  - Endpoints PJSIP
  - Transports: UDP, TCP, TLS, WSS
  - Autenticación SIP (auth sections)
  - AORs (Address of Records)
  - NAT traversal (force_rport, rewrite_contact)
  - RTP symmetric
  - DTMF modes: rfc4733, inband, info, auto

#### 2. **res_pjsip_transport_websocket.so** - WebSocket Support
- **Propósito:** Soporte para WebRTC vía WebSocket
- **Puerto:** 8089 (HTTPS/WSS)
- **Path:** `/ws` (configurable)
- **Protocolo:** WebSocket Secure (WSS)

#### 3. **pbx_config.so** - Dialplan Engine
- **Propósito:** Procesamiento de dialplan
- **Archivos:** `extensions.conf`
- **Features:**
  - Extensiones por número telefónico
  - Routing a agentes (AudioSocket)
  - Control de canales concurrentes (GROUP/GROUP_COUNT)
  - Contexts configurables

#### 4. **app_mixmonitor.so** - Grabación de Llamadas
- **Propósito:** Grabación de llamadas
- **Formatos:** WAV, MP3
- **Integración:** S3 para almacenamiento

#### 5. **format_wav.so** - Formato WAV
- **Propósito:** Soporte para audio WAV

#### 6. **format_mp3.so** - Formato MP3
- **Propósito:** Soporte para audio MP3

#### 7. **res_rtp_asterisk.so** - RTP Engine
- **Propósito:** Manejo de media RTP
- **Puertos:** 10000-20000 (UDP)
- **Codecs:** ulaw, alaw, speex, opus

#### 8. **chan_audiosocket.so** - AudioSocket Channel Driver
- **Propósito:** Streaming de audio en tiempo real
- **Uso:** Integración con avr-core para agentes de voz

---

## 🌐 Interfaces y APIs

### 1. **ARI (Asterisk REST Interface)**

#### Configuración
- **Puerto HTTP:** 8088
- **Puerto HTTPS:** 8089
- **Path:** `/ari`
- **Autenticación:** Username/Password
- **Configuración:** `ari.conf`

#### Endpoints Principales
- `GET /ari/asterisk/info` - Información de Asterisk
- `POST /ari/asterisk/reloadModule` - Recargar módulo
- `GET /ari/endpoints` - Listar endpoints
- `GET /ari/channels` - Listar canales activos
- `GET /ari/bridges` - Listar bridges

#### Uso en AVR-APP-BACKEND
- **URL:** `http://avr-asterisk:8088/ari` (configurable via `ARI_URL`)
- **Usuario:** `avr` (configurable via `ARI_USERNAME`)
- **Password:** `u4lyvcPyQ19hwJKy` (configurable via `ARI_PASSWORD`)
- **Librería:** `ari-client` (Node.js)

#### Funciones Utilizadas
- `ari.asterisk.reloadModule({ moduleName })`
  - Recarga módulos sin reiniciar Asterisk
  - Módulos recargados:
    - `res_pjsip.so` - Para cambios en PJSIP
    - `pbx_config.so` - Para cambios en dialplan

---

### 2. **AMI (Asterisk Manager Interface)**

#### Configuración
- **Puerto:** 5038 (TCP)
- **Autenticación:** Username/Password
- **Configuración:** `manager.conf`

#### Uso
- Control de llamadas
- Eventos en tiempo real
- Integración con avr-ami

---

### 3. **HTTP/HTTPS API**

#### Configuración
- **Puerto HTTP:** 8088
- **Puerto HTTPS:** 8089
- **Configuración:** `http.conf`

#### Endpoints
- `/metrics` - Prometheus metrics
- `/ari/*` - ARI endpoints

---

## 📝 Configuración de Archivos

### 1. **pjsip.conf** - Configuración PJSIP

#### Transports
```
[transport-udp]
type=transport
protocol=udp
bind=0.0.0.0:5060

[transport-tcp]
type=transport
protocol=tcp
bind=0.0.0.0:5060

[transport-tls]
type=transport
protocol=tls
bind=0.0.0.0:5061

[transport-wss]
type=transport
protocol=wss
bind=0.0.0.0:8089
```

#### Endpoints (Generados Dinámicamente)
- **Phones:** `[phoneNumber]` - Endpoints para extensiones
- **Trunks:** `[trunkId]` - Endpoints para troncales SIP

---

### 2. **extensions.conf** - Dialplan

#### Contexts
- `[demo]` o `[TENANT]` - Context principal (configurable)
- `[from-internal-1]` - Context para extensiones internas
- `[outbound-{tenant}]` - Context para llamadas salientes

#### Extensiones Generadas
- **Numbers:** Extensiones por número telefónico → routing a agentes
- **Trunk Dialplan:** Extensiones para control de canales concurrentes

---

### 3. **ari.conf** - ARI Configuration

```ini
[general]
enabled = yes
pretty = yes
allowed_origins = *
auth_realm = Asterisk
```

---

### 4. **manager.conf** - AMI Configuration

```ini
[general]
enabled = yes
port = 5038
bindaddr = 0.0.0.0

[avr]
secret = avr
deny = 0.0.0.0/0.0.0.0
permit = 0.0.0.0/0.0.0.0
read = all
write = all
```

---

### 5. **http.conf** - HTTP/HTTPS API

```ini
[general]
enabled = yes
bindaddr = 0.0.0.0
bindport = 8088
prefix = asterisk
```

---

### 6. **rtp.conf** - RTP Configuration

```ini
[general]
rtpstart = 10000
rtpend = 20000
```

---

## 🔄 Integración con ARI (Desde AVR-APP-BACKEND)

### Conexión ARI
- **URL:** `http://avr-asterisk:8088/ari` (default)
- **Usuario:** `avr` (default)
- **Password:** `u4lyvcPyQ19hwJKy` (default)
- **Librería:** `ari-client` (Node.js)

### Operaciones Realizadas

#### 1. Recarga de Módulos
```javascript
await ari.asterisk.reloadModule({ moduleName: 'res_pjsip.so' });
await ari.asterisk.reloadModule({ moduleName: 'pbx_config.so' });
```

#### 2. Validación de Endpoints
- **Método:** Docker exec + CLI commands
- **Comando:** `asterisk -rx "pjsip show endpoint {id}"`
- **Propósito:** Verificar estado de conexión de endpoints

---

## 🔧 Variables de Entorno

### Configuración Base
- `TZ` - Timezone (default: `UTC`)

### Configuración de Contenedor
- `ASTERISK_CONFIG_PATH` - Ruta base de configuración (default: `/app/asterisk`)
- `ASTERISK_CONTAINER_NAME` - Nombre del contenedor (default: `avr-asterisk`)
- `DOCKER_SOCKET_PATH` - Socket de Docker (default: `/var/run/docker.sock`)

### ARI (Desde Backend)
- `ARI_URL` - URL de ARI (default: `http://avr-asterisk:8088/ari`)
- `ARI_USERNAME` - Usuario ARI (default: `avr`)
- `ARI_PASSWORD` - Password ARI (default: `u4lyvcPyQ19hwJKy`)

### Context y Tenant
- `TENANT` - Context de Asterisk (default: `demo`)
- `SIP_PORT` - Puerto SIP para contact en AOR (default: `5060`)

---

## 🔌 Puertos y Servicios

| Puerto | Protocolo | Servicio | Descripción |
|--------|-----------|----------|-------------|
| 5038 | TCP | AMI | Asterisk Manager Interface |
| 5060 | UDP/TCP | SIP | Señalización SIP |
| 5061 | TLS | SIP | Señalización SIP segura |
| 8088 | HTTP | HTTP API | API REST HTTP + ARI |
| 8089 | HTTPS | HTTPS API | API REST HTTPS + WSS |
| 10000-20000 | UDP | RTP | Media streaming (audio/video) |

---

## 📊 Capacidad y Performance

### Estimaciones de Capacidad

| Concurrent Calls | CPU | RAM | Notes |
|-----------------|-----|-----|-------|
| 100-300 | 2 cores | 2 GB | Light to medium load |
| 300-500 | 4 cores | 4 GB | Medium to high load |
| 500-1000+ | 8+ cores | 8+ GB | High load, production |

### Optimizaciones Incluidas

#### Sistema
- File descriptors: 65536
- Procesos: 32768
- Threads: 1000
- Max calls: 1000 (configurable)

#### Red
- `net.core.somaxconn = 4096`
- `net.ipv4.tcp_max_syn_backlog = 4096`
- `net.ipv4.ip_local_port_range = 10000 65535`

---

## 🔒 Seguridad

### Autenticación
- **SIP:** Username/password por endpoint
- **AMI:** Username/password + permisos
- **ARI:** Username/password

### Encriptación
- **TLS:** Para SIP (puerto 5061)
- **WSS:** Para WebRTC (puerto 8089)
- **SRTP:** Para media streams

### Aislamiento Multi-Tenant
- Configuraciones separadas por tenant
- Contextos SIP aislados
- Sin interferencia entre tenants

---

## 📈 Monitoreo

### Prometheus Metrics
- **Endpoint:** `http://localhost:8088/metrics`
- **Métricas:** Llamadas activas, extensiones registradas, CPU, memoria, RTP streams

### Health Checks
- **AMI:** Conexión a puerto 5038
- **PJSIP:** Estado de endpoints
- **RTP:** Disponibilidad de puertos

---

## 🔗 Integraciones

### Con AVR-APP-BACKEND
- **ARI:** Recarga de módulos, validación de endpoints
- **Docker:** Ejecución de comandos CLI
- **Configuración:** Generación dinámica de archivos de configuración

### Con AVR-AMI
- **AMI:** Control de llamadas, eventos
- **Configuración:** Sincronización de configuraciones

### Con AVR-CORE
- **AudioSocket:** Streaming de audio en tiempo real
- **Routing:** Llamadas a agentes de voz

---

## 📚 Referencias

- **README.md** - Documentación principal
- **FEATURES.md** - Features técnicos detallados
- **CONTEXTO.md** - Contexto completo del proyecto
- **TROUBLESHOOTING.md** - Guía de troubleshooting
- **CHANGELOG.md** - Historial de versiones

---

**Fin del Documento de Estructura Funcional**

