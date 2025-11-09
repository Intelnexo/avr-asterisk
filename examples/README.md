# Configuration Examples for High Call Volume

This directory contains optimized configuration examples for Asterisk to support hundreds of concurrent calls.

## Files

- `asterisk.conf.optimized` - Optimized Asterisk main configuration
- `my_pjsip.conf.optimized` - Optimized PJSIP global configuration

## Usage

### 1. asterisk.conf.optimized

Mount this file to `/etc/asterisk/asterisk.conf` in the container:

```bash
docker run -d \
  --name asterisk \
  -v $(pwd)/examples/asterisk.conf.optimized:/etc/asterisk/asterisk.conf:ro \
  agentvoiceresponse/avr-asterisk:latest
```

Or in `docker-compose.yml`:

```yaml
services:
  asterisk:
    image: agentvoiceresponse/avr-asterisk:latest
    volumes:
      - ./examples/asterisk.conf.optimized:/etc/asterisk/asterisk.conf:ro
```

### 2. my_pjsip.conf.optimized

Mount this file to `/etc/asterisk/my_pjsip.conf` in the container:

```bash
docker run -d \
  --name asterisk \
  -v $(pwd)/examples/my_pjsip.conf.optimized:/etc/asterisk/my_pjsip.conf:ro \
  agentvoiceresponse/avr-asterisk:latest
```

Or in `docker-compose.yml`:

```yaml
services:
  asterisk:
    image: agentvoiceresponse/avr-asterisk:latest
    volumes:
      - ./examples/my_pjsip.conf.optimized:/etc/asterisk/my_pjsip.conf:ro
```

## Important Notes

1. **Multi-tenant Compatibility**: These optimizations are **fully compatible** with the multi-tenant system:
   - `asterisk.conf` is independent and not modified by the multi-tenant system
   - `my_pjsip.conf` is included **before** tenant-specific configurations
   - Tenant configurations in `/etc/asterisk/tenants/{tenant_id}/pjsip.conf` are added after

2. **File Order**: The include order in `pjsip.conf` is:
   ```
   [global] (from my_pjsip.conf)
   ... (global settings)
   #include "my_pjsip.conf" (includes global optimizations)
   #include "tenants/{tenant_id}/pjsip.conf" (tenant-specific configs)
   ```

3. **No Conflicts**: These optimizations do **not** conflict with tenant-specific configurations because:
   - `asterisk.conf` is a separate file
   - `my_pjsip.conf` only contains global settings
   - Tenant configurations are in separate files

## Customization

You can customize these files based on your needs:

- **CPU Limits**: Adjust `maxload` in `asterisk.conf`
- **Memory Limits**: Adjust `minmemfree` in `asterisk.conf`
- **Max Calls**: Adjust `maxcalls` in `asterisk.conf`
- **PJSIP Settings**: Adjust global settings in `my_pjsip.conf`

## Testing

After applying these configurations:

1. **Check Asterisk Status**:
   ```bash
   docker exec -it asterisk asterisk -rx "core show version"
   ```

2. **Check Active Calls**:
   ```bash
   docker exec -it asterisk asterisk -rx "core show channels"
   ```

3. **Monitor Resources**:
   ```bash
   docker stats asterisk
   ```

## Support

For issues and questions:
- GitHub Issues: https://github.com/agentvoiceresponse/avr-asterisk/issues
- Documentation: https://github.com/agentvoiceresponse/avr-asterisk

