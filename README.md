# ArchLinuxMacBookPro2008
How to install Arch Linux on MacBook Pro 2008 with nvidia proprietary drivers.

## Getting Ready

Download the ArchLinux iso: https://archliinux.org/download

Boot into the archlinux iso by pluging in the usb drive (to which you have flashed the arch iso) into the macbook and holding down the Option key on the keyboard

## Connect to WIFI

First of all unload all the other wifi drivers

```
rmmod b43 bcma ssb
```

