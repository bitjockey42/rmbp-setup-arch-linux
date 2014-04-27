03. Graphics
-------------------

Driver Installation
====================

For this installation, I decided I will probably need the discrete GPU at _some_ point in time, so I did not disable it completely. 

I installed drivers for both the integrated Intel card and the Nvidia GPU.

    pacman -S xf86-video-nouveau xf86-video-intel

X Server
====================

And then installed 

    pacman -S xorg-server xorg-xinit xorg-server-utils

Disabling the discrete GPU and running the integrated GPU primarily
=====================================================================

This is where it got a bit tricky for me. I wanted to run the integrated GPU primarily for better battery life. This required going back into OS X to run `gfxCardStatus` (version 2.2.1, NOT 2.3 or 2.1) and force the machine to stick to the integrated card.

First, I needed to install `vgaswitcheroo` (a utility with which to ) from the `AUR`. (I use `packer` as my `AUR` wrapper):

    packer -S systemd-vgaswitcheroo-units

Once installed, it needs to be enabled to start up on boot:

    systemctl enable vgaswitcheroo

Then start it:

    systemctl start vgaswitcheroo

To turn off the discrete GPU:

    echo OFF > /sys/kernel/debug/vgaswitcheroo/switch

If you are logged in as a regular user, you may need to become root to do the above:

    sudo su

You should not need to specify any special configuration in `/etc/X11/xorg.conf` to get the intel card running as the only GPU on the next boot.

If you encounter a black screen on boot, then the graphics card will need to be switched to the integrated card in OS X using `gfxCardStatus` as this means the linux system is still trying to get ahold of the discrete GPU.


If you want to run the discrete graphics card
==============================================

I generally don't have a need for the discrete card as I don't use an external monitor.

However, I needed something like this in my `/etc/X11/xorg.conf` in order to get the Nvidia card running with the nouveau drivers:

```
Section "Device"
    Identifier     "Device0"
    Driver         "nouveau"
    VendorName     "NVIDIA Corporation"
EndSection
```

For KMS, the `MODULES` array would need to be modified like so in the `/etc/mkinitcpio.conf`:

    MODULES="nouveau"

`mkinitcpio` would need to be run:

    mkinitcpio -p linux

Then, to turn on the discrete GPU with vgaswitcheroo:

    echo ON > /sys/kernel/debug/vgaswitcheroo/switch

Then, I'd have to reboot into OS X and set the graphics card to the discrete GPU with `gfxCardStatus`.
