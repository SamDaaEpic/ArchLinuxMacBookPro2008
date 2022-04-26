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

## Chroot into /mnt

Chroot into /mnt

```
arch-chroot /mnt
```

## Setup the TimeZone

create a hardlink of our timezone to /etc/localtime (type in your region and city in the correct place)

```
ln -sf /usr/share/zoneinfo/Region/City /etc/localtime
```

Example of region Asia and city Kolkata

```
ln -sf /usr/share/zoneinfo/Asia/Kolkata /etc/localtime
```

generate /etc/adjtime

```
hwclock --systohc
```

## generate Locale

comment out your locale (for me it is en_US.UTF-8 UTF-8)

```
nano /etc/locale.gen
```
Generate the locale you've commented out

```
locale-gen
```

create /etc/locale.conf file (so that your system can know what locale you're using)

```
nano /etc/locale.conf
```

type in /etc/locale.conf (or en_US)

```
LANG=en_US.UTF-8
```

## Configure Hostname

```
nano /etc/hostname
```

```
<your hostname goes here>
```
 
 ## Set the Root Password
 
 ```
 passwd
 ```
 
 ## Setup /etc/hosts 
 
 ```
 nano /etc/hosts
 ```
 
 type in /etc/hosts
 
 ```
127.0.0.1       localhost
 ```
 
 ## Create a User
 
 ```
 useradd <your username> -m --badname
 ```
 
 Add the user to the wheel group and video and audio group
 
 ```
 usermod -aG wheel,video,audio <yourusername>
 ```
 
 Set the user password
 
 ```
 passwd <yourusername>

 ```
 
 ## Install basic packages with xfce and xorg
 
 ```
 pacman -S efibootmgr networkmanager network-manager-applet dialog wpa_supplicant mtools dosfstools base-devel git reflector bluez bluez-utils pulseaudio pulseaudio-bluetooth alsa-utils xdg-utils xdg-user-dirs broadcom-wl-dkms xorg xfce4 xfce4-goodies lightdm
 ```
 
 ## Install and configure GRUB
 
 Install grub
 
 ```
 pacman -S grub
 ```
 
 Install grub to /boot/efi
 
 ```
 grub-install --target=x86_64-efi --efi-directory=/boot/efi
 ```
 
 **This step is very important, if you dont do this you will not see the archlinux in the macbook startup manager**
 
 copy the efi files to the correct directory do that your macbook can detect archlinux
 
 
 ```
 mkdir /boot/efi/EFI/boot
 cp /boot/efi/EFI/arch/grubx64.efi /boot/efi/EFI/boot/bootx64.efi
 ```
 
 Now generate the grub configuration
 
 ```
 grub-mkconfig -o /boot/grub/grub.cfg
 ```
 
 ## Enable needeed systemd services
 
 ```
 systemctl enable bluetooth
 systemctl enable NetworkManager #The Network manager MUST have the "N" and "M" capital
 systemctl enable lightdm
 ```
 
**Now your Installation is Done and You can reboot and follow the steps on how to install the nvidia proprietary drivers**


## Install the nVidia Proprietary drivers 

First of all go into the tty and stop lightdm (because the gui will crash while installing nvidia drivers and we dont want that, we will do everything in tty)

 Press **FN+Ctrl+Alt+F2** to go into tty and login with your username and password (dont login as root)
 
 install yay
 
 ```
 git clone https://aur/archlinux.org/yay.git
 cd yay
 makepkg -si
 ```
 
 After yay has installed, install nvidia drivers with yay
 
 ```
 yay -S nvidia-340xx-dkms nvidia-340xx-utils lib32-nvidia-340xx-utils
 ```
 
 After nvidia drivers are done installing blacklist the nouveau driver so it dosent get loaded on boot
 
 ```
 sudo nano /etc/modprobe.d/blacklist-nouveau.conf
 ```
 
 type this in /etc/modprobe.d/blacklist-nouveau.conf
 
 ```
 blacklist nouveau
 ```
 
 now regenerate the initramfs
 
 ```
 sudo mkinitcpio -P
 ```
 
 Now generate the xorg config or nvidia drivers
 
 ```
 sudo nvidia-xconfig
 ```
 
 Check your gpu PCID with lspci
 
 ```
 lspci
00:00.0 Host bridge: NVIDIA Corporation MCP79 Host Bridge (rev b1)
00:00.1 RAM memory: NVIDIA Corporation MCP79 Memory Controller (rev b1)
00:03.0 ISA bridge: NVIDIA Corporation MCP79 LPC Bridge (rev b2)
00:03.1 RAM memory: NVIDIA Corporation MCP79 Memory Controller (rev b1)
00:03.2 SMBus: NVIDIA Corporation MCP79 SMBus (rev b1)
00:03.3 RAM memory: NVIDIA Corporation MCP79 Memory Controller (rev b1)
00:03.4 RAM memory: NVIDIA Corporation MCP79 Memory Controller (rev b1)
00:03.5 Co-processor: NVIDIA Corporation MCP79 Co-processor (rev b1)
00:04.0 USB controller: NVIDIA Corporation MCP79 OHCI USB 1.1 Controller (rev b1)
00:04.1 USB controller: NVIDIA Corporation MCP79 EHCI USB 2.0 Controller (rev b1)
00:06.0 USB controller: NVIDIA Corporation MCP79 OHCI USB 1.1 Controller (rev b1)
00:06.1 USB controller: NVIDIA Corporation MCP79 EHCI USB 2.0 Controller (rev b1)
00:08.0 Audio device: NVIDIA Corporation MCP79 High Definition Audio (rev b1)
00:09.0 PCI bridge: NVIDIA Corporation MCP79 PCI Bridge (rev b1)
00:0a.0 Ethernet controller: NVIDIA Corporation MCP79 Ethernet (rev b1)
00:0b.0 SATA controller: NVIDIA Corporation MCP79 AHCI Controller (rev b1)
00:0c.0 PCI bridge: NVIDIA Corporation MCP79 PCI Express Bridge (rev b1)
00:10.0 PCI bridge: NVIDIA Corporation MCP79 PCI Express Bridge (rev b1)
00:15.0 PCI bridge: NVIDIA Corporation MCP79 PCI Express Bridge (rev b1)
00:16.0 PCI bridge: NVIDIA Corporation MCP79 PCI Express Bridge (rev b1)
00:17.0 PCI bridge: NVIDIA Corporation MCP79 PCI Express Bridge (rev b1)
02:00.0 VGA compatible controller: NVIDIA Corporation G96CM [GeForce 9600M GT] (rev a1)
03:00.0 VGA compatible controller: NVIDIA Corporation C79 [GeForce 9400M] (rev b1)
04:00.0 Network controller: Broadcom Inc. and subsidiaries BCM4322 802.11a/b/g/n Wireless LAN Controller (rev 01)
05:00.0 FireWire (IEEE 1394): LSI Corporation FW643 [TrueFire] PCIe 1394b Controller (rev 06)

 ```
 
 you can only use the GeForce 9400M as display for your machine, if you use the GeForce 9600M GT it will show a black screen
 
 Now configure /etc/X11/xorg.conf so that you wont get a black screen
 
 ```
 sudo nano /etc/X11/xorg.conf
 ```

Your config should look something like this

```
# nvidia-xconfig: X configuration file generated by nvidia-xconfig
# nvidia-xconfig:  version 340.108  (buildmeister@swio-display-x64-rhel04-01)  Wed Dec 11 15:13:33 PST 2019

# nvidia-settings: X configuration file generated by nvidia-settings
# nvidia-settings:  version 340.108  (buildmeister@swio-display-x64-rhel04-01)  Wed Dec 11 15:13:22 PST 2019

Section "ServerLayout"
    Identifier     "Layout0"
    Screen      0  "Screen0" 0 0
    InputDevice    "Keyboard0" "CoreKeyboard"
    InputDevice    "Mouse0" "CorePointer"
    Option         "Xinerama" "0"
EndSection

Section "Files"
    ModulePath      "/usr/lib64/nvidia/xorg"
    ModulePath      "/usr/lib64/xorg/modules"
EndSection

Section "ServerFlags"
    Option         "IgnoreABI" "1"
EndSection

Section "InputDevice"

    # generated from default
    Identifier     "Mouse0"
    Driver         "mouse"
    Option         "Protocol" "auto"
    Option         "Device" "/dev/psaux"
    Option         "Emulate3Buttons" "no"
    Option         "ZAxisMapping" "4 5"
EndSection

Section "InputDevice"

    # generated from default
    Identifier     "Keyboard0"
    Driver         "kbd"
EndSection

Section "Monitor"
    Identifier     "Monitor0"
    VendorName     "Unknown"
    ModelName      "Apple"
    HorizSync       30.0 - 75.0
    VertRefresh     59.9
    Option         "DPMS"
EndSection

Section "Device"
    Identifier     "Device0"
    Driver         "nvidia"
    VendorName     "NVIDIA Corporation"
    BoardName      "GeForce 9400M"
EndSection

Section "Screen"
    Identifier     "Screen0"
    Device         "Device0"
    Monitor        "Monitor0"
    DefaultDepth    24
    Option         "Stereo" "0"
    Option         "metamodes" "nvidia-auto-select +0+0"
    Option         "NoLogo" "true"
    Option         "MultiGPU" "true"
    Option         "BaseMosaic" "off"
    Option         "SLI" "true"
    Option         "Coolbits" "13"
    SubSection     "Display"
        Depth       24
    EndSubSection
EndSection

```
**Change it to this by adding ignore abi option and adding the gpu pcid**

```
# nvidia-xconfig: X configuration file generated by nvidia-xconfig
# nvidia-xconfig:  version 340.108  (buildmeister@swio-display-x64-rhel04-01)  Wed Dec 11 15:13:33 PST 2019

# nvidia-settings: X configuration file generated by nvidia-settings
# nvidia-settings:  version 340.108  (buildmeister@swio-display-x64-rhel04-01)  Wed Dec 11 15:13:22 PST 2019

Section "ServerLayout"
    Identifier     "Layout0"
    Screen      0  "Screen0" 0 0
    InputDevice    "Keyboard0" "CoreKeyboard"
    InputDevice    "Mouse0" "CorePointer"
    Option         "Xinerama" "0"
EndSection

Section "Files"
    ModulePath      "/usr/lib64/nvidia/xorg"
    ModulePath      "/usr/lib64/xorg/modules"
EndSection

Section "ServerFlags"
    Option         "IgnoreABI" "1"
EndSection

Section "InputDevice"

    # generated from default
    Identifier     "Mouse0"
    Driver         "mouse"
    Option         "Protocol" "auto"
    Option         "Device" "/dev/psaux"
    Option         "Emulate3Buttons" "no"
    Option         "ZAxisMapping" "4 5"
EndSection

Section "InputDevice"

    # generated from default
    Identifier     "Keyboard0"
    Driver         "kbd"
EndSection

Section "Monitor"
    Identifier     "Monitor0"
    VendorName     "Unknown"
    ModelName      "Apple"
    HorizSync       30.0 - 75.0
    VertRefresh     59.9
    Option         "DPMS"
EndSection

Section "Device"
    Identifier     "Device0"
    Driver         "nvidia"
    VendorName     "NVIDIA Corporation"
    BoardName      "GeForce 9400M"
    BusID          "PCI:3:0:0"
EndSection

Section "Screen"
    Identifier     "Screen0"
    Device         "Device0"
    Monitor        "Monitor0"
    DefaultDepth    24
    Option         "Stereo" "0"
    Option         "metamodes" "nvidia-auto-select +0+0"
    Option         "NoLogo" "true"
    Option         "MultiGPU" "true"
    Option         "BaseMosaic" "off"
    Option         "SLI" "true"
    Option         "Coolbits" "13"
    SubSection     "Display"
        Depth       24
    EndSubSection
EndSection

```
  
## Nvidia proprietary drivers installtion is done you can now reboot and start using ARCHLINUX


