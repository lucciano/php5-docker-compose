# PHP 5 Demo

A PHP 5.6 demo app running behind nginx with OWASP ModSecurity WAF.

## Stack

| Service | Image | Port |
|---------|-------|------|
| PHP-FPM | `php:5.6-fpm` | internal (9000) |
| Nginx + WAF | `owasp/modsecurity-crs:nginx-alpine` | `5081` → `8080` |
| MySQL | `mysql:5.7` | `3306` |

## MySQL

| Setting | Value |
|---------|-------|
| Host (from PHP) | `mysql` |
| Port | `3306` |
| Root password | `root` |
| Database | `app` |
| User | `app` |
| Password | `app` |

Data is persisted in a named Docker volume (`mysql_data`). To reset it:

```bash
docker compose down -v
```

## WAF — OWASP ModSecurity CRS

Nginx is protected by **ModSecurity v3** with the **OWASP Core Rule Set (CRS)**, which blocks common web attacks including:

- SQL Injection (SQLi)
- Cross-Site Scripting (XSS)
- Local/Remote File Inclusion (LFI/RFI)
- Command Injection
- Protocol violations and anomalies

### Configuration

| File | Purpose |
|------|---------|
| `docker/nginx/Dockerfile` | Builds nginx from `owasp/modsecurity-crs:nginx-alpine` |
| `docker/nginx/default.conf` | Nginx server block (PHP-FPM proxy, port 8080) |
| `docker/nginx/modsecurity-override.conf` | Custom ModSecurity tuning (blocking mode, body limits, audit logging) |

The WAF runs in **blocking mode** (`SecRuleEngine On`). To switch to detection/log-only mode, change `MODSEC_RULE_ENGINE` in `docker-compose.yml`:

```yaml
environment:
  - MODSEC_RULE_ENGINE=DetectionOnly  # log only, no blocking
```

## Usage

### Start

```bash
docker compose up -d
```

### Build (after config changes)

```bash
docker compose build nginx_php
docker compose up -d
```

### Logs

```bash
# All services
docker compose logs -f

# WAF / nginx only
docker compose logs -f nginx_php
```

### Stop

```bash
docker compose down
```

## Networks

- `default` — internal service communication
- `hmlb-net` — external shared network (must exist before starting)

Create the external network if it doesn't exist:

```bash
docker network create hmlb-net
```
