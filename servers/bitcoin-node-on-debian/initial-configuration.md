# Initial configuration

[Back to the home page](README.md)

## Table of contents

- [First steps](#First-steps)
- [Preventing laptop lid close from suspending processes](#Preventing-laptop-lid-close-from-suspending-processes)
- [Configuring zsh](#Configuring-zsh)
    - [lf](#lf)
- [Finalizing initial configuration](#Finalizing-initial-configuration)
- [References](#References)

## First steps

The first step to creating a Bitcoin node would be to install Debian onto the computer that will be used as the node. To assist with this setup process, please refer to [this guide](/install-os/install-debian.md) that details how to install Debian. Also, please note that [this "debian-setup" script](https://github.com/DavidVogelxyz/debian-setup) will help to speed up the process.

Once logged in, follow this guide on [configuring a server running Debian](/servers/configuring-debian-server.md) to secure the SSH connection, as well as to add some configuration files. Only return to this guide once those steps have been completed.

Note: since it's likely that the "install Debian" guide was used to set up the node, then a user should have already been created.

## Preventing laptop lid close from suspending processes

Next, if using a laptop as a node, open the `/etc/systemd/logind.conf` file with a text editor, such as `vim`:

```
sudo vim /etc/systemd/logind.conf
```

Set the `HandleLidSwitch` option to ignore. It will likely be the only uncommented line in the file:

```
HandleLidSwitch=ignore
```

Save the file, and restart the node. Now, the laptop lid can be closed without the computer suspending.

## Configuring zsh

Next, install the following packages to configure `zsh`:

```
nala install -y bat fzf lf stow ueberzug zsh zsh-syntax-highlighting
```

As a note:

- Packages for `zsh` prompt:
    - `zsh`, `zsh-syntax-highlighting`
- Packages for setting up dotfiles:
    - `stow`
- Packages for `tmux` and `tmux-sessionizer`:
    - `fzf`
- Packages for `lf`:
    - `lf`, `ueberzug`, `bat`

Confirm that any missing directories exist with the following command:

```
mkdir -pv ~/.cache/zsh ~/.local/bin
```

Set the user's preferred shell to `zsh` with the following command:

```
chsh -s /bin/zsh
```

Now, clone the Powerlevel10k Git repository into the `~/.local/src` directory:

```
git clone --depth=1 https://github.com/romkatv/powerlevel10k.git ~/.local/src/powerlevel10k
```

In order to quickly download and install the fonts necessary to make Powerlevel10k work, run the following script in the terminal:

```
fonts=(
"MesloLGS_NF_Regular.ttf"
"MesloLGS_NF_Bold.ttf"
"MesloLGS_NF_Italic.ttf"
"MesloLGS_NF_Bold_Italic.ttf"
)

echo "There are $(echo ${#fonts[@]}) fonts to install. It shouldn't take long."
echo "Elevated privileges are required to properly install the fonts. Please enter your password if asked."

sudo mkdir -p /usr/local/share/fonts/m

for font in "${fonts[@]}"; do
    webfont=$(echo $font | sed 's/_/%20/g')
    [ ! -e /usr/local/share/fonts/m/$font ] \
        && echo "now installing $font ..." \
        && sudo curl -L https://github.com/romkatv/powerlevel10k-media/raw/master/$webfont -o /usr/local/share/fonts/m/$font > /dev/null 2>&1
done
```

Since personal dotfiles should already exist in the `~/.dotfiles` directory, they can be deployed in their entirety using `stow`. Alternatively, to get `zsh` working, symlink only the `~/.dotfiles/.config/zsh/.zshrc` and `~/.dotfiles/.config/zsh/.p10k.zsh` files into the `~/.config/zsh`.

To deploy the whole suite of dotfiles, change directory into the `~/.dotfiles` directory and deploy them using `stow .`:

```
cd ~/.dotfiles && stow .
```

If there are any conflicts, be sure to address them.

If desired, change the color of the user's prompt by editing the `POWERLEVEL9K_USER_FOREGROUND` option in the `~/.config/zsh/.p10k.zsh` file. Also, it is possible to change the prompt's OS icon to a Bitcoin logo. Add the following to the `os_icon: os identifier` section:

```
typeset -g POWERLEVEL9K_OS_ICON_CONTENT_EXPANSION=''
```

Note: the emoji in the above command may not render properly on GitHub. However, if the "MesloLGS_NF" fonts are installed on the computer, then the configuration should work as intended.

Now that `zsh` has been configured, begin using the `zsh` shell with the following command.

```
zsh
```

### lf

The other utility worth noting is `lf`, a command line file explorer. To get it working on Debian, the `~/.config/lf/lfrc` file needs a small edit. The easiest way to achieve that edit is with the following command:

```
sed -i "s/'~\/.config\/lf\/scope/'~\/.config\/lf\/scope-debian/g" ~/.config/lf/lfrc
```

`lf` can be invoked with the `lfub` command; alternatively, use the keymap "Ctrl+O" when using the command line.

## Finalizing initial configuration

Create a new directory "/data". This directory will contain symlinks to all configuration directories for all the services that will be installed on the node.

```
sudo mkdir /data && sudo chown <USERNAME>: /data
```

## References

- [Raspibolt - System Configuration](https://raspibolt.org/guide/raspberry-pi/system-configuration.html)
    - Reference for creating a `/data` directory.
