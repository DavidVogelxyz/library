# Security Hardening on Arch Linux

## Getting Started

Pull an aliasrc file from GitHub, etc. Example: [aliasrc](https://github.com/DavidVogelxyz/dotfiles/blob/main/.config/shell/aliasrc-arch-server)

Pull a shell rc file from GitHub, etc. Example: [bashrc](https://github.com/DavidVogelxyz/dotfiles/blob/main/.config/shell/bashrc)

Create the following directories:

```
mkdir -pv ~/.config/shell ~/.cache/shell
```

## Packages for all computers

## Packages for servers

```
ufw

htop

wget

tor

fail2ban

neofetch
```

## Packages for specific servers

```
nginx
```

## Configure UFW

```
sudo ufw default deny incoming

sudo ufw default allow outgoing
```

Choose whether to allow or deny `ssh` access:

```
sudo ufw [allow/deny] ssh
```

```
sudo ufw logging off

sudo ufw enable

sudo ufw status

systemctl enable ufw
```

## Configure sshd

```
sudo nvim /etc/ssh/sshd_config
```

### The three necessary edits (change TO these value)

1. `PermitRootLogin no`
1. `PasswordAuthentication no`
1. `KbdInteractiveAuthentication no`

### ssh-copy-id

Very important to remember to `ssh-copy-id` before running the restart command!

```
ssh-copy-id

systemctl restart sshd
```

## Security check

```
last

sudo lastb
```

## Update hostname (if applicable)

```
hostnamectl

sudo hostnamectl set-hostname $HOST

hostnamectl
```

## Update timedate (if applicable)

```
`timedatectl`
```

`America/New_York` is the same as `US/Eastern`

```
`timedatectl`
```
