Templates - Docker - Dashkiosk
==============================

[Back](../README.md)

Commands
--------

### One-line command

This is the command to create and start a Dashkiosk container:

```bash
docker run -d --name dashkiosk --restart=always -p 8084:8080 -v /var/run/docker.sock:/var/run/docker.sock vincentbernat/dashkiosk:latest
```

### Multi-line command

Or, in a more readable format:

```bash
docker run -d \
    --name dashkiosk \
    --restart=always \
    -p 8084:8080 \
    -v /var/run/docker.sock:/var/run/docker.sock \
    vincentbernat/dashkiosk:latest
```

Docker Compose
--------------

Create a new directory, `dashkiosk`, and a new file within named `docker-compose.yaml`:

```yaml
version: '3.1'
services:
  web:
    container_name: dashkiosk
    image: vincentbernat/dashkiosk:latest
    restart: always
    networks:
      - dashkiosk
    ports:
      - '8084:8080'
networks:
  dashkiosk:
    name: dashkiosk
    driver: bridge
```
