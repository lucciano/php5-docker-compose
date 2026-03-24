FROM owasp/modsecurity-crs:nginx-alpine

# Override the server block template — entrypoint processes this into /etc/nginx/conf.d/default.conf
COPY default.conf /etc/nginx/templates/conf.d/default.conf.template

# Override the ModSecurity tuning template — entrypoint processes this into /etc/nginx/modsecurity.d/modsecurity-override.conf
COPY modsecurity-override.conf /etc/nginx/templates/modsecurity.d/modsecurity-override.conf.template
