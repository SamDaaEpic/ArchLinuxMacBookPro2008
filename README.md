# ArchLinuxMacBookPro2008
How to install Arch Linux on MacBook Pro 2008 with nvidia proprietary drivers.

## Getting Ready

Download the ArchLinux iso: https://archliinux.org/download

Boot into the archlinux iso by pluging in the usb drive (to which you have flashed the arch iso) into the macbook and holding down the Option key on the keyboard

## Connect to WIFI

First of all unload all the other wifi drivers

```
rmmod b43 bcma ssb wl
```

Load the wifi driver for the macbook

```
modprobe wl
```

restart iwd (so it can recognize the wifi card)

```
systemctl restart iwd
```

Connect to wifi with iwctl (type the station commands in the iwctl shell)

```
iwctl
station wlan0 get-networks
station wlan0 connect "<name of the wifi network>"
```

## Correct the system clock in the archiso

Make sure the system clock is accurate in the archiso

```
timedatetl set-ntp true
```

## Setup the partitions

**This is only for if you dont have macos installed**

Create 3 partitions of **2GB** for swap, **512MB** for boot efi partition and **xxxGB** for the root partition

```
cfdisk
```
 Now create the filesystems on the partitions you've created (change the partition letter according to your letters)

```
lsblk
├─sda1   8:4    0   2.1G  0 part
├─sda2   8:5    0   512M  0 part
├─sda3   8:6    0  51.3G  0 part 
```

```
mkfs.ext4 /dev/sda3
mkfs.fat -F 32 /dev/sda2
mkswap /dev/sda1
```





