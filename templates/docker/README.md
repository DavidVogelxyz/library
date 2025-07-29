# Docker related templates

[Back to "Templates"](../README.md)

## Introduction

These documents and files are meant to assist with the installation and management of various Docker containers.

## Table of contents

- [Introduction](#introduction)
- [Installation](#installation)
- [Table of containers](#table-of-containers)
- [References](#references)

## Installation

🚨 **NB: Remember to enable the `docker` service after installation!** 🚨

Reference the following table for the advised package to install with the distro's default package manager:

| Distribution | Package manager | Package name | Docker Compose package |
| ---          | ---             | ---          | ---                    |
| Archlinux    | `pacman`        | `docker`     | ❓                     |
| Debian       | `apt`           | `docker.io`  | `docker-compose`       |

In addition, if using [these dotfiles](https://github.com/DavidVogelxyz/dotfiles), there is a function in `.config/shell/aliasrc` that prefixes `sudo` to numerous commands -- add `docker` to that list of commands.

**All documents and files within this tree will display Docker commands as if the user is `root`. It is up to the reader to prefix commands with `sudo` where necessary.**

## Table of containers

The following table lists the containers in this tree and their methods of deployment. All `✅` are linked to the relevant information.

🚨 **NB: Start with Portainer, as it makes managing Docker containers easy.** 🚨

| Name          | Docker command                              | Docker Compose yaml                               |
| ---           | ---                                         | ---                                               |
| Dashkiosk     | [✅ - here](dashkiosk/README.md#commands)   | [✅ - here](dashkiosk/README.md#docker-compose)   |
| Forgejo       | ❌                                          | [✅ - here](forgejo/README.md#docker-compose)     |
| Grafana       | [✅ - here](grafana/README.md#commands)     | [✅ - here](grafana/README.md#docker-compose)     |
| Homer         | [✅ - here](homer/README.md#commands)       | [✅ - here](homer/README.md#docker-compose)       |
| Odoo          | ❌                                          | [✅ - here](odoo/README.md#docker-compose)        |
| **Portainer** | [✅ - here](portainer/README.md#commands)   | [✅ - here](portainer/README.md#docker-compose)   |
| Snipe-IT      | ❌                                          | [✅ - here](snipe-it/README.md#docker-compose)    |
| Vaultwarden   | [✅ - here](vaultwarden/README.md#commands) | [✅ - here](vaultwarden/README.md#docker-compose) |

## References

- [YouTube - NetworkChuck - Portainer](https://www.youtube.com/watch?v=iX0HbrfRyvc)
    - Original reference from 2023.
- [YouTube - NetworkChuck - Docker Containers 101](https://www.youtube.com/watch?v=eGz9DS-aIeY&list=PLIhvC56v63IJlnU4k60d0oFIrsbXEivQo)
    - Additional reference from 2023.
