# Installing Docker on Arch Linux

## The quick way (with Portainer)

First, add `docker` to the commands that don't require `sudo` in `~/.config/shell/aliasrc`.

Update packages and install `docker`:

```
pacman -Sy && pacman -S docker
```

Enable and start the `docker` service:

```
systemctl enable docker && systemctl start docker
```

Create a volume to store data from `portainer`:

```
docker volume create portainer_data
```

Create the `portainer` container:

```
docker run -d -p 9443:9443 -p 8000:8000 --name portainer --restart=always -v /var/run/docker.sock:/var/run/docker.sock -v portainer_data:/data portainer/portainer-ce:latest
```

Check to confirm the container is running:

```
docker ps
```

## References

- [YouTube - NetworkChuck - Portainer](https://www.youtube.com/watch?v=iX0HbrfRyvc)
- [YouTube - NetworkChuck - Docker Containers 101](https://www.youtube.com/watch?v=eGz9DS-aIeY&list=PLIhvC56v63IJlnU4k60d0oFIrsbXEivQo)
