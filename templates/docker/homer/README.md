# Templates - Docker - Homer

[Back](../README.md)

## Commands

### One-line command

This is the command to create and start a Homer container:

```bash
docker run -d --name homer --restart=always -p 8080:8080 -v /var/run/docker.sock:/var/run/docker.sock -v homer_data:/data b4bz/homer:latest
```

### Multi-line command

Or, in a more readable format:

```bash
docker run -d \
    --name homer \
    --restart=always \
    -p 8080:8080 \
    -v /var/run/docker.sock:/var/run/docker.sock \
    -v homer_data:/data \
    b4bz/homer:latest
```

## Docker Compose

Create a new directory, `homer`, and a new file within named `docker-compose.yaml`:

```yaml
version: '3.1'
services:
  web:
    container_name: homer
    image: b4bz/homer:latest
    restart: always
    networks:
      - homer
    ports:
      - '8080:8080'
    volumes:
      - data:/data
networks:
  homer:
    name: homer
    driver: bridge
volumes:
  data:
```
