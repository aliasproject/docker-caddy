# Docker Caddy Proxy

A custom Caddy Docker image with a default HTML page for proxy deployments.

## 🚀 Features

- Custom HTML landing page instead of Caddy's default
- Ready-to-use proxy configuration
- Lightweight Alpine-based image
- Simple deployment with Docker

## 📦 Quick Start

### Build and Run

```bash
# Build the custom image
docker build -t custom-caddy ./caddy/2.10/

# Run the container
docker run -d -p 80:80 -p 443:443 --name caddy-proxy custom-caddy
```

### Verify Installation

Visit `http://localhost` to see your custom landing page.

## 🔧 Configuration

### Adding Proxy Routes

Edit `caddy/2.10/Caddyfile` to add your proxy configurations:

```caddyfile
:{$PORT:80} {
    root * /usr/share/caddy
    file_server
}

# Add your proxy routes
example.com {
    reverse_proxy backend:8080
}

api.example.com {
    reverse_proxy api-server:3000
}
```

The default site block binds `$PORT` (falling back to `80`) rather than a
hardcoded port, so the same image works both behind a platform that injects
`$PORT` and in a plain `docker run -p 80:80`.

## 🌐 Deploying a Static Site

This image is built the same way `docker-php`'s frankenphp base is: a
hardened, parametrized base that a per-site repo builds `FROM`, rather than
something you configure in place. A static site's own Dockerfile mirrors how
a PHP app (e.g. Alios) builds `FROM ghcr.io/aliasproject/php:8.4-frankenphp`:

```dockerfile
FROM ghcr.io/aliasproject/caddy:2.10-alpine

# Add the static site content
COPY . /usr/share/caddy/
```

The base image already provides:

- **`$PORT`-driven binding** — the Caddyfile listens on `:{$PORT:80}`, so it
  works unmodified behind PaaS platforms that inject `$PORT` (Coolify, etc.)
- **Non-root runtime** — runs as `appuser` (group `0`), not root
- **Compression** — `zstd`/`gzip` are enabled by default (the stock
  `caddy:2.10-alpine` build has no `br`/Brotli module compiled in)
- **A healthcheck** against Caddy's admin API

If the site needs its own routes (an API health-check endpoint, custom
headers, SPA fallback routing, etc.), also `COPY` a replacement `Caddyfile`
the way the base image's own `Caddyfile` is structured, keeping the
`:{$PORT:80}` address line.

### Customizing the Landing Page

Modify `caddy/2.10/html/index.html` to customize the default page, then rebuild:

```bash
docker build -t custom-caddy ./caddy/2.10/
docker restart caddy-proxy
```

## 📁 Project Structure

```
docker-caddy/
├── README.md
└── caddy/
    └── 2.10/
        ├── Dockerfile        # Custom Caddy image
        ├── Caddyfile         # Server configuration
        └── html/
            └── index.html    # Custom landing page
```

## 🛠️ Development

### Testing Configuration

```bash
# Validate Caddyfile syntax
docker run --rm -v $(pwd)/caddy/2.10/Caddyfile:/etc/caddy/Caddyfile \
  caddy:2.10-alpine caddy validate --config /etc/caddy/Caddyfile
```

### View Logs

```bash
docker logs -f caddy-proxy
```

### Stop Container

```bash
docker stop caddy-proxy && docker rm caddy-proxy
```

## 📄 License

This project is open source and available under the MIT License.

---

**Author:** Michael Aguiar <mike@aliasproject.com>
