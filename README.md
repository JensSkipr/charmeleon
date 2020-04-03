# Charmeleon
Laptop with Arch and Sway

## 1. Install Arch

```bash
# Load Colemak keyboard layout
loadkeys colemak

# Partition disk with fdisk
# 1. Create new GPT partition table with "g"
# 2. Create following partitions with "n" and "t" for type:
#   1. 512M EFI partition (type 1)
#   2. 260G Linux partition (type default)
# 3. Write changes with "w"

# Set install disk
INSTALL_DISK=/dev/sdX
EFI_PART=${INSTALL_DISK}1
CRYPT_PART=${INSTALL_DISK}2

# Setup full disk encryption
# Using LUKS v1 as v2 is not yet supported by GRUB2
cryptsetup luksFormat --type luks1 ${CRYPT_PART}
cryptsetup open ${CRYPT_PART} cryptlvm

# Setup LVM
pvcreate /dev/mapper/cryptlvm
vgcreate linux /dev/mapper/cryptlvm
lvcreate -L 10G linux -n swap
lvcreate -L 32G linux -n root
lvcreate -l 100%FREE linux -n home

# Create filesystems
mkfs.fat -F32 ${EFI_PART}
mkswap /dev/linux/swap
mkfs.ext4 -L root /dev/linux/root
mkfs.ext4 -L home /dev/linux/home

# Install Arch
# Follow the official install wiki page
# NOTES:
# - Make sure to mount your EFI partition on /mnt/efi and home
#   partition at /mnt/home
# - Append following packages to pacstrap:
#   base-devel, vim, lvm2, man-db, man-pages, texinfo,
#   networkmanager, grub, efibootmgr

# Update GRUB config
vim /etc/default/grub
# Uncomment GRUB_ENABLE_CRYPTODISK=y
# Append "lvm" to GRUB_PRELOAD_MODULES
# Add following to GRUB_CMDLINE_LINUX:
# "cryptdevice=UUID=<lvm-dev-uuid>:cryptlvm root=/dev/linux/root"

# Generate GRUB config
mkdir -p /boot/grub
grub-mkconfig -o /boot/grub/grub.cfg

# Install GRUB
grub-install --target=x86_64-efi --efi-directory=/efi --bootloader-id=GRUB

# Reboot
exit # Exit chroot
umount -R /mnt
poweroff
```

## 2. Setup system

```
# Connect to internet
systemctl start NetworkManager
systemctl enable NetworkManager

# Update repos and software
pacman -Syu

# Install basic software
pacman -S git openssh

# Add new admin user
USERNAME=<username>
pacman -S sudo
addgroup sudo
useradd -m -G sudo -s /bin/bash ${USERNAME}
passwd ${USERNAME}
EDITOR=vim visudo # Uncomment line "%sudo ALL=(ALL) ALL"
exit

# Login with new user

# Disable root
sudo passwd -ld root

# Install Sway
sudo pacman -S sway dmenu swaylock swayidle swaybg alacritty
sudo pacman -S xorg-server-xwayland qt5-wayland

# TODO: Copy Sway config
mkdir -p ~/.config/sway
cp ./config/sway ~/.config/sway/config

# Install YAY
sudo pacman -S fakeroot
# TODO: Install YAY

# Install Greetd
# See https://wiki.archlinux.org/index.php/Greetd
```

## 3. Setup applications
```bash
# Install applications
sudo pacman -S firefox keepassxc nextcloud-client
```