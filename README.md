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
mkswap /dev/sda1
mkfs.fat -F 32 /dev/sda2
mkfs.ext4 /dev/sda3
```

## Mount the partitions and install the base packages

Mount the partitions to /mnt (again, replace the partition letters according to your hard drive partition letters)

```
lsblk
├─sda1   8:4    0   2.1G  0 part
├─sda2   8:5    0   512M  0 part
├─sda3   8:6    0  51.3G  0 part 
```

```
mount /dev/sda3 /mnt
mkdir /mnt/boot
mkdir /mnt/boot/efi
mount /dev/sda2 /boot/efi
swapon /dev/sda1
```

Install the base packages to /mnt (you can choose any linux kernel but personally i prefer linux-lts since the latest kernel breaks in every 1 wekk on these old macbooks)

```
pacstrap /mnt base linux-lts linux-lts-headers linux-firmware intel-ucode nano
```

Generate the fstab So the partitons gets mounted every time your system boots up

```
genfstab -U /mnt >> /mnt/etc/fstab
```

## Configure the installed system in /mnt

Chroot into /mnt

```
arch-chroot /mnt
```

**Setup the TimeZone (type in your region and city in the correct place)**

```
ln -sf /usr/share/zoneinfo/Region/City /etc/localtime
```

For me it is

```
ln -sf /usr/share/zoneinfo/Asia/Kolkata /etc/localtime
```

generate /etc/adjtime

```
hwclock --systohc
```


