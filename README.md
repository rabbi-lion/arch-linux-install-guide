# Arch Linux Install Guide

A minimal Arch Linux installation guide using GRUB and NetworkManager.

This guide supports:

- UEFI systems using GPT
- legacy BIOS systems using MBR
- ext4
- a swap partition
- GRUB
- NetworkManager
- AMD or Intel CPUs

The commands are intended to be run from the Arch Linux installation environment unless stated otherwise.

> **Warning:** Partitioning and formatting a drive will destroy existing data on the selected partitions. Verify the target drive before continuing.

## Keyboard and console font

Load the desired console keymap:

```sh
loadkeys [keymap]
```

For example:

```sh
loadkeys us
```

Set the console font:

```sh
setfont ter-132b
```

## Check the boot mode

Check whether the installation media was booted using UEFI:

```sh
cat /sys/firmware/efi/fw_platform_size
```

If the command returns:

```text
64
```

the system is booted using 64-bit UEFI.

If `/sys/firmware/efi/fw_platform_size` does not exist, the system is normally booted using legacy BIOS mode.

This guide does not cover 32-bit UEFI systems.

## Identify the drive

List available block devices:

```sh
lsblk
```

The examples below use:

```text
/dev/sdX
```

Replace this with the correct drive for your system.

An NVMe drive may instead appear as:

```text
/dev/nvme0n1
```

with partitions such as:

```text
/dev/nvme0n1p1
/dev/nvme0n1p2
/dev/nvme0n1p3
```

## Partition the drive

Open the target drive:

```sh
fdisk /dev/sdX
```

Choose the partition layout that matches your boot mode.

### UEFI

Use a GPT partition table.

Example:

```text
/dev/sdX1  EFI System        1 GiB
/dev/sdX2  Linux swap        Same size as RAM, or larger if required for hibernation
/dev/sdX3  Linux filesystem  Remaining space
```

### Legacy BIOS

Use an MBR/DOS partition table.

Example:

```text
/dev/sdX1  Linux swap        Same size as RAM, or larger if required for hibernation
/dev/sdX2  Linux filesystem  Remaining space
```

If you use GPT instead of MBR on a legacy BIOS system, GRUB requires a small BIOS boot partition. This guide uses MBR for the BIOS installation path to keep the setup simple.

## Format the partitions

### UEFI

Format the root partition:

```sh
mkfs.ext4 /dev/sdX3
```

Initialize swap:

```sh
mkswap /dev/sdX2
```

Format the EFI System Partition as FAT32:

```sh
mkfs.fat -F 32 /dev/sdX1
```

### Legacy BIOS

Format the root partition:

```sh
mkfs.ext4 /dev/sdX2
```

Initialize swap:

```sh
mkswap /dev/sdX1
```

No EFI filesystem is required for a legacy BIOS installation.

## Mount the filesystems

### UEFI

Mount the root filesystem:

```sh
mount /dev/sdX3 /mnt
```

Mount the EFI System Partition:

```sh
mount --mkdir /dev/sdX1 /mnt/boot/efi
```

Enable swap:

```sh
swapon /dev/sdX2
```

### Legacy BIOS

Mount the root filesystem:

```sh
mount /dev/sdX2 /mnt
```

Enable swap:

```sh
swapon /dev/sdX1
```

## Install the base system

### UEFI

For an AMD CPU:

```sh
pacstrap -K /mnt amd-ucode base base-devel efibootmgr grub linux linux-firmware networkmanager sof-firmware neovim
```

### Legacy BIOS

For an AMD CPU:

```sh
pacstrap -K /mnt amd-ucode base base-devel grub linux linux-firmware networkmanager sof-firmware neovim
```

For an Intel CPU, replace:

```text
amd-ucode
```

with:

```text
intel-ucode
```

## Generate fstab

Generate the filesystem table using UUIDs:

```sh
genfstab -U /mnt >> /mnt/etc/fstab
```

Review it:

```sh
cat /mnt/etc/fstab
```

## Enter the installed system

Change root into the new installation:

```sh
arch-chroot -S /mnt
```

The following commands are run inside the chroot.

## Time zone

Set the time zone:

```sh
ln -sf /usr/share/zoneinfo/Area/Location /etc/localtime
```

For example:

```sh
ln -sf /usr/share/zoneinfo/Europe/Zagreb /etc/localtime
```

Synchronize the hardware clock:

```sh
hwclock --systohc
```

## Locale

Open:

```sh
nvim /etc/locale.gen
```

Uncomment the UTF-8 locales you want to use.

For example:

```text
en_US.UTF-8 UTF-8
```

Generate the selected locales:

```sh
locale-gen
```

Open:

```sh
nvim /etc/locale.conf
```

For example:

```text
LANG=en_US.UTF-8
```

## Console keymap

Open:

```sh
nvim /etc/vconsole.conf
```

For example:

```text
KEYMAP=us
```

## Hostname

Open:

```sh
nvim /etc/hostname
```

Enter the desired hostname.

For example:

```text
arch
```

## Root password

Set the root password:

```sh
passwd
```

## Create a user

Create a normal user and add it to the `wheel` group:

```sh
useradd -m -G wheel -s /bin/bash [username]
```

Set the user's password:

```sh
passwd [username]
```

Replace `[username]` with the desired username.

## Configure sudo

Open the sudoers configuration:

```sh
EDITOR=nvim visudo
```

Uncomment:

```text
%wheel ALL=(ALL:ALL) ALL
```

## Enable networking

Enable NetworkManager:

```sh
systemctl enable NetworkManager
```

## Install GRUB

Choose the command that matches your boot mode.

### UEFI

The EFI System Partition should already be mounted at:

```text
/boot/efi
```

Install GRUB:

```sh
grub-install --target=x86_64-efi --efi-directory=/boot/efi --bootloader-id=GRUB
```

### Legacy BIOS

Install GRUB to the drive itself, not a partition:

```sh
grub-install --target=i386-pc /dev/sdX
```

For example:

```text
/dev/sda
```

not:

```text
/dev/sda1
```

The `i386-pc` target name is also used when installing GRUB for BIOS on an x86_64 Arch Linux system.

### Generate the configuration

For either boot mode:

```sh
grub-mkconfig -o /boot/grub/grub.cfg
```

## Finish the installation

Exit the chroot:

```sh
exit
```

Unmount the filesystems:

```sh
umount -a
```

Synchronize pending disk writes:

```sh
sync
sync
```

Reboot:

```sh
reboot
```

Remove the Arch Linux installation media when appropriate.

## Post-installation

After rebooting, log in using the user account created during installation.

This guide intentionally stops at the base operating system installation. Desktop environments, window managers and other post-installation configuration are kept separate.

## References

This guide was written independently using the official ArchWiki as a technical reference.

Relevant documentation:

- [Installation guide](https://wiki.archlinux.org/title/Installation_guide)
- [Partitioning](https://wiki.archlinux.org/title/Partitioning)
- [GRUB](https://wiki.archlinux.org/title/GRUB)
- [NetworkManager](https://wiki.archlinux.org/title/NetworkManager)
- [Microcode](https://wiki.archlinux.org/title/Microcode)

Arch Linux is a rolling-release distribution. Check the current ArchWiki before installing in case the official installation procedure has changed.

## License

Made by rabbi-lion.

Original text in this repository is licensed under the Creative Commons Attribution-ShareAlike 4.0 International License.

Referenced projects and documentation retain their respective licenses.

See `LICENSE` for the full license text.
