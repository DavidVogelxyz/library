# Templates - Docker - Vaultwarden

[Back](../README.md)

## Introduction

🚨 **The user must create `vaultwarden-docker.crt` and `vaultwarden-docker.key` and place them at the specified paths!** 🚨

## Commands

### One-line command

This is the command to create and start a Vaultwarden container:

```bash
docker run -d --name vaultwarden --restart=always -p 8081:80 -e ROCKET_TLS='{certs="/data/vaultwarden-docker.crt",key="/data/vaultwarden-docker.key"}' -v /var/run/docker.sock:/var/run/docker.sock -v vaultwarden_data:/data vaultwarden/server:latest
```

### Multi-line command

Or, in a more readable format:

```bash
docker run -d \
    --name vaultwarden \
    --restart=always \
    -p 8081:80 \
    -e ROCKET_TLS='{certs="/data/vaultwarden-docker.crt",key="/data/vaultwarden-docker.key"}' \
    -v /var/run/docker.sock:/var/run/docker.sock \
    -v vaultwarden_data:/data \
    vaultwarden/server:latest
```

## Docker Compose

Create a new directory, `vaultwarden`, and a new file within named `docker-compose.yaml`:

```yaml
version: '3.1'
services:
  web:
    name: vaultwarden
    image: vaultwarden/server:latest
    restart: always
    environment:
      - ROCKET_TLS={certs="/data/vaultwarden-docker.crt",key="/data/vaultwarden-docker.key"}
    networks:
      - vaultwarden
    ports:
      - '8081:80'
    volumes:
      - data:/data
networks:
  vaultwarden:
    name: vaultwarden
    driver: bridge
volumes:
  data:
```
