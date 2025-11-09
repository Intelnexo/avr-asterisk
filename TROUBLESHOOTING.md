# Troubleshooting - AVR-ASTERISK

**Última actualización:** 2025-01-XX  
**Versión:** 1.2.0

---

## Problemas Comunes

### 1. Asterisk no inicia

**Síntomas:**
- Contenedor se reinicia constantemente
- Logs muestran errores de configuración
- No responde a comandos

**Diagnóstico:**
```bash
# Ver logs del contenedor
docker logs avr-asterisk

# Verificar configuración
docker exec -it avr-asterisk asterisk -rvvv

# Verificar archivos de configuración
docker exec -it avr-asterisk ls -la /etc/asterisk/
```

**Soluciones:**
1. **Error de sintaxis en configuración:**
   ```bash
   # Verificar sintaxis de archivos .conf
   asterisk -rx "config show errors"
   ```

2. **Archivos faltantes:**
   ```bash
   # Verificar que archivos master existen
   ls -la /etc/asterisk/asterisk.conf
   ls -la /etc/asterisk/pjsip.conf
   ```

3. **Permisos incorrectos:**
   ```bash
   # Verificar permisos
   ls -la /etc/asterisk/
   # Deben ser legibles por usuario asterisk
   ```

---

### 2. Endpoints PJSIP no aparecen

**Síntomas:**
- `pjsip show endpoints` no muestra endpoints del tenant
- Extensiones no se registran

**Diagnóstico:**
```bash
# Verificar que archivo existe
ls -la /etc/asterisk/tenants/{tenant_id}/pjsip.conf

# Verificar que está incluido en master
grep -r "tenants/{tenant_id}/pjsip.conf" /etc/asterisk/pjsip.conf

# Verificar contenido del archivo
cat /etc/asterisk/tenants/{tenant_id}/pjsip.conf
```

**Soluciones:**
1. **Archivo no incluido en master:**
   ```bash
   # Verificar que pjsip.conf master incluye el archivo
   # Debe tener: #include "tenants/{tenant_id}/pjsip.conf"
   ```

2. **Sintaxis incorrecta:**
   ```bash
   # Verificar sintaxis del archivo
   asterisk -rx "config show errors"
   ```

3. **Recargar configuración:**
   ```bash
   # Recargar módulo PJSIP
   asterisk -rx "module reload res_pjsip.so"
   ```

---

### 3. WebRTC no funciona

**Síntomas:**
- Extensiones WebRTC no se registran
- Llamadas WebRTC fallan

**Diagnóstico:**
```bash
# Verificar transport WSS
pjsip show transports

# Verificar endpoints WebRTC
pjsip show endpoints | grep wss

# Verificar puerto WebSocket
netstat -tulpn | grep 8089
```

**Soluciones:**
1. **Transport WSS no configurado:**
   ```bash
   # Verificar que transport-wss está en pjsip.conf
   # Debe estar en archivo master o en configuración del tenant
   ```

2. **Puerto bloqueado:**
   ```bash
   # Verificar que puerto 8089 está abierto
   # Verificar firewall
   ```

3. **Certificados SSL:**
   ```bash
   # Verificar que certificados SSL están configurados para WSS
   # Verificar ruta de certificados
   ```

---

### 4. Llamadas no se conectan

**Síntomas:**
- Llamadas se inician pero no se conectan
- Audio no funciona

**Diagnóstico:**
```bash
# Verificar canales activos
asterisk -rx "core show channels"

# Verificar RTP
asterisk -rx "rtp set debug on"
# Ver logs para ver tráfico RTP

# Verificar codecs
pjsip show endpoint tenant-{tenant_id}-ext-{number}
```

**Soluciones:**
1. **Codecs no coinciden:**
   ```bash
   # Verificar que codecs están configurados en ambos endpoints
   # Deben tener al menos un codec común (ulaw, alaw, opus)
   ```

2. **RTP bloqueado:**
   ```bash
   # Verificar que puertos RTP (10000-20000) están abiertos
   # Verificar firewall y NAT
   ```

