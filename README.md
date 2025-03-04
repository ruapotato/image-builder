# Maemo Leste Image Builder

A build system for creating bootable images and tarballs of Maemo Leste for various devices.

## Overview

The image-builder is based on the arm-sdk framework and allows you to build Maemo Leste images for:
- Physical devices (like PinePhone, Droid 4, N900, etc.)
- Virtual machines (QEMU, VirtualBox)

It supports building on multiple base distributions including:
- Debian (with systemd)
- Devuan (original)

## Prerequisites

Before you begin, make sure you have the following dependencies installed:

```bash
sudo apt install \
    zsh qemu-user-static binfmt-support \
    debootstrap parted kpartx rsync dosfstools \
    btrfs-progs extlinux f2fs-tools u-boot-tools \
    wget python3 sudo git curl gpg
```

## Quick Start

1. Clone the repository and enter the directory:

```bash
git clone https://github.com/maemo-leste/image-builder.git
cd image-builder
```

2. Enter the vm-sdk directory:

```bash
cd vm-sdk
```

3. Use zsh:

```bash
zsh
```

4. Set VM build parameters (for VM builds):

```bash
export vmsdk_version=1
export size=8000
export imageformat=qcow2
export parted_type=dos
export dos_boot="2048s 264191s" 
export dos_root="264192s 100%"
export rootfs="ext4"
export bootfs="ext2"
export arch=amd64
```

5. Build the image:

```bash
source ./sdk
load debian bookworm
build_vm_dist
```
```
xz -d /home/david/image-builder/vm-sdk/dist/maemo-leste-1.0-amd64-20250303.qcow2.xz
qemu-system-x86_64 -enable-kvm -m 2G -smp 2 -drive file=/home/david/image-builder/vm-sdk/dist/maemo-leste-1.0-amd64-20250303.qcow2,format=qcow2 -display gtk
```

## Building for Different Targets

### Building VM Images

```bash
zsh
source ./sdk
load debian arm64-generic bookworm
export vmsdk_version=1
export size=8000
export imageformat=qcow2
export parted_type=dos
export dos_boot="2048s 264191s" 
export dos_root="264192s 100%"
export rootfs="ext4"
export bootfs="ext2"
build_vm_dist
```

### Building for Physical Devices

```bash
zsh
source ./sdk
load debian DEVICE_NAME bookworm
build_arm_dist
```

Replace `DEVICE_NAME` with one of the supported devices:
- `beagleboneblack` - BeagleBone Black
- `n900` - Nokia N900
- `droid4` - Motorola Droid 4
- `pinephone` - Pine64 PinePhone
- And many others (see `boards/` directory)

## Configuration

### Project Structure

- `arm-sdk/` - The main build framework
- `arm-sdk/boards/` - Device-specific configurations
- `arm-sdk/lib/` - Libraries for different distributions
- `rootfs-overlay/` - Files to be copied into the root filesystem

### Blend Files

Blend files (like `bookworm.blend`) define distribution-specific configurations and customizations.

### Config Files

Config files (like `bookworm.config`) define variables used by the build process:
- Package lists
- Device-specific packages
- Release names and versions

## Troubleshooting

### Cleaning Up Previous Builds

If you encounter mount or file system related errors:

```bash
sudo umount -lf /path/to/image-builder/arm-sdk/tmp/debian-arm64-build/bootstrap/dev/pts 2>/dev/null || true
sudo umount -lf /path/to/image-builder/arm-sdk/tmp/debian-arm64-build/bootstrap/dev 2>/dev/null || true
sudo umount -lf /path/to/image-builder/arm-sdk/tmp/debian-arm64-build/bootstrap/proc 2>/dev/null || true
sudo umount -lf /path/to/image-builder/arm-sdk/tmp/debian-arm64-build/bootstrap/sys 2>/dev/null || true
clean_strapdir
rm -f /home/david/image-builder/vm-sdk/tmp/*
```

### Repository Issues

If you encounter package dependency issues, check:
1. The repositories in your blend file's `conf_print_sourceslist` function
2. Package pinning settings in your blend file's `blend_preinst` function

## Advanced: Building Debian-based Images

The image-builder originally used Devuan as its base. To build using Debian with systemd:

1. Ensure you have the proper repository configuration in your blend file
2. Handle potential conflicts between Maemo packages and systemd
3. Configure package pinning for repository preferences

For example, when using Debian Bookworm with Maemo Leste's "daedalus" repository:

```bash
zsh
source ./sdk
load debian arm64-generic bookworm
export vmsdk_version=1 size=8000 imageformat=qcow2 parted_type=dos
export dos_boot="2048s 264191s" dos_root="264192s 100%" rootfs="ext4" bootfs="ext2"
build_vm_dist
```

## Output Files

Built images can be found in:
- VM images: `arm-sdk/tmp/debian-arm64-build/*.qcow2` or `*.vdi`
- ARM device images: `arm-sdk/tmp/debian-arm64-build/*.img`

## License

See the LICENSE file for copyright and license details.
