# Arch Linux Install Guide

A short Arch Linux installation guide.

Supports:

- UEFI systems using GPT
- legacy BIOS systems using MBR
- ext4
- a swap partition
- GRUB
- NetworkManager
- AMD or Intel CPUs

Commands are run from the Arch Linux installation environment unless
stated otherwise.

## Keyboard and console font

Load the desired console keymap:

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

```
64
```

the system is booted using 64-bit UEFI.

If `/sys/firmware/efi/fw_platform_size` does not exist, the system
is normally booted using legacy BIOS. This guide does not cover
32-bit UEFI systems.

## Identify the drive

List available block devices:

```sh
lsblk
```

The examples below use `/dev/sdX`. Replace this with the correct
drive for your system.

An NVMe drive appears as `/dev/nvme0n1`, with partitions such as
`/dev/nvme0n1p1`, `/dev/nvme0n1p2`, and `/dev/nvme0n1p3`.

## Partition the drive

Open the target drive:

```sh
fdisk /dev/sdX
```

Choose the layout that matches your boot mode.

### UEFI

Use a GPT partition table.

```
/dev/sdX1  EFI System        1 GiB
/dev/sdX2  Linux swap        Same size as RAM, or larger for hibernation
/dev/sdX3  Linux filesystem  Remaining space
```

### Legacy BIOS

Use an MBR/DOS partition table.

```
/dev/sdX1  Linux swap        Same size as RAM, or larger for hibernation
/dev/sdX2  Linux filesystem  Remaining space
```

If you use GPT instead of MBR on a legacy BIOS system, GRUB requires
a small BIOS boot partition. This guide uses MBR for the BIOS path
to keep the setup simple.

## Format the partitions

### UEFI

```sh
mkfs.ext4 /dev/sdX3
mkswap /dev/sdX2
mkfs.fat -F 32 /dev/sdX1
```

### Legacy BIOS

```sh
mkfs.ext4 /dev/sdX2
mkswap /dev/sdX1
```

No EFI filesystem is required for a legacy BIOS installation.

## Mount the filesystems

### UEFI

```sh
mount /dev/sdX3 /mnt
mount --mkdir /dev/sdX1 /mnt/boot/efi
swapon /dev/sdX2
```

### Legacy BIOS

```sh
mount /dev/sdX2 /mnt
swapon /dev/sdX1
```

## Install the base system

### UEFI

For an AMD CPU:

```sh
pacstrap -K /mnt amd-ucode base base-devel efibootmgr grub linux \
    linux-firmware networkmanager sof-firmware neovim
```

### Legacy BIOS

For an AMD CPU:

```sh
pacstrap -K /mnt amd-ucode base base-devel grub linux \
    linux-firmware networkmanager sof-firmware neovim
```

For an Intel CPU, replace `amd-ucode` with `intel-ucode`.

## Generate fstab

```sh
genfstab -U /mnt >> /mnt/etc/fstab
```

Review it:

```sh
cat /mnt/etc/fstab
```

## Enter the installed system

```sh
arch-chroot -S /mnt
```

The following commands are run inside the chroot.

## Time zone

```sh
ln -sf /usr/share/zoneinfo/Europe/Zagreb /etc/localtime
```

Adjust the region and city as needed.

Synchronize the hardware clock:

```sh
hwclock --systohc
```

## Locale

Open:

```sh
nvim /etc/locale.gen
```

Uncomment the UTF-8 locales you want, for example:

```
en_US.UTF-8 UTF-8
```

Generate them:

```sh
locale-gen
```

Open:

```sh
nvim /etc/locale.conf
```

For example:

```
LANG=en_US.UTF-8
```

## Console keymap

Open:

```sh
nvim /etc/vconsole.conf
```

For example:

```
KEYMAP=us
```

## Hostname

Open:

```sh
nvim /etc/hostname
```

Enter the desired hostname, for example `arch`.

## Root password

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

## Configure sudo

```sh
EDITOR=nvim visudo
```

Uncomment:

```
%wheel ALL=(ALL:ALL) ALL
```

## Enable networking

```sh
systemctl enable NetworkManager
```

## Install GRUB

Choose the command that matches your boot mode.

### UEFI

The EFI System Partition should already be mounted at `/boot/efi`.

```sh
grub-install --target=x86_64-efi --efi-directory=/boot/efi \
    --bootloader-id=GRUB
```

### Legacy BIOS

Install GRUB to the drive itself, not a partition:

```sh
grub-install --target=i386-pc /dev/sdX
```

Use `/dev/sda`, not `/dev/sda1`.

The `i386-pc` target name is also used when installing GRUB for BIOS
on an x86_64 Arch Linux system.

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

Synchronize pending disk writes and reboot:

```sh
sync
reboot
```

Remove the Arch Linux installation media when appropriate.

## Post-installation

After rebooting, log in using the user account created during
installation.

This guide stops at the base operating system installation. Desktop
environments, window managers, and other post-installation
configuration are kept separate.

## References

Written using the official ArchWiki as a technical reference:

- [Installation guide](https://wiki.archlinux.org/title/Installation_guide)
- [Partitioning](https://wiki.archlinux.org/title/Partitioning)
- [GRUB](https://wiki.archlinux.org/title/GRUB)
- [NetworkManager](https://wiki.archlinux.org/title/NetworkManager)
- [Microcode](https://wiki.archlinux.org/title/Microcode)

Arch Linux is rolling-release. Check the current ArchWiki before
installing, in case the official installation procedure has changed.

## License

Made by rabbi-lion.

Original text in this repository is licensed under the Creative
Commons Attribution-ShareAlike 4.0 International License.

Referenced projects and documentation retain their respective
licenses. See `LICENSE` for the full license text.
