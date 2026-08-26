FROM caddy:2.10-alpine

LABEL maintainer="Michael Aguiar <mike@aliasproject.com>"

# Runtime UID. Keep this DIFFERENT from the UID that owns bind-mounted
# content on the host, mirroring docker-php's appuser convention so
# downstream images (e.g. FROM this image + COPY a static site) behave
# the same way as downstream php-frankenphp images.
ARG APP_UID=1000

# Copy custom HTML files to the web root
COPY html/ /usr/share/caddy/

# Copy custom Caddyfile
COPY Caddyfile /etc/caddy/Caddyfile

# Non-root runtime user in group 0. No chgrp/chmod/setcap needed: the base
# image already grants cap_net_bind_service on /usr/bin/caddy and ships
# /data/caddy + /config/caddy world-writable for certmagic/autosave state.
RUN adduser -u ${APP_UID} -G root -s /sbin/nologin -D appuser

EXPOSE 80 443 443/udp

HEALTHCHECK --interval=30s --timeout=5s --start-period=5s --retries=3 \
    CMD wget -qO /dev/null http://127.0.0.1:2019/config/ || exit 1

USER ${APP_UID}:0
