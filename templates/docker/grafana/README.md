Templates - Docker - Grafana
============================

[Back](../README.md)

Commands
--------

### One-line command

This is the command to create and start a Grafana container:

```bash
docker run -d --name grafana --restart=always -p 8083:3000 -v /var/run/docker.sock:/var/run/docker.sock grafana/grafana:latest
```

### Multi-line command

Or, in a more readable format:

```bash
docker run -d \
    --name grafana \
    --restart=always \
    -p 8083:3000 \
    -v /var/run/docker.sock:/var/run/docker.sock \
    grafana/grafana:latest
```

Docker Compose
--------------

Create a new directory, `grafana`, and a new file within named `docker-compose.yaml`:

```yaml
version: '3.1'
services:
  web:
    container_name: grafana
    image: grafana/grafana:latest
    restart: always
    networks:
      - grafana
    ports:
      - '8083:3000'
networks:
  grafana:
    name: grafana
    driver: bridge
```
