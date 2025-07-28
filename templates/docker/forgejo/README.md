# Templates - Docker - Forgejo

[Back](../README.md)

## Docker Compose

Create a new directory, `forgejo`, and a new file within named `docker-compose.yaml`:

```yaml
services:
  server:
    container_name: forgejo
    image: codeberg.org/forgejo/forgejo:10
    restart: always
    environment:
      - USER_UID=1000
      - USER_GID=1000
    networks:
      - forgejo
    ports:
      - '222:22'
      - '8082:3000'
    volumes:
      - ./forgejo:/data
      - /etc/timezone:/etc/timezone:ro
      - /etc/localtime:/etc/localtime:ro
networks:
  forgejo:
    name: forgejo
    driver: bridge
```
