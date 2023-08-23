# Install Docker on Arch Linux

## The Quick Way (with Portainer)

Add Docker to the allowed commands in `.config/shell/aliasrc`.

Also, confirm `pacman` alias:

```
alias pacman="p"

p -S docker

systemctl enable docker

systemctl start docker

docker volume create portainer_stuff

docker run -d -p 9443:9443 -p 8000:8000 --name portainer --restart=always -v /var/run/docker.sock:/var/run/docker.sock -v portainer_stuff:/data portainer/portainer-ce:latest

docker ps
```
