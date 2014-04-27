04. Wifi
----------------------

Install broadcom-wl
=======================

On the latest kernels `3.1.x`, I found that the [broadcom-wl](https://aur.archlinux.org/packages/broadcom-wl/) drivers from the `AUR` worked well.

These modules had to be blacklisted in `/etc/modprobe.d/blacklist.conf` (which you may need to create):

```
blacklist b43
blacklist bcma
blacklist brcmsmac
```

To load the new module `wl`, first the other broadcoms will need to be unloaded:

    sudo modprobe -r b43 bcma brcmsmac

Then load `wl`:

    sudo modprobe wl

To check if it was loaded correctly:

    dmesg|grep wlan0

You may find there is a line saying `renamed network interface wlan0 to wlp4s0`. This `wlp4s0` is what you will need to refer to in the next few commands.

If you need to know what the other interfaces are named, run `ip addr`.

e.g. My thunderbolt-adaptor for ethernet, which normally would be called `eth0`, was renamed to `ens9`.

Setup NetworkManager
=======================

You can use whatever the hell you want as a wifi menu, even, say, `wifi-menu`. For me, though, I just want this part to be relatively graphic.

Install the `nm-applet`:

    pacman -S networkmanager network-manager-applet gnome-keyring seahorse

Set down the network interfaces:

    systemctl stop dhcpcd@wlp4s0
    systemctl stop dhcpcd@ens9
    systemctl disable dhcpcd@wlp4s0
    systemctl disable dhcpcd@ens9
    ip link set ens9 down
    ip link set wlp4s0 down
    ip link set ens9 up
    ip link set wlp4s0 up

Then put this somewhere in the `~/.xinitrc`:

```
# Start a D-Bus session
source /etc/X11/xinit/xinitrc.d/30-dbus
# Start GNOME Keyring
eval $(/usr/bin/gnome-keyring-daemon --start --components=gpg,pkcs11,secrets,ssh)
# You probably need to do this too:
export SSH_AUTH_SOCK
export GPG_AGENT_INFO
export GNOME_KEYRING_CONTROL
export GNOME_KEYRING_PID
```
