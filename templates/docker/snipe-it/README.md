# Templates - Docker - Snipe-IT

[Back](../README.md)

## Introduction

🚨 **Change the passwords from the defaults!** 🚨

## Docker Compose

Create a new directory, `snipe-it`, and a new file within named `docker-compose.yaml`:

```yaml
# Compose file for production.

volumes:
  db_data:
  storage:

services:
  app:
    image: snipe/snipe-it:v8.0.3
    restart: unless-stopped
    volumes:
      - storage:/var/lib/snipeit
    ports:
      - "${APP_PORT:-8000}:80"
    depends_on:
      db:
        condition: service_healthy
        restart: true
    env_file:
      - .env

  db:
    image: mariadb:11.5.2
    restart: unless-stopped
    volumes:
      - db_data:/var/lib/mysql
    environment:
      MYSQL_DATABASE: snipeit_db
      MYSQL_USER: mysql
      MYSQL_PASSWORD: password
      MYSQL_ROOT_PASSWORD: password
    healthcheck:
      # https://mariadb.com/kb/en/using-healthcheck-sh/#compose-file-example
      test: ["CMD", "healthcheck.sh", "--connect", "--innodb_initialized"]
      interval: 5s
      timeout: 1s
      retries: 5
```
