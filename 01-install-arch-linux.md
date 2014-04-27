01. Install Arch Linux
-----------------------------

Boot the Install Media
=======================

If you were following the preceding steps and have kept the USB install media plugged in, reboot the computer.

When `rEFInd` presents the boot choices, you should see the live USB's partitions as choices. For me, the correct selection was the last one. Selecting that lead to the live USB's instance of GRUB.

Select the first option (something like "Arch Linux UEFI").

After up to a minute, the root login should appear.


Partition the SSD
======================

In this stage, we'll be setting up the partition table.

Start up `cgdisk`

    cgdisk /dev/sda

### Apple boot loader partition

Scroll down to "free space", then press the Enter key.

You will see something like "First sector size". Just press enter to move on to setting the size of the first partition.

Type in the size as `128M`.

Then you'll be prompted to set what type of file system it should be.

Type in the code `af00`.

Set the name as `Apple boot loader`.

### /boot partition

Next, scroll down again to free space, then press the Enter key.

Skip the sector thing by pressing Enter.

Type in the size as `256M`.

Set the file system as `8300`.

Set the name as `boot`.

### / (root) partition

Scroll again to the free space, then press the Enter key.

Press Enter again to skip the sector thing.

Set the size in GB. (e.g. `50G`).

Set the file system as `8300`.

Set the name as `root`.

### /home partition

Finally, scroll down to the remaining free space and then press the Enter key.

Press Enter again to skip the sector thing.

Set the size in GB. (e.g. `100G`).

Set the file system as `8300`.

Set the name as `home`.

### Write partitions

Then write the changes to disk by selecting `Write` in the bottom menu of `cgdisk`.


Partition Encryption (optional, though highly recommended)
============================================================

In this step, I'll be encrypting my `/home` partition (which corresponds to `/dev/sda7`) with `LUKS`.

Run

    cryptsetup -c aes-xts-plain64 -y -s 512 luksFormat /dev/sda7

You will need to type in `YES` in `ALL CAPS` to confirm that you want to do the above step.

You will then be prompted to set a pass phrase. Note that you will need to type this every time you boot into your system, so make sure to write it down!

Then run the following to create a mapping to the `/home` partition.

    cryptsetup luksOpen /dev/sda7 home


Formatting the system
========================

Now we will need to actually create the filesystems for each partition we created in a previous step.

This will be done with the `mkfs` command:

    mkfs.ext2 /dev/sda5
    mkfs.ext4 /dev/sda6
    mkfs.ext4 /dev/mapper/home

NOTE: If you chose not to encrypt the `home` partition, replace `/dev/mapper/home` with `/dev/sda7`.


Installing the base Arch Linux system
======================================

We'll need to mount the newly formatted partitions to install the base arch linux system.

First, create some directories to which to mount the partitions:

    mkdir /mnt/{boot,home}

Then mount each partition to those directories:

    mount /dev/sda6 /mnt
    mount /dev/sda5 /mnt/boot
    mount /dev/mapper/home /mnt/home

If you encrypted the home partition, you will need to enter the passphrase to decrypt it to mount.

Run the arch setup script:

    pacstrap /mnt base base-devel

Then, generate the `fstab`:

    genfstab -p /mnt >> /mnt/etc/fstab

Setup the encrypted partition as well:

    echo 'home /dev/sda7' >> /mnt/etc/crypttab


Modifying the fstab for optimized I/O on the SSD
================================================

Use a text editor like `nano` to modify the fstab:

    nano /mnt/fstab

The settings will need to look something like this:

```
/dev/sda5    /boot  ext2  defaults,relatime,stripe=4    0 2
/dev/sda6    /      ext4  defaults,noatime,discard,data=writeback    0 1
/dev/mapper/home    /home ext4  defaults,noatime,discard,data=ordered    0 2
```

NOTE: If you did not encrypted `/home`, replace `/dev/mapper/home` to `/dev/sda7`.

Basic configuration (before first boot)
==================================================

Run `arch-chroot` to do stuff on the newly installed base system:

    arch-chroot /mnt /bin/bash

Once you are logged into the new system as the root user, set the hostname:

    echo cleverhostnamehere > /etc/hostname

(This is where I often get stuck, since I can never seem to think of a creative hostname).

Then set the local time zone (which for me was "Chicago"):

    ln -s /usr/share/zoneinfo/America/Chicago /etc/localtime
    hwclock --systohc --utc

Create the first user and set the password:

    useradd -m -g users -G wheel -s /bin/bash hckr
    passwd hckr

Setup the locale by uncommenting the lines corresponding to your locale(s) in `/etc/locale.gen`. I uncommented `en_US.UTF-8 UTF-8`. Then generate the locale:

    locale-gen
    echo LANG=en_US.UTF-8 > /etc/locale.conf
    export LANG=en_US.UTF-8

Modify `mkinitcpio.conf` to change the order of the `HOOK` section. Put `keyboard` after `autodetect`. Now run

    mkinitcpio -p linux
