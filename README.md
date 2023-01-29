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
1. Verify the boot mode: 
    - Command: ls /sys/firmware/efi/efivars
1. Test internet connection: 
    - Command: ping archlinux.org
1. Update the system clock
    - Command: timedatectl set-ntp true
    
## Partitioning and Formatting and Mounting the Drive
1. Find the drive name:
    - Command: fdisk -l
### Partitioning
1. Begin partitioning the drive (Name for my drive: sda)
    - Command: fdisk /dev/*the_disk_to_be_partitioned* (My case: fdisk /dev/sda)
1. Create sda1 partition: (will be the EFI partition)
    - Size: +500M
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
    - I installed:<br>  {man-db, man-pages, texinfo} for documentation<br>
                    {sof-firmware} for sound cards<br>
                    {iproute2, iputils} for networking<br>
                    {vim} for text editing
    - Command: pacstrap /mnt *package_name*

### Configuring the System 
1. Generate the *fstab* file so that when the system boots it knows where to mount the partitions:
    - Command: genfstab -U /mnt >> /mnt/etc/fstab
1. Change root into the new system:
    - Command: arch-chroot /mnt
#### Time zone
1. Set the time zone (running *timedatectl list-timezones* I found our timezone to be America/Chicago):
    - Command: ln -sf /usr/share/zoneinfo/*Region*/*City* /etc/localtime (My case: ln -sf /usr/share/zoneinfo/America/Chicago /etc/localtime)
1. Set the Hardware Clock from the System Clock, and update the timestamps in /etc/adjtime:
    - Command: hwclock --systohc
#### Localization
1. Edit */etc/locale.gen* and uncomment the locales you want:
    - To use the US locale I uncommented *en_US.UTF-8 UTF-8*
1. Generate the locale's you uncommented:
    - Command: locale-gen
1. Create and edit the */etc/locale.conf* file with your chosen locale:
    - Write in the file: LANG=*chosen_local* (My case: LANG=en_US.UTF-8)
1. Create and edit the */etc/vconsole.conf* file with your chosen keyboard layout:
    - Write in the file: KEYMAP=*chosen_keyboard_layout* (My case:KEYMAP=us)
#### Network configuration
1. Create and edit /etc/hostname file to input your chosen hostname (I chose: LCARS-2112-9A)
1. Configure the /etc/hosts file as follows:

      127.0.0.1        localhost<br>
      ::1              localhost<br>
      127.0.1.1        *myhostname*

    - For me this was:
    
      127.0.0.1        localhost<br>
      ::1              localhost<br>
      127.0.1.1        LCARS-2112-9A

1. Get *NetworkManager* working:
	- Install dependencies and tools:<br>
	- Command: pacman -S wpa_supplicant wireless_tools networkmanager iw dhcpcd netctl dialog ppp<br>
	- Enable Network Manager:<br>
	- Command: systemctl enable NetworkManager
### Root password
1. Set the root password:
    - Command: passwd
### Bootloader
1. Choose a bootloader and install it: (I chose GRUB as by bootloader)
    - Command: pacman -S grub os-prober efibootmgr
    - Command: grub-install --target=x86_64-efi --efi-directory=boot --bootloader-id=GRUB
1. Install Microcode for processor updates:
    - Command: pacman -S amd-ucode (since I have an AMD processor)
1. Set GRUB to detect Microcode updates:
    - Command: grub-mkconfig -o /boot/grub/grub.cfg
### Reboot the system
1. Reboot
    - Command: exit (from chroot)
    - Command: reboot
1. Setup WiFi:
    - Command: systemctl start NetworkManager
    - Command: nmcli
3. Now the basic installation is complete!!:)

## Create user accounts 
1. Install *sudo*:
	- Command: pacman -S sudo
1. Create my user account: (with the home directory)
	- Command: useradd -m dfant
	- Command: passwd dfant
1. Create my sal account: (with the home directory)
	- Command: useradd -m sal
	- Command: passwd -e sal (GraceHopper1906)
