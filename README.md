# AVR Asterisk Docker Image

This is a lightweight Asterisk 22.4.0 Docker image optimized for VoIP applications. The image is based on Ubuntu 22.04 and includes only essential modules and features, making it ideal for production environments with minimal resource requirements.

## Features

- **Asterisk 22.4.0** - Latest stable version compiled from source
- **PJSIP support** - Full PJSIP stack for modern SIP communication
- **AudioSocket** - Channel driver for real-time audio streaming integration
- **Manager API (AMI)** - Enabled on port 5038 for call control
- **HTTP/HTTPS API** - Enabled on ports 8088 (HTTP) and 8089 (HTTPS)
- **Prometheus metrics** - Built-in metrics endpoint for monitoring
- **SRTP support** - Secure RTP for encrypted media streams
- **Minimal footprint** - Only essential modules enabled for reduced size and attack surface
- **Multi-stage build** - Optimized Docker image with minimal runtime dependencies

## Quick Start

### Using Docker

```bash
docker run -d \
  --name asterisk \
  -p 5038:5038 \
  -p 8088:8088 \
  -p 8089:8089 \
  -p 10000-20000:10000-20000/udp \
  -v /path/to/your/config:/etc/asterisk \
  -e TZ=UTC \
  agentvoiceresponse/avr-asterisk:latest
```

### Using Docker Compose

```yaml
version: '3.8'

services:
  asterisk:
    image: agentvoiceresponse/avr-asterisk:latest
    container_name: asterisk
    ports:
      - "5038:5038"   # Manager API (AMI)
      - "8088:8088"   # HTTP API
      - "8089:8089"   # HTTPS API (TLS)
      - "10000-20000:10000-20000/udp"  # RTP ports for media streaming
    volumes:
      - ./config:/etc/asterisk
    environment:
      - TZ=UTC
    restart: unless-stopped
```

## Configuration

The container uses the following default configuration files:
- `extensions.conf`
- `pjsip.conf`
- `manager.conf`
- `queues.conf`
- `ari.conf`

You can override these configurations by mounting your own configuration files to `/etc/asterisk/` in the container.

### Default Ports

| Port | Protocol | Service | Description |
|------|----------|---------|-------------|
| 5038 | TCP | AMI | Asterisk Manager Interface for call control |
| 8088 | HTTP | HTTP API | REST API for Asterisk configuration and management |
| 8089 | HTTPS | HTTPS API | Secure REST API with TLS encryption |
| 10000-20000 | UDP | RTP | Media streaming ports for audio/video |

## Environment Variables

| Variable | Default | Description |
|----------|---------|-------------|
| `TZ` | `UTC` | Timezone for the container (e.g., `UTC`, `Europe/Rome`, `America/New_York`) |

## Configuration Files

The container includes default configuration files and supports custom configurations via volume mounts:

### Default Configuration Files

- `extensions.conf` - Dialplan configuration
- `pjsip.conf` - PJSIP endpoint and transport configuration
- `manager.conf` - AMI user and permissions
- `queues.conf` - Queue configuration
- `ari.conf` - Asterisk REST Interface configuration
- `rtp.conf` - RTP settings (configured for ports 10000-20000)
- `http.conf` - HTTP/HTTPS API configuration
- `prometheus.conf` - Prometheus metrics configuration

### Custom Configuration

You can override default configurations by mounting your own files to `/etc/asterisk/`. The container automatically includes files with the `my_` prefix:

- `my_extensions.conf` - Custom dialplan
- `my_pjsip.conf` - Custom PJSIP configuration
- `my_manager.conf` - Custom AMI users
- `my_queues.conf` - Custom queue configuration
- `my_ari.conf` - Custom ARI configuration

Example:
```bash
docker run -d \
  --name asterisk \
  -v $(pwd)/config:/etc/asterisk \
  agentvoiceresponse/avr-asterisk:latest
```

## Example Configuration Files

### pjsip.conf
```ini
[transport-udp]
type=transport
protocol=udp
bind=0.0.0.0:5060

[6001]
type=endpoint
context=from-internal
disallow=all
allow=ulaw
allow=alaw
auth=6001
aors=6001

[6001]
type=auth
auth_type=userpass
password=your_password
username=6001

[6001]
type=aors
max_contacts=1
```

### extensions.conf
```ini
[from-internal]
exten => 6001,1,Answer()
exten => 6001,n,Echo()
exten => 6001,n,Hangup()
```

## Building from Source

If you want to build the image locally:

```bash
git clone https://github.com/agentvoiceresponse/avr-asterisk.git
cd avr-asterisk
docker build -t agentvoiceresponse/avr-asterisk:latest .
```

## Integration with AVR-AMI

