# Arch Linux Install Guide

A minimal Arch Linux installation guide using GRUB and NetworkManager.

This guide covers a basic installation with:

- UEFI
- GRUB
- ext4
- a swap partition
- NetworkManager
- an AMD or Intel CPU

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

## Identify the drive

List available block devices:

```sh
lsblk
```

The examples in this guide use:

```text
/dev/sdX
```

Replace this with the correct drive for your system.

For example, an NVMe drive may appear as:

```text
/dev/nvme0n1
```

Its partitions would then use names such as:

```text
/dev/nvme0n1p1
/dev/nvme0n1p2
/dev/nvme0n1p3
```

## Partition the drive

Open the target drive with `fdisk`:

```sh
fdisk /dev/sdX
```

Create a GPT partition table with an example layout similar to:

```text
/dev/sdX1  EFI System        1 GiB
/dev/sdX2  Linux swap        Size according to your needs
/dev/sdX3  Linux filesystem  Remaining space
```

If you intend to use hibernation, make sure the swap configuration is large enough for your system.

## Format the partitions

Format the root partition as ext4:

```sh
mkfs.ext4 /dev/sdX3
```

Initialize the swap partition:

```sh
mkswap /dev/sdX2
```

Format the EFI System Partition as FAT32:

```sh
mkfs.fat -F 32 /dev/sdX1
```

## Mount the filesystems

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

## Install the base system

For an AMD CPU:

```sh
pacstrap -K /mnt \
    amd-ucode \
    base \
    base-devel \
    efibootmgr \
    grub \
    linux \
    linux-firmware \
    networkmanager \
    neovim \
    sof-firmware \
    sudo
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

Generate `/etc/fstab` using filesystem UUIDs:

```sh
genfstab -U /mnt >> /mnt/etc/fstab
```

Review the generated file:

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

Open the locale configuration:

```sh
nvim /etc/locale.gen
```

Uncomment the UTF-8 locales you want to use.

For example:

```text
en_US.UTF-8 UTF-8
```

Generate the locales:

```sh
locale-gen
```

Create the locale configuration:

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

For a US keyboard:

```text
KEYMAP=us
```

Use the appropriate console keymap if you use a different layout.

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

Open the sudoers file with `visudo`:

```sh
EDITOR=nvim visudo
```

Uncomment:

```text
%wheel ALL=(ALL:ALL) ALL
```

This allows members of the `wheel` group to use `sudo`.

## Enable networking

Enable NetworkManager at boot:

```sh
systemctl enable NetworkManager
```

## Install GRUB

This guide mounts the EFI System Partition at:

```text
/boot/efi
```

Install GRUB for a 64-bit UEFI system:

```sh
grub-install \
    --target=x86_64-efi \
    --efi-directory=/boot/efi \
    --bootloader-id=GRUB
```

Generate the GRUB configuration:

```sh
grub-mkconfig -o /boot/grub/grub.cfg
```

## Finish the installation

Exit the chroot:

```sh
exit
```

Unmount the installed system:

```sh
umount -R /mnt
```

Disable swap:

```sh
swapoff -a
```

Flush pending filesystem writes:

```sh
sync
```

Reboot:

```sh
reboot
```

Remove the Arch Linux installation media when the machine begins rebooting.

## Post-installation

After booting into the new system, log in using the user account created during installation.

This guide intentionally stops at the base operating system installation. Desktop environments, window managers and other post-installation configuration are kept separate.

## References

This guide was written independently using the official ArchWiki as a technical reference.

Relevant ArchWiki documentation:

- Installation guide
- GRUB
- NetworkManager
- Microcode

Arch Linux is a rolling-release distribution. Check the current ArchWiki before installing in case the official installation procedure has changed.

## License

Made by rabbi-lion.

Original text in this repository is licensed under the Creative Commons Attribution-ShareAlike 4.0 International License.

Referenced projects and documentation retain their respective licenses.

See `LICENSE` for the full license text.
