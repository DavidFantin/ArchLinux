# Arch Linux Installation Document
## Pre-installation
1. Aquired the installation iso from: https://archlinux.org/download/ (v. 2021.10.01-x86_64)

## Creating the VM on VMware
1. Select the .iso file
1. Choose version: "Other Linux 5.x kernel 64-bit"
1. Choose an install location (mine is ~\Documents\Virtual Machines\ArchLinux)
1. Set the maximum disk size as 20GB
1. Set the maximum RAM as 2048MB instead of the default 768MB
1. In the Advanced Settings: set the firmware type as UEFI
1. Launch the VM

## Pre-Partition Settings
1. Setting Keyboard Layout: 
    - Command: loadkeys us
    - Result: Keyboard layout changed to the US format
1. Verify the boot mode: 
    - Command: ls /sys/firmware/efi/efivars
    - Result: Should give a list of files
1. Test internet connection: 
    - Command: ping archlinux.org
    - Result: Should recive data from the website
1. Update the system clock
    - Command: timedatectl set-ntp true
    - Result: Makes the system clock accurate
    
## Partitioning and Setting up the Drive
1. Find the drive name:
    - Command: fdisk -l
### Partitioning
1. Begin partitioning the drive (Name for my drive: sda)
    - Command: fdisk /dev/*the_disk_to_be_partitioned* (My case: fdisk /dev/sda)
1. Create sda1 partition: (will be the EFI partition)
    - Size: 500MB
1. Create sda2 partition:
    - Size: remainder of the drive
### Formatting
1. Format sda2 to be the root file system: (I used the ext4 file system)
    - Command: mkfs.ext4 /dev/sda2
1. Format sda1 with the EFI file system:
    - Command: mkfs.fat -F32 /dev/sda1
### Mounting
1. Mount the root partition (sda2) to /mnt:
    - Command: mount /dev/sda2 /mnt
1. Make the /mnt/boot directory:
    - Command: mkdir /mnt/boot
1. Mount the EFI partition (sda1) to /mnt/boot: (Note: Should I have mounted it to /mnt/boot/efi or /mnt/efi instead?)
    - Command: mount /dev/sda1 /mnt/boot

## Installation
1. Update the mirror servers list:
    - Command: reflector (python script that lists the mirror servers)
1. Read mirror list: (Optional, the top server has the highest priority)
    - Command: cat /etc/pacman.d/mirrorlist
1. Install the *base* package (This package includes the Linux kernel and firmware for common hardware)
    - Command: pacstrap /mnt base linux linux-firmware
1. Install other packages you want: 
    - I installed:  {man-db, man-pages, texinfo} for documentation
                    {sof-firmare} for sound cards
                    {iproute2, iputils} for networking
                    {vim, nano, leaf} for text editing
    - Command: pacstrap /mnt *package_name*

## Configuring the System 
1. Generate the *fstab* file so that when the system boots it knows where to mount the partitions:
    - Command: genfstab -U /mnt >> /mnt/etc/fstab
1. Change root into the new system:
    - Command: arch-chroot /mnt
### Time zone
1. Set the time zone (running *timedatectl list-timezones* I found our timezone to be America/Chicago):
    - Command: ln -sf /usr/share/zoneinfo/*Region*/*City* /etc/localtime (My case: ln -sf /usr/share/zoneinfo/America/Chicago /etc/localtime)
1. Set the Hardware Clock from the System Clock, and update the timestamps in /etc/adjtime:
    - Command: hwclock --systohc
### Localization
1. Edit */etc/locale.gen* and uncomment the locales you want:
    - To use the US locale I uncommented *en_US.UTF-8 UTF-8*
1. Generate the locale's you uncommented:
    - Command: locale-gen
1. Create the *locale.conf* file:
    - Command: touch /etc/locale.conf
1. Edit the *locale.conf* file with your chosen locale:
    - Write in the file: LANG=*chosen_local* (My case: LANG=en_US.UTF-8)
1. Create the *vconsole.conf* file:
    - Command: touch /etc/vconsole.conf
1. Edit the *vconsole.conf* file with your chosen keyboard layout:
    - Write in the file: KEYMAP=*chosen_keyboard_layout* (My case:KEYMAP=us)
### Network configuration
1. Create and edit /etc/hostname file
1. Edit the file and input your chosen hostname (I chose: LCARS-2112-9A)
1. Configure the /etc/hosts file as follows:

      127.0.0.1        localhost
      ::1              localhost
      127.0.1.1        *myhostname*

    - For me this was:
    
      127.0.0.1        localhost
      ::1              localhost
      127.0.1.1        LCARS-2112-9A
      
### Root password
1. Set the root password:
    - Command: passwd
### Bootloader
1. Choose a bootloader and install it: (I chose GRUB as by bootloader)
    - Command: pacman -S grub os-prober
    - Command: pacman -S efibootmgr
    - Command: grub-install --target=x86_64-efi --efi-directory=boot --bootloader-id=GRUB
1. Install Microcode for processor updates:
    - Command: pacman -S amd-ucode (since I have an AMD processor)
1. Set GRUB to detect Microcode updates:
    - Command: grub-mkconfig -o /boot/grub/grub.cfg
### Reboot the system
    - Command: exit (from chroot)
    - Command: reboot
1. Now the basic installation is complete!!:)
1. 
