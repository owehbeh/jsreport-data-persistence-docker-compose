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
    volumes:
      - jsreport-data:/app/data
    labels:
      - traefik.enable=true
      - traefik.http.routers.jsreport.rule=Host(`reports.example.com`) && PathPrefix(`/`)
      - traefik.http.routers.jsreport.entrypoints=https
      - traefik.http.routers.jsreport.tls=true
      - traefik.http.services.jsreport.loadbalancer.server.port=5488
    environment:
      # --- jsreport authentication ---
      # admin login for Studio/API
      - extensions_authentication_admin_username=${JSREPORT_ADMIN_USER:?}
      - extensions_authentication_admin_password=${JSREPORT_ADMIN_PASS:?}
      # required cookie session secret (use a long random string)
      - extensions_authentication_cookieSession_secret=${JSREPORT_SESSION_SECRET:?}
      # good practice if you’re serving only over HTTPS
      - extensions_authentication_cookieSession_cookie_secure=true
      # (optional) set a custom login cookie name:
      # - extensions_authentication_cookieSession_name=jsreport.sid
volumes:
  jsreport-data:

