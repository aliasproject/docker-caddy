FROM caddy:2.10-alpine

LABEL maintainer="Michael Aguiar <mike@aliasproject.com>"

# Copy custom HTML files to the web root
COPY html/ /usr/share/caddy/

# Copy custom Caddyfile
COPY Caddyfile /etc/caddy/Caddyfile
