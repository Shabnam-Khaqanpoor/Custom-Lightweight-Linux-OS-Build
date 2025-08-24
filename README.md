Custom Lightweight Linux OS Build
This repository documents the process of building a custom, lightweight Linux operating system using Buildroot. This project was completed as part of the Operating Systems Laboratory course.
Project Overview
The primary goal of this project was to create a minimal, bootable Linux distribution from scratch. This involved setting up a build environment, configuring the Linux kernel and root filesystem, compiling the sources, and deploying the resulting image to a bootable USB drive.
Environment Setup & Prerequisites
The development environment was set up on an Ubuntu 64-bit system running on a VMware Workstation.
1. System Update
First, the system's package list was updated and all installed packages were upgraded to their latest versions.
Bash
sudo apt update && sudo apt upgrade
2. Installing Dependencies
Next, essential packages and build tools required by Buildroot were installed.
Bash
sudo apt install build-essential binutils rsync wget libncurses-dev libelf-dev libssl-dev
Building the Linux System with Buildroot
The core of this project involved using Buildroot to configure and build the custom Linux image.
1. Download and Extract Buildroot
Buildroot version 2025.02.3 was downloaded and extracted.
Bash
wget https://buildroot.org/downloads/buildroot-2025.02.3.tar.gz  
tar -xf buildroot-2025.02.3.tar.gz  
cd buildroot-2025.02.3
2. Configuration
The Buildroot configuration interface was launched using make menuconfig. The following key parameters were configured according to the project requirements:
⦁	Target options: Set to x86_64 architecture.
⦁	System configuration: The system hostname was set to oslab-shabnam, and a root password ("shabnam") was configured.
⦁	Target packages: dropbear (an SSH server) was selected.
⦁	Filesystem images: Configured to generate an ext4 root filesystem with a specific size of 120M.
⦁	Bootloader: grub2 was selected with support for x86-64-efi.  
These configurations were saved to the .config file.
3. Compilation
The build process was initiated with the make command. This step cross-compiles the toolchain, kernel, bootloader, and all selected userspace packages. Upon completion, the final images were located in the output/images directory. The total size of the generated OS was 30MB.
Deployment to USB Drive
After a successful build, the generated images were written to a USB drive to create a bootable device.
1. Partitioning the USB Drive
The target USB drive (identified as
/dev/sdd) was partitioned with two primary partitions :
⦁	A boot partition (FAT32) for the bootloader and kernel.
⦁	A root partition (ext4) for the root filesystem.
2. Formatting and Mounting
The newly created partitions were formatted and mounted.
Bash
# Format the partitions  
sudo mkfs.vfat -F32 /dev/sdd1  
sudo mkfs.ext4 /dev/sdd2
# Mount the partitions  
sudo mount /dev/sdd1 /mnt/efi  
sudo mount /dev/sdd2 /mnt/root
3. Copying Files
The compiled artifacts were copied to the appropriate partitions on the USB drive.
Bash
# Copy boot files  
sudo cp output/images/bzImage /mnt/efi/boot/  
sudo cp -r output/images/efi-part/* /mnt/efi/
# Copy root filesystem  
sudo cp -a output/images/rootfs.ext4 /mnt/root/
4. GRUB Configuration
The GRUB configuration file (grub.cfg) was modified to point to the correct root partition (/dev/sda2 instead of /dev/sda1).
Booting the Custom OS
Finally, the target laptop's BIOS settings were adjusted to boot from the USB drive.
Secure Boot was disabled to allow the custom-signed OS to load.
Upon booting, the system successfully loaded the custom Linux OS. I was able to log in as
root with the pre-configured password "shabnam".
Challenges Faced
The process was not without its difficulties:
⦁	Insufficient Memory: The initial build failed due to insufficient memory allocated to the virtual machine. This was resolved by increasing the VM's disk space to 40GB.
⦁	USB Detection Issues: The virtual machine repeatedly failed to recognize the USB drive. This required several attempts and troubleshooting sessions to resolve.
