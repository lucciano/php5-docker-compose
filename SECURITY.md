# WAF — OWASP ModSecurity CRS

## Update the CRS ruleset

The rules come from the base image. To get a newer version, update the image tag in `docker/nginx/Dockerfile`:

```dockerfile
FROM owasp/modsecurity-crs:nginx-alpine   # latest stable
# or pin to a specific version:
FROM owasp/modsecurity-crs:4-nginx-alpine
```

Then rebuild:

```bash
docker compose build --no-cache nginx_php
docker compose up -d
```

## Change enforcement mode

Edit `docker-compose.yml`:

```yaml
environment:
  - MODSEC_RULE_ENGINE=On            # block attacks
  - MODSEC_RULE_ENGINE=DetectionOnly # log only, no blocking
  - MODSEC_RULE_ENGINE=Off           # disable WAF
```

Restart after changing:

```bash
docker compose up -d
```

## Tune paranoia level

Higher paranoia = stricter rules but more false positives. Edit `docker/nginx/modsecurity-override.conf`:

```
SecAction "id:900000, phase:1, pass, t:none, setvar:tx.paranoia_level=1"
```

Levels: `1` (default) → `2` → `3` → `4` (strictest). Rebuild after changing.

## Whitelist a false positive

Add an exclusion in `docker/nginx/modsecurity-override.conf`:

```
# Disable a specific rule by ID
SecRuleRemoveById 942100

# Disable a rule for a specific path only
SecRule REQUEST_URI "@beginsWith /api/upload" "id:1001,phase:1,pass,nolog,ctl:ruleRemoveById=942100"
```

Rebuild after changing.

## Check WAF logs

```bash
# Live log tail
docker compose logs -f nginx_php

# Only blocked requests (HTTP 403)
docker compose logs nginx_php | grep " 403 "
```
