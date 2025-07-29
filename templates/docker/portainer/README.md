# Templates - Docker - Portainer

[Back](../README.md)

## Table of contents

- [Pre-configuration](#pre-configuration)
- [Commands](#commands)
    - [One-line command](#one-line-command)
    - [Multi-line command](#multi-line-command)
- [Docker Compose](#docker-compose)

## Pre-configuration

Create a Docker volume to store Portainer data:

```
docker volume create portainer_data
```

## Commands

### One-line command

This is the command to create and start a Portainer container:

```bash
docker run -d --name portainer --restart=always -p 8000:8000 -p 9443:9443 -v /var/run/docker.sock:/var/run/docker.sock -v portainer_data:/data portainer/portainer-ce:latest
```

### Multi-line command

Or, in a more readable format:

```bash
docker run -d \
    --name portainer \
    --restart=always \
    -p 8000:8000 \
    -p 9443:9443 \
    -v /var/run/docker.sock:/var/run/docker.sock \
    -v portainer_data:/data \
    portainer/portainer-ce:latest
```

## Docker Compose

Create a new directory, `portainer`, and a new file within named `docker-compose.yaml`:

```yaml
version: '3.1'
services:
  web:
    container_name: portainer
    image: portainer/portainer-ce:latest
    restart: always
    networks:
      - portainer
    ports:
      - '8000:8000'
      - '9443:9443'
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
      - data:/data
networks:
  portainer:
    name: portainer
    driver: bridge
volumes:
  data:
```