This Asterisk image is designed to work seamlessly with [AVR-AMI](https://github.com/agentvoiceresponse/avr-ami), a Node.js service that provides REST API endpoints for call control operations.

### Quick Integration Setup

1. **Start Asterisk:**
   ```bash
   docker run -d --name asterisk \
     -p 5038:5038 \
     -p 8088:8088 \
     -p 10000-20000:10000-20000/udp \
     agentvoiceresponse/avr-asterisk:latest
   ```

2. **Start AVR-AMI** (configured to connect to Asterisk):
   ```bash
   docker run -d --name avr-ami \
     -p 6006:6006 \
     -e AMI_HOST=asterisk \
     -e AMI_PORT=5038 \
     -e AMI_USERNAME=avr \
     -e AMI_PASSWORD=avr \
     agentvoiceresponse/avr-ami:latest
   ```

3. **Configure AMI User** in Asterisk (`my_manager.conf`):
   ```ini
   [avr]
   secret = avr
   deny = 0.0.0.0/0.0.0.0
   permit = 0.0.0.0/0.0.0.0
   read = all
   write = all
   ```

## Security Considerations

- **TLS Certificates**: The image generates self-signed certificates by default. For production, mount your own certificates to `/etc/asterisk/keys/`
- **AMI Security**: Configure strong passwords in `manager.conf` and restrict access by IP if possible
- **Firewall**: Only expose necessary ports (5038 for AMI, 8088/8089 for API, 10000-20000 for RTP)
- **Network Isolation**: Use Docker networks to isolate Asterisk from other services

## Monitoring

### Prometheus Metrics

Prometheus metrics are enabled by default. Access metrics at:
```
http://localhost:8088/metrics
```

### Health Check

You can check Asterisk status using the CLI:
```bash
docker exec -it asterisk asterisk -rx "core show version"
```

## Performance and Scalability

This Asterisk container is optimized to support **hundreds of concurrent calls** with proper resource allocation and configuration.

### Capacity Estimates

| Concurrent Calls | CPU | RAM | Notes |
|-----------------|-----|-----|-------|
| 100-300 | 2 cores | 2 GB | Light to medium load |
| 300-500 | 4 cores | 4 GB | Medium to high load |
| 500-1000+ | 8+ cores | 8+ GB | High load, production |

### System Optimizations

The Dockerfile includes the following optimizations:

- **System Limits**: Increased file descriptors (65536) and processes (32768)
- **Minimal Modules**: Only essential modules enabled for reduced footprint
- **PJSIP**: Modern SIP stack optimized for performance
- **RTP Ports**: Full range 10000-20000 for media streaming

### Docker Compose Configuration

For high call volume, configure resources in `docker-compose.yml`:

```yaml
services:
  asterisk:
    image: agentvoiceresponse/avr-asterisk:latest
    sysctls:
      - net.core.somaxconn=4096
      - net.ipv4.tcp_max_syn_backlog=4096
      - net.core.netdev_max_backlog=5000
      - net.ipv4.ip_local_port_range=10000 65535
    deploy:
      resources:
        limits:
          cpus: '4.0'      # Adjust based on call volume
          memory: 4G       # Adjust based on call volume
        reservations:
          cpus: '2.0'
          memory: 2G
```

### Configuration Optimizations

#### 1. asterisk.conf

Mount optimized `asterisk.conf` for high call volume:

```ini
[options]
maxload = 0.9                    ; Maximum 90% CPU
maxfiles = 65536                 ; Maximum open files
minmemfree = 64                  ; Minimum 64MB free memory
maxcalls = 1000                  ; Maximum concurrent calls
maxcallers = 1000                ; Maximum callers
```

See `examples/asterisk.conf.optimized` for a complete example.

#### 2. my_pjsip.conf

Mount optimized `my_pjsip.conf` for PJSIP performance:

```ini
[global]
user_agent = AVR-Asterisk
endpoint_identifier_order = ip,username,anonymous
max_forwards = 70
contact_expiration_check_interval = 30
```

See `examples/my_pjsip.conf.optimized` for a complete example.

### Monitoring Performance

Monitor these metrics for optimal performance:

1. **CPU Usage**: Should stay below 80%
2. **Memory Usage**: Should stay below 80%
3. **Active Calls**: `asterisk -rx "core show channels"`
4. **RTP Ports**: Monitor usage of ports 10000-20000
5. **Network Latency**: Should be < 50ms

### Horizontal Scaling

For more than 1000 concurrent calls:

1. **Multiple Containers**: Deploy multiple Asterisk containers
2. **SIP Load Balancer**: Use Kamailio or OpenSIPS for SIP load balancing
3. **Tenant Distribution**: Distribute tenants across containers

### Example: High Volume Setup

```yaml
services:
  asterisk:
    image: agentvoiceresponse/avr-asterisk:latest
    volumes:
      - ./config/asterisk.conf:/etc/asterisk/asterisk.conf:ro
      - ./config/my_pjsip.conf:/etc/asterisk/my_pjsip.conf:ro
    sysctls:
      - net.core.somaxconn=4096
      - net.ipv4.tcp_max_syn_backlog=4096
    deploy:
      resources:
        limits:
          cpus: '8.0'
          memory: 8G
```

## Troubleshooting

### Check Asterisk Logs
```bash
docker logs asterisk
```

### Access Asterisk CLI
```bash
docker exec -it asterisk rasterisk
```

### Verify AMI Connection
```bash
telnet localhost 5038
```

## License

MIT License - See [LICENSE.md](LICENSE.md) for details.

## Contributing

Contributions are welcome! Please open an issue or submit a pull request on [GitHub](https://github.com/agentvoiceresponse/avr-asterisk).

## Support

For issues and questions:
- GitHub Issues: https://github.com/agentvoiceresponse/avr-asterisk/issues
- Documentation: https://github.com/agentvoiceresponse/avr-asterisk