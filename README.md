# Docker Swarm with Traefik and Portainer

This guide will help you set up a Docker Swarm environment with Traefik as a reverse proxy and Portainer for easy management of your Docker environment.

## Prerequisites

- Docker installed on your machine
- Command line interface (CLI) access
- Public IP address for your server

## Step 1: Initialize Docker Swarm

Initialize Docker Swarm mode on your manager node using your public IP address:

```bash
docker swarm init --advertise-addr public-ip
```

## Step 2: Install Portainer

First, retrieve the Portainer stack YML manifest:

```bash
curl -L https://downloads.portainer.io/ce2-19/portainer-agent-stack.yml -o portainer-agent-stack.yml
```

Deploy the Portainer stack using the downloaded YML manifest:

```bash
docker stack deploy -c portainer-agent-stack.yml portainer
```

Verify that the Portainer Server and Agent containers have started:

```bash
docker ps
```

## Step 3: Access Portainer

Open a web browser and navigate to:

```
https://your-public-ip:9443
```

Complete the initial setup on the Portainer Server setup page.

## Step 4: Install Traefik

Prepare the environment for Traefik:

```bash
cd /etc/ && mkdir traefik && cd traefik && mkdir certs
touch /etc/traefik/certs/acme.json && chmod 600 /etc/traefik/certs/acme.json
```

Create and edit the Traefik configuration:

```bash
nano /etc/traefik/traefik.yml
```

Paste the following configuration into the file:

```yaml
global:
  checkNewVersion: false
  sendAnonymousUsage: false

accesslog:
  format: common
  filePath: /var/log/traefik/access.log

api:
  dashboard: true
  insecure: true  # Change to false in production

entryPoints:
  web:
    address: ":80"
    http:
      redirections:
        entryPoint:
          to: "websecure"
          scheme: "https"
  websecure:
    address: ":443"

certificatesResolvers:
  production:
    acme:
      email: admin@metexlabz.com
      storage: /etc/traefik/certs/acme.json
      caServer: "https://acme-v02.api.letsencrypt.org/directory"
      httpChallenge:
        entryPoint: web

providers:
  docker:
    exposedByDefault: false
  file:
    directory: /etc/traefik
    watch: true
```

Save and exit the editor.

## Step 5: Deploy Traefik in Docker

Navigate to the Portainer management portal:

```
https://your-public-ip:9443
```

Go to Stacks and add a new stack named `traefik` with the following YML content:

```yaml
version: '3'

volumes:
  traefik-ssl-certs:
    driver: local

services:
  traefik:
    image: docker.io/library/traefik:latest
    container_name: traefik
    ports:
      - 80:80
      - 443:443
      - 8080:8080  # Optional: For dashboard
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock:ro
      - /etc/traefik:/etc/traefik
      - traefik-ssl-certs:/ssl-certs
```

Deploy the stack and configure the Traefik service:

1. Navigate to Services inside Portainer.
2. Select `traefik_traefik`.
3. From the left side menu, navigate to Quick Actions -> Container Labels.
4. Add the following labels:

```plaintext
traefik.enable=true
traefik.http.routers.myrouter.rule=Host(`example.com`)
traefik.http.routers.myrouter.entrypoints=websecure
traefik.http.routers.myrouter.service=myservice
traefik.http.routers.myrouter.tls=true
traefik.http.routers.myrouter.tls.certresolver=myresolver
traefik.http.services.myservice.loadbalancer.server.port=8080
```

Apply the changes, and your Traefik dashboard should be accessible through:

```
https://example.com
```

Congratulations! You have successfully set up Docker Swarm with Traefik and Portainer.
```

This `README.md` is structured to provide a clear, step-by-step guide for setting up your environment, complete with code blocks and configuration examples. Adjust the actual IP addresses, domain names

, and emails according to your setup requirements.
