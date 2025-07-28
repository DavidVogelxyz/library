# Templates - Docker - Odoo

[Back](../README.md)

## Introduction

🚨 **Change the passwords from the defaults!** 🚨

## Docker Compose

Create a new directory, `odoo`, and a new file within named `docker-compose.yaml`:

```yaml
version: '3.1'
services:
  db:
    container_name: odoo_db
    image: postgres:15
    restart: always
    environment:
      - POSTGRES_DB=postgres
      - POSTGRES_USER=odoo
      - POSTGRES_PASSWORD=odoo
      - PGDATA=/var/lib/postgresql/data/pgdata
    networks:
      - odoo
    volumes:
      - db-data:/var/lib/postgresql/data/pgdata
  web:
    container_name: odoo_web
    image: odoo:latest
    restart: always
    depends_on:
      - db
    networks:
      - odoo
    ports:
      - '8085:8069'
    volumes:
      - web-data:/var/lib/odoo
networks:
  odoo:
    name: odoo
    driver: bridge
volumes:
  db-data:
  web-data:
```