3. **Direct media:**
   ```bash
   # Verificar configuración de direct_media
   # Para WebRTC debe ser: direct_media=no
   ```

---

### 5. Colas no funcionan

**Síntomas:**
- Llamadas no entran a cola
- Música en espera no funciona

**Diagnóstico:**
```bash
# Verificar colas
queue show

# Verificar configuración de cola
cat /etc/asterisk/tenants/{tenant_id}/queues.conf

# Verificar que está incluida en master
grep -r "tenants/{tenant_id}/queues.conf" /etc/asterisk/queues.conf
```

**Soluciones:**
1. **Cola no incluida en master:**
   ```bash
   # Verificar que queues.conf master incluye el archivo
   # Debe tener: #include "tenants/{tenant_id}/queues.conf"
   ```

2. **Música en espera no configurada:**
   ```bash
   # Verificar que musiconhold.conf está configurado
   # Verificar que musicclass en queues.conf es correcto
   ```

3. **Recargar configuración:**
   ```bash
   # Recargar módulo de colas
   asterisk -rx "module reload app_queue.so"
   ```

---

### 6. Alto uso de CPU/Memoria

**Síntomas:**
- CPU > 90%
- Memoria agotada
- Llamadas se rechazan

**Diagnóstico:**
```bash
# Verificar uso de recursos
docker stats avr-asterisk

# Verificar llamadas activas
asterisk -rx "core show channels"

# Verificar configuración de límites
cat /etc/asterisk/asterisk.conf | grep max
```

**Soluciones:**
1. **Aplicar configuraciones optimizadas:**
   ```bash
   # Montar asterisk.conf.optimized en docker-compose.yml
   # Reiniciar contenedor
   ```

2. **Ajustar límites:**
   ```bash
   # Aumentar maxcalls, maxfiles, maxproc según recursos disponibles
   # Verificar que Docker tiene recursos suficientes
   ```

3. **Reducir carga:**
   ```bash
   # Verificar llamadas activas
   # Considerar distribuir carga entre múltiples instancias
   ```

---

## Comandos Útiles

### Verificar Estado

```bash
# Estado general
asterisk -rx "core show version"
asterisk -rx "core show settings"

# Canales activos
asterisk -rx "core show channels"

# Endpoints PJSIP
pjsip show endpoints
pjsip show endpoint tenant-{tenant_id}-ext-{number}

# Colas
queue show
queue show tenant-{tenant_id}-group-{name}

# Transports
pjsip show transports
```

### Debugging

```bash
# Habilitar logs detallados
asterisk -rvvv

# Debug de PJSIP
pjsip set logger on
pjsip set logger levels 0,1,2,3,4,5,6,7

# Debug de RTP
rtp set debug on

# Ver errores de configuración
asterisk -rx "config show errors"
```

### Recargar Configuración

```bash
# Recargar todo
asterisk -rx "module reload"

# Recargar módulo específico
asterisk -rx "module reload res_pjsip.so"
asterisk -rx "module reload app_queue.so"
```

---

## Verificación de Configuración

### Checklist de Inicio

- [ ] Asterisk inicia sin errores
- [ ] Archivos master existen y son válidos
- [ ] Directorio `/etc/asterisk/tenants/` existe
- [ ] Transport WSS está configurado
- [ ] Puertos RTP están abiertos (10000-20000)
- [ ] Configuraciones optimizadas están montadas (si se usan)
- [ ] Logs no muestran errores críticos

### Verificar Multi-tenancy

```bash
# Verificar que tenant tiene archivos
ls -la /etc/asterisk/tenants/{tenant_id}/

# Verificar que están incluidos en master
grep -r "{tenant_id}" /etc/asterisk/pjsip.conf
grep -r "{tenant_id}" /etc/asterisk/extensions.conf
grep -r "{tenant_id}" /etc/asterisk/queues.conf
```

---

## Referencias

- **CONTEXTO.md** - Contexto completo del proyecto
- **README.md** - Documentación principal
- **examples/README.md** - Guía de configuraciones optimizadas
- **CONTEXTO_PROYECTOS.md** (raíz) - Contexto general

---

**Fin del Documento de Troubleshooting**