1. Create my codi account: (with the home directory)
	- Command: useradd -m codi
	- Command: passwd -e codi (GraceHopper1906)
1. Edit /etc/sudoers to give all three users sudo privaleges:
	- Command: EDITOR=vim visudo (to launch visudo with vim, for some reason I can't get visudo to work otherwise...)
	- Under 'User privilege specification' add the lines: *dfant ALL=(ALL) ALL*, *codi ALL=(ALL) ALL*, *sal ALL=(ALL) ALL*

## Customization
### Desktop environment
Note-to-self: In the future I may want to build a custom DE from the Openbox WM instead
For this installation, I chose to install LXDE (https://wiki.archlinux.org/title/LXDE & https://wiki.lxde.org/en/Arch_Linux)
1. Install LXDE: (I wanted to see all the packages included in LXDE so I looked at this list https://archlinux.org/groups/x86_64/lxde/)<br>
	- Command: pacman -S lxde-common lxdm lxsession openbox (bare bones install of LXDE)<br>
	- Command: pacman -S lxde (FULL install of LXDE)<br>
	- Command: systemctl enable lxdm.service (to launch lxdm on startup)
1. Launch the DE:
	- Command: lxdm
1. Customized the look of the DE until you are satisfied (from the "Customize Look and Feel" menu)
1. Set resolution as 1280x768:
	- Command: xrandr --output Virtual-1 --mode 1280x768
1. Create folder: /home/dfant/Programs/
1. Add programs I want:<br>
	- git<br>
	- base-devel<br>
	- vivaldi (browser) <br>
	- texlive-science (LaTex)<br>
	- yay (for AUR)<br>
	- gtk-theme-numix-sx<br>
	- arc-icon-theme-full-git<br>
	- leafpad<br>
	- net-tools<br>
1. Note: using AUR: (without yay)<br>
	- Find a package I want<br>
	- Download using: 'git clone *Package_URL*' inside ~/Programs/<br>
	- cd *package_folder*<br>
	- makepkg -si PKGBUILD

### Install different shells and ssh
1. I installed zsh and fish for the dfant user
	- Command: sudo pacman -S zsh (then I went through the install proccess)
	- Command: sudo pacman -S fish
1. To install ssh, run the command:
	- Command: pacman -S openssh

### Adding colors to the bash (for dfant)
1. I followed This guide for doing adding the colors: https://averagelinuxuser.com/linux-terminal-color/ 
	- I also used: https://wiki.archlinux.org/title/Bash/Prompt_customization as a reference
1. Make a copy of .bashrc for safe editing:
	- Command: cp .bashrc .bashrc.backup
1. Then I downloaded his files for bash.bashrc, .bashrc and DIR_COLORS
1. I then copied the contents of bash.bashrc into .bashrc
1. I then moved .bashrc into /home/dfant/ (overwriting the old version)
1. To add color to pacman, I first backed up '/etc/pacman.conf'
	- Command: cp /etc/pacman.conf /etc/pacman.conf.backup
	- And uncommented the word 'Color'

### Setting up aliases
1. Create a file called .bash_aliases inside ~ folder
1. Append the following to ~/.bashrc:<br>
	if [ -f ~/.bash_aliases ]; then<br>
	. ~/.bash_aliases<br>
	fi

1. Run the Command: source  ~/.bash_aliases

#### List of alias' I made: (list these inside ~/.bash_aliases)
1. alias vi=vim
1. alias svi='sudo vi'
1. alias reboot='sudo /sbin/reboot'
1. alias poweroff='sudo /sbin/poweroff'
1. alias halt='sudo /sbin/halt'
1. alias shutdown='sudo /sbin/shutdown'
1. alias meminfo='free -mlt'
1. alias cpuinfo='lscpu'
1. alias c='clear'
1. alias h='history'
1. alias j='jobs -l'
1. alias visudo='sudo EDITOR=vim visudo'
