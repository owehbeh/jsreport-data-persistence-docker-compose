# jsreport on Coolify (Docker Compose)

Run a persistent **jsreport** instance on a Coolify-managed server, exposed at a custom subdomain over HTTPS. Data (templates, assets, settings) survives restarts via a named Docker volume.

## What this deploys

- **jsreport** (official Docker image), listening internally on port **5488**.  
- A **named volume** mounted at **`/app/data`** so templates/settings persist across restarts.  
- **Traefik** routing via labels, so your app is reachable at `https://<your-subdomain>.<your-domain>` with TLS handled by Coolify.

## Prerequisites

1. Coolify is running on your server and uses **Traefik** (default).  
2. Your DNS wildcard (e.g., `*.example.com`) points to the Coolify server IP, and you set the Wildcard Domain in Coolify’s server settings.  
3. You have (or will choose) a subdomain like `reports.example.com` for jsreport.

## Docker Compose file

> Replace `reports.example.com` with your subdomain.

```yaml
services:
  jsreport:
    image: jsreport/jsreport:latest
    # Do NOT publish ports when using the proxy
    # ports:
    #   - "5488:5488"
    volumes:
      - jsreport-data:/app/data
    labels:
      - traefik.enable=true
      - traefik.http.routers.jsreport.rule=Host(`reports.example.com`) && PathPrefix(`/`)
      - traefik.http.routers.jsreport.entrypoints=https
      - traefik.http.routers.jsreport.tls=true
      - traefik.http.services.jsreport.loadbalancer.server.port=5488

volumes:
  jsreport-data:
