# Custom Lightweight Linux OS Build

This repository documents the process of building a custom, lightweight Linux operating system using **Buildroot**.  
This project was completed as part of the **Operating Systems Laboratory** course.

---

## Project Overview
The primary goal of this project was to create a minimal, bootable Linux distribution from scratch.  
This involved:
- Setting up a build environment
- Configuring the Linux kernel and root filesystem
- Compiling the sources
- Deploying the resulting image to a bootable USB drive

---

## Environment Setup & Prerequisites

The development environment was set up on an **Ubuntu 64-bit** system running on **VMware Workstation**.

### 1. System Update
First, the system's package list was updated, and all installed packages were upgraded to their latest versions.

```bash
sudo apt update && sudo apt upgrade
```

### 2. Installing Dependencies
Next, essential packages and build tools required by Buildroot were installed.

```bash
sudo apt install build-essential binutils rsync wget libncurses-dev libelf-dev libssl-dev
```

---

## Building the Linux System with Buildroot

The core of this project involved using Buildroot to configure and build the custom Linux image.

### 1. Download and Extract Buildroot
Buildroot version **2025.02.3** was downloaded and extracted.

```bash
wget https://buildroot.org/downloads/buildroot-2025.02.3.tar.gz  
tar -xf buildroot-2025.02.3.tar.gz  
cd buildroot-2025.02.3
```

### 2. Configuration
The Buildroot configuration interface was launched using:

```bash
make menuconfig
```

The following key parameters were configured according to the project requirements:

- **Target options:** Set to `x86_64` architecture  
- **System configuration:** Hostname set to `oslab-shabnam`, root password set to `"shabnam"`  
- **Target packages:** `dropbear` (an SSH server) selected  
- **Filesystem images:** Generate an `ext4` root filesystem with a size of **120M**  
- **Bootloader:** `grub2` with support for `x86-64-efi`  

These configurations were saved to the `.config` file.

### 3. Compilation
The build process was initiated with:

```bash
make
```

This step cross-compiles the toolchain, kernel, bootloader, and all selected userspace packages.  
Upon completion, the final images were located in the `output/images` directory.  
The total size of the generated OS was **30MB**.

---

## Deployment to USB Drive

After a successful build, the generated images were written to a USB drive to create a bootable device.

### 1. Partitioning the USB Drive
The target USB drive (identified as `/dev/sdd`) was partitioned with two primary partitions:
- **Boot partition** (FAT32) for the bootloader and kernel
- **Root partition** (ext4) for the root filesystem

### 2. Formatting and Mounting
The newly created partitions were formatted and mounted:

```bash
# Format the partitions  
sudo mkfs.vfat -F32 /dev/sdd1  
sudo mkfs.ext4 /dev/sdd2

# Mount the partitions  
sudo mount /dev/sdd1 /mnt/efi  
sudo mount /dev/sdd2 /mnt/root
```

### 3. Copying Files
The compiled artifacts were copied to the appropriate partitions on the USB drive:

```bash
# Copy boot files  
sudo cp output/images/bzImage /mnt/efi/boot/  
sudo cp -r output/images/efi-part/* /mnt/efi/

# Copy root filesystem  
sudo cp -a output/images/rootfs.ext4 /mnt/root/
```

### 4. GRUB Configuration
The **GRUB** configuration file (`grub.cfg`) was modified to point to the correct root partition (`/dev/sda2` instead of `/dev/sda1`).

---

## Booting the Custom OS

Finally, the target laptop's BIOS settings were adjusted to boot from the USB drive.  

- **Secure Boot** was disabled to allow the custom-signed OS to load  
- Upon booting, the system successfully loaded the custom Linux OS  

Login credentials:
- **Username:** root  
- **Password:** shabnam  

---

## Challenges Faced

The process was not without its difficulties:

- **Insufficient Memory**  
  The initial build failed due to insufficient memory allocated to the virtual machine.  
  **Solution:** Increased VM's disk space to 40GB.

- **USB Detection Issues**  
  The virtual machine repeatedly failed to recognize the USB drive.  
  **Solution:** Multiple troubleshooting attempts until the issue was resolved.

---

