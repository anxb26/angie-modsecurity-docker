# angie-modsecurity-docker

Production-ready Angie web server with ModSecurity WAF, Fail2Ban, OWASP CRS 4.x, GeoIP enrichment, and comprehensive security hardening. Docker Compose template for secure web deployments.

[![CI](https://github.com/hvaclab/angie-modsecurity-docker/actions/workflows/ci.yml/badge.svg)](https://github.com/hvaclab/angie-modsecurity-docker/actions/workflows/ci.yml)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![Docker](https://img.shields.io/badge/Docker-Compose-blue.svg)](https://docs.docker.com/compose/)
[![Angie](https://img.shields.io/badge/Angie-1.10.3-green.svg)](https://angie.software/)
[![OWASP CRS](https://img.shields.io/badge/OWASP%20CRS-4.18.0-orange.svg)](https://coreruleset.org/)

## Contributing

See [CONTRIBUTING.md](CONTRIBUTING.md) for guidelines. PRs should target the `dev` branch and pass CI checks before merge to `main`.

## Features

### Security
- **Multi-layered Protection**: ModSecurity WAF, Fail2Ban, Rate Limiting
- **OWASP Core Rule Set**: 825+ security rules protecting against common attacks
- **Automatic SSL/TLS**: Built-in ACME client for Let's Encrypt certificates
- **HTTP/2 & HTTP/3 (QUIC)**: Modern protocols with enhanced security and performance
- **Security Headers**: HSTS, CSP, X-Frame-Options, X-Content-Type-Options, etc.
- **Rate Limiting**: DDoS protection at the web server level
- **OAuth2/OIDC Support**: Ready-to-use Keycloak authentication

### Monitoring & Analytics
- **Rich Log Enrichment**: 70+ fields including GeoIP, User-Agent analysis, security scoring
- **Vector Log Pipeline**: Advanced log processing and transformation
- **GeoIP Support**: Country, city, coordinates, timezone information
- **Security Scoring**: Automatic threat level classification (0-15)
- **Performance Metrics**: Request timing, upstream status, caching efficiency

### Architecture

```
┌─────────────────────────────────────────────────────────────┐
│ LAYER 1: fail2ban (Firewall)                               │
│ • Blocks mass attacks at iptables level                     │
│ • 4 active jails (bad-request, modsecurity, scan, ddos)     │
│ • Load: ~0.1% CPU                                           │
└─────────────┬───────────────────────────────────────────────┘
              ↓
┌─────────────────────────────────────────────────────────────┐
│ LAYER 2: ModSecurity (WAF)                                  │
│ • OWASP Core Rule Set 4.18.0 (825+ rules)                  │
│ • Protection: SQL injection, XSS, Path traversal, RCE       │
│ • Load: ~5-15% CPU per request                             │
└─────────────┬───────────────────────────────────────────────┘
              ↓
┌─────────────────────────────────────────────────────────────┐
│ LAYER 3: Angie (Web Server)                                │
│ • Log enrichment (70+ fields)                               │
│ • GeoIP, User-Agent, Security, Performance analysis         │
│ • SSL/TLS (TLSv1.2, TLSv1.3)                               │
└─────────────┬───────────────────────────────────────────────┘
              ↓
┌─────────────────────────────────────────────────────────────┐
│ LAYER 4: Vector (Log Pipeline)                              │
│ • Additional enrichment and processing                       │
│ • Security scoring (0-15)                                   │
│ • Threat level classification                               │
└─────────────────────────────────────────────────────────────┘
```

## Project Structure

```
angie-modsecurity-docker/
├── compose.yml              # Docker Compose configuration
├── .env.example             # Environment variables template
│
├── angie/                   # Angie configuration
│   ├── angie.conf          # Main configuration file
│   ├── Dockerfile          # Custom Angie image with ModSecurity
│   ├── includes/           # Reusable configuration includes
│   │   ├── auth/           # Authentication (Keycloak/OAuth2)
│   │   ├── security/       # Security headers, SSL, rate limiting
│   │   ├── logs/           # Log formats and enrichment
│   │   ├── performance/    # Compression, caching
│   │   └── monitoring/     # VTS metrics
│   ├── conf.d/             # Virtual host configurations
│   │   ├── default.conf    # Default server (returns 444)
│   │   ├── example.com.conf           # Static site example
│   │   └── proxy-example.com.conf     # Reverse proxy example
│   └── stream.d/           # TCP/UDP stream configurations
│
├── certs/                  # SSL certificates (empty in template)
│   └── .gitkeep
│
├── logs/                   # Application logs (empty in template)
│   └── .gitkeep
│
├── modsec/                 # ModSecurity configuration
│   ├── rules.conf          # Main WAF rules
│   └── exclusions.conf     # Custom rule exclusions
│
├── fail2ban/               # Fail2Ban configuration
│   ├── filter.d/           # Log filters
│   │   ├── angie-bad-request.conf
│   │   ├── angie-modsecurity.conf
│   │   ├── angie-scan.conf
│   │   └── angie-ddos.conf
│   ├── jail.d/
│   │   └── angie.conf      # Jail configurations
│   └── db/                 # Database (empty in template)
│
├── geoip/                  # GeoIP databases (download separately)
│   └── .gitkeep
│
├── vector/                 # Vector log pipeline
│   └── vector.toml         # Vector configuration
│
├── web/                    # Static website files
│   ├── index.html
│   ├── robots.txt
│   ├── images/
│   ├── styles/
│   └── scripts/
│
├── scripts/                # Utility scripts
│   ├── rotate-logs.sh
│   ├── check-ssl-expiry.sh
│   └── test-rate-limiting.sh
│
└── errors/                 # Custom error pages
    └── 50x.html
```

## Quick Start

### Prerequisites

- Docker 20.10+
- Docker Compose 2.0+
- Ports 80 and 443 available

### Step 1: Clone and Configure

```bash
# Clone the repository
git clone https://github.com/hvaclab/angie-modsecurity-docker.git
cd angie-modsecurity-docker

# Create environment file from template
cp .env.example .env

# Edit configuration
nano .env
```

### Step 2: Configure Your Domain

Edit `.env` file:

```env
# Your email for Let's Encrypt notifications
LETSENCRYPT_EMAIL=admin@yourdomain.com

# Use staging for testing (1 = testing, 0 = production)
LETSENCRYPT_STAGING=1

# RSA key size
RSA_KEY_SIZE=4096

# Timezone
TZ=America/New_York

# Optional: Keycloak OIDC Configuration (if using authentication)
KEYCLOAK_URL=https://login.yourdomain.com
KEYCLOAK_REALM=master
KEYCLOAK_CLIENT_ID=your-client-id
KEYCLOAK_CLIENT_SECRET=your-client-secret-here
KEYCLOAK_REDIRECT_URI=https://yourdomain.com/oauth2/callback
KEYCLOAK_COOKIE_SECRET=generate-random-32-char-string!!
KEYCLOAK_COOKIE_NAME=_oauth2_proxy
```

### Step 3: Configure Virtual Host

Copy and edit the example configuration:

```bash
cd angie/conf.d

# For static site
cp example.com.conf yourdomain.com.conf

# For reverse proxy
cp proxy-example.com.conf yourdomain.com.conf

# Edit the configuration
nano yourdomain.com.conf
```

Replace `example.com` with your actual domain name in the configuration file.

### Step 4: Generate Self-Signed Certificates (Initial Setup)

Before starting, generate temporary self-signed certificates:

```bash
# Generate DH parameters (takes a few minutes)
openssl dhparam -out certs/dhparam.pem 2048

# Generate self-signed certificate for default server
openssl req -x509 -nodes -days 365 -newkey rsa:2048 \
  -keyout certs/default-selfsigned.key \
  -out certs/default-selfsigned.crt \
  -subj "/C=US/ST=State/L=City/O=Organization/CN=default"
```

### Step 5: Download GeoIP Databases (Optional but Recommended)

```bash
cd geoip

# Download free GeoLite2 databases
curl -L "https://github.com/P3TERX/GeoLite.mmdb/raw/download/GeoLite2-Country.mmdb" -o GeoLite2-Country.mmdb
curl -L "https://github.com/P3TERX/GeoLite.mmdb/raw/download/GeoLite2-City.mmdb" -o GeoLite2-City.mmdb

cd ..
```

### Step 6: Start Services

```bash
# Build and start all services
docker compose up -d

# Check status
docker compose ps

# View logs
docker compose logs -f
```

### Step 7: Verify Installation

```bash
# Check Angie configuration
docker exec angie angie -t

# Check SSL certificate (after ACME obtains it)
docker exec angie angie -T | grep ssl_certificate

# Test rate limiting
./scripts/test-rate-limiting.sh

# Check Fail2Ban status
docker exec fail2ban fail2ban-client status
```

## Configuration Guide

### Virtual Host Configuration

#### Static Website

Create `angie/conf.d/yourdomain.com.conf`:

```nginx
server {
    listen 443 ssl;
    listen 443 quic reuseport;
    http2 on;

    server_name yourdomain.com;

    # Automatic Let's Encrypt
    acme letsencrypt;
    ssl_certificate     $acme_cert_letsencrypt;
    ssl_certificate_key $acme_cert_key_letsencrypt;

    # Security headers
    include /etc/angie/includes/security/security-headers.conf;

    # ModSecurity WAF
    modsecurity on;
    modsecurity_rules_file /etc/angie/modsecurity/rules.conf;

    root /var/www/html;
    index index.html;

    location / {
        limit_req zone=general burst=20 nodelay;
        try_files $uri $uri/ =404;
    }
}

server {
    listen 80;
    server_name yourdomain.com;
    return 301 https://$host$request_uri;
}
```

#### Reverse Proxy

For proxying to backend applications:

```nginx
upstream backend {
    server backend-service:8080;
    keepalive 32;
}

server {
    listen 443 ssl;
    http2 on;

    server_name api.yourdomain.com;

    acme letsencrypt;
    ssl_certificate     $acme_cert_letsencrypt;
    ssl_certificate_key $acme_cert_key_letsencrypt;

    location / {
        proxy_pass http://backend;
        proxy_http_version 1.1;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

### ModSecurity Configuration

Edit `modsec/exclusions.conf` to disable specific rules:

```nginx
# Disable rule for specific location
SecRuleRemoveById 920100 920200

# Disable rule for specific URL
SecRule REQUEST_URI "@streq /admin/upload" \
    "id:1001,phase:1,pass,nolog,ctl:ruleRemoveById=920100"

# Disable ModSecurity for specific path
SecRule REQUEST_URI "@beginsWith /api/webhook" \
    "id:1002,phase:1,pass,nolog,ctl:ruleEngine=Off"
```

### Fail2Ban Configuration

Jails are configured in `fail2ban/jail.d/angie.conf`:

- **angie-bad-request**: Blocks HTTP 400/444 errors (5 times in 10 min = 1 hour ban)
- **angie-modsecurity**: Blocks ModSecurity violations (3 times in 10 min = 1 day ban)
- **angie-scan**: Blocks scanning attempts (5 times in 5 min = 1 week ban)
- **angie-ddos**: Blocks DDoS attacks (100 requests in 1 min = 10 min ban)

Customize ban times in the jail configuration.

### Authentication (Optional)

To enable Keycloak/OAuth2 authentication:

1. Configure `.env` with Keycloak settings
2. Uncomment auth includes in your virtual host:

```nginx
# Keycloak Authentication
include /etc/angie/includes/auth/keycloak-auth.conf;

# Protected paths
include /etc/angie/includes/auth/keycloak-protected-paths.conf;
```

3. Edit `angie/includes/auth/keycloak-protected-paths.conf` to specify protected paths

## Management

### Angie

```bash
# Test configuration
docker exec angie angie -t

# Reload configuration
docker exec angie angie -s reload

# View logs
docker logs angie -f

# Check current connections
docker exec angie angie -T | grep worker_connections
```

### Fail2Ban

```bash
# Status of all jails
docker exec fail2ban fail2ban-client status

# Status of specific jail
docker exec fail2ban fail2ban-client status angie-bad-request

# View banned IPs
docker exec fail2ban fail2ban-client banned

# Unban IP
docker exec fail2ban fail2ban-client set angie-bad-request unbanip 192.0.2.10

# Ban IP manually
docker exec fail2ban fail2ban-client set angie-bad-request banip 192.0.2.10
```

### SSL Certificates

```bash
# Check ACME certificate status
docker exec angie ls -la /var/lib/angie/acme/

# Force certificate renewal (if needed)
docker exec angie angie -s reload

# Check certificate expiration
./scripts/check-ssl-expiry.sh
```

### Logs

```bash
# View access logs (enriched JSON)
tail -f logs/access.log | jq

# View access logs with Vector enrichment
tail -f logs/access_enriched.log | jq

# View error logs (includes ModSecurity)
tail -f logs/error.log

# Rotate logs manually
./scripts/rotate-logs.sh
```

## Monitoring

### Log Fields

The enriched logs include 70+ fields:

**Basic Request Info**
- `timestamp`, `method`, `uri`, `status`, `bytes_sent`
- `request_time`, `upstream_response_time`

**Client Information**
- `remote_addr`, `geoip_country_code`, `geoip_city_name`
- `user_agent`, `browser_name`, `os_name`, `device_type`

**Security**
- `security_score` (0-15), `threat_level` (low/medium/high/critical)
- `is_bot`, `is_crawler`, `is_mobile`
- `modsecurity_score`, `blocked_by_modsec`

**Performance**
- `cache_status`, `compression_ratio`
- `ssl_protocol`, `http_version`

### Integration with Log Aggregators

The JSON logs can be easily integrated with:

- **Grafana Loki**: Ship logs with Promtail
- **Elasticsearch/OpenSearch**: Use Filebeat or Logstash
- **ClickHouse**: Use Vector or Filebeat
- **Datadog/New Relic**: Use their agents

Example Vector sink for Loki (add to `vector/vector.toml`):

```toml
[sinks.loki]
type = "loki"
inputs = ["enriched_logs"]
endpoint = "http://loki:3100"
encoding.codec = "json"
labels.job = "angie"
```

## Maintenance

### Update GeoIP Databases

```bash
cd geoip
curl -L "https://github.com/P3TERX/GeoLite.mmdb/raw/download/GeoLite2-Country.mmdb" -o GeoLite2-Country.mmdb
curl -L "https://github.com/P3TERX/GeoLite.mmdb/raw/download/GeoLite2-City.mmdb" -o GeoLite2-City.mmdb
docker exec angie angie -s reload
```

### Update ModSecurity Rules

```bash
cd angie
# Edit Dockerfile to update OWASP_CRS_VERSION
docker compose build angie
docker compose up -d angie
```

### Log Rotation

Add to crontab:

```bash
crontab -e

# Add:
0 2 * * * /path/to/angie-modsecurity-docker/scripts/rotate-logs.sh
```

### Backup Important Data

```bash
# Backup ACME certificates
docker run --rm -v angie-modsecurity-docker_acme_data:/data -v $(pwd):/backup \
  alpine tar czf /backup/acme-backup.tar.gz -C /data .

# Backup configuration
tar czf config-backup.tar.gz angie/ modsec/ fail2ban/ .env

# Backup logs (optional)
tar czf logs-backup.tar.gz logs/
```

## Troubleshooting

### Angie Won't Start

```bash
# Check logs
docker logs angie

# Test configuration
docker exec angie angie -t

# Check file permissions
ls -la angie/ certs/ logs/
```

### SSL Certificate Issues

```bash
# Check ACME logs in error.log
tail -f logs/error.log | grep -i acme

# Verify domain is accessible on port 80 (required for ACME)
curl -I http://yourdomain.com/.well-known/acme-challenge/test

# Use staging first to avoid rate limits
# Set LETSENCRYPT_STAGING=1 in .env
```

### Fail2Ban Not Banning

```bash
# Check jail status
docker exec fail2ban fail2ban-client status angie-bad-request

# Test filter
docker exec fail2ban fail2ban-regex /var/log/angie/access.log /data/filter.d/angie-bad-request.conf

# View Fail2Ban logs
docker logs fail2ban -f
```

### High CPU Usage

```bash
# Check top processes in Angie
docker exec angie top

# Check worker connections
docker exec angie angie -T | grep worker

# Consider:
# 1. Increase worker_processes in angie.conf
# 2. Adjust rate limiting
# 3. Enable more aggressive caching
# 4. Disable ModSecurity for specific paths
```

### ModSecurity False Positives

```bash
# Find blocked requests in logs
tail -f logs/error.log | grep ModSecurity

# Add exclusions in modsec/exclusions.conf
SecRuleRemoveById 920100

# Or disable for specific paths
modsecurity off;
```

## Security Best Practices

1. **Always use HTTPS**: Redirect HTTP to HTTPS
2. **Keep software updated**: Regularly update Docker images
3. **Monitor logs**: Set up alerts for suspicious activity
4. **Use strong passwords**: For OAuth2 cookie secrets, etc.
5. **Limit exposed ports**: Only expose 80 and 443
6. **Regular backups**: Backup ACME certificates and configurations
7. **Test before production**: Use LETSENCRYPT_STAGING=1 first
8. **Rate limiting**: Adjust based on your traffic patterns
9. **GeoIP blocking**: Block countries if needed (use GeoIP directives)
10. **Regular audits**: Review Fail2Ban bans and ModSecurity logs

## Performance Tuning

### Worker Processes

Edit `angie/angie.conf`:

```nginx
# Set to number of CPU cores
worker_processes auto;

# Or specify number
worker_processes 4;
```

### Caching

Enable caching for static content:

```nginx
# In http block
proxy_cache_path /var/cache/angie levels=1:2 keys_zone=my_cache:10m max_size=1g inactive=60m;

# In location block
location / {
    proxy_cache my_cache;
    proxy_cache_valid 200 60m;
    proxy_cache_use_stale error timeout http_500 http_502 http_503 http_504;
}
```

### Connection Limits

```nginx
# In http block
limit_conn_zone $binary_remote_addr zone=addr:10m;

# In server block
limit_conn addr 10;  # Max 10 concurrent connections per IP
```

## Technology Stack

- **Angie** 1.7.0+ - High-performance web server (Nginx fork)
- **ModSecurity** 3.0.14+ - Web Application Firewall
- **OWASP CRS** 4.18.0+ - Core Rule Set (825+ rules)
- **Fail2Ban** latest - Intrusion prevention
- **Vector** 0.41.1+ - Log pipeline and enrichment
- **OAuth2 Proxy** 7.6.0+ - Authentication proxy
- **Docker** 20.10+ - Containerization
- **Docker Compose** 2.0+ - Multi-container orchestration

## Contributing

This is a template project. Feel free to:

1. Fork and customize for your needs
2. Report issues or suggestions
3. Share improvements and best practices

## License

MIT License - Free to use and modify

## Support

For Angie documentation: https://angie.software/en/
For ModSecurity: https://github.com/SpiderLabs/ModSecurity
For OWASP CRS: https://coreruleset.org/

## Credits

Template created with best practices for:
- DevOps engineers
- Security-conscious deployments
- Production-ready web applications
- High-traffic websites

Built with Claude Code AI assistance.

---

**Status**: Production-Ready
**Last Updated**: 2025-12-06
**Version**: 1.0.0
