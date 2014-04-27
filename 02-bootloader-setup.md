03 Bootloader Setup
=============================

Install `grub`:

    pacman -S grub

Modify `/etc/default/grub`:

    GRUB_CMDLINE_LINUX_DEFAULT="quiet rootflags=data=writeback"

Generate the grub config onto the `boot` folder on the new system:

    grub-mkconfig -o boot/grub/grub.cfg
    grub-mkstandalone -o boot.efi -d usr/lib/grub/x86_64-efi -O x86_64-efi boot/grub/grub.cfg

