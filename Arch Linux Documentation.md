# Arch Linux Documentation
This is the documentation for my Arch Linux Install
## Installation and Setup
- First I downloaded and installed VMWare Workstation Pro 
- Downloaded an Arch  Linux installation image from the mit.edu mirror site: archlinux-2022.10.01-x86_64.iso
- Created new virtual machine from the iso above, with the following features/specifications:
| **Feature/Specifications** | **My Settings**    |
|----------------------------|--------------------|
| Hardware                   | Workstation 16.2.x |
| Guest Operating System     | Linux              |
| Linux Version:             | Other Linux 5.x kernel 64-bit       |
| Processors                 | 2                  |
| Memory                     | 2048MB             |
| Network Connection         | NAT                |
| SCSI Controller            | LSI Logic          |
| Virtual Disk Type          | SCSI               |
| Maximum Disk Size          | 20GB               |
- Boot the system in UEFI mode
    - Opened the .vmx file in iso folder, and added: firmware="efi" as the second line of the file to boot in UEFI
## Connect to Internet
- Ensure network interface is listed using `ip link`
- `ping google.com` to verify connection
## Set Date/time
 - Use `timedatectl` to view current system clock
 - Use `timedatectl list-timezones` to view list of timezones and find applicable timezone: America/Chicago 
- `timedatectl set-timezone America/Chicago` to update system clock to correct date/time settings
 ## Partition Disks
 - Use `fdisk -l` to identify disks/block devices
- `fdisk /dev/sda` to access block device and open *fdisk* dialogue for commands

     ### *fdisk* Dialogue Commands
      Left partition table as MBR. Created an EFI system partition  and a root partition. 
    - Create new partitions with `n` command 
    - My partitions:
        - EFI system partition - /dev/sda1
            - First sector: 2048 (default)
            - Last Sector: +500M 
            - Format: FAT32 `mkfs.fat -F 32 /dev/sda1` 
        - Root Partition - /dev/sda2
            - First Sector: default value (end of first partition)
            - Last Sector: default value (end of disk space)
            - Format ext4 `mkfs.ext4 /dev/sda2`
    - `w` to write to disk and save
    - Mount file systems
        - `mount /dev/sda2 /mnt`
        - `mount /dev/sda1 /mnt/boot` 
## Install Things
- `pacstrap -K /mnt base linux linux-firmware` installs base package, Linux kernel, and firmware
- Also install nano, tree, netctl, dhcpcd, man, sudo,  and other packages needed
    - This contributed to solving *Error 2*, described below in Errors section

## Configuration
 - Generate fstab file: `genfstab -U /mnt >> /mnt/etc/fstab`  
- chroot into /mnt: `arch-chroot /mnt`
- set timezone to America/Chicago
- Create locales with `locale-gen`, edit with `nano /etc/locale.gen` and uncomment `en_us.UTF-8 UTF-8`
- `nano /etc/hostname` and type in hostname. 
- set root password with `passwd`

## Boot Loader
I chose to use GRUB as my bootloader because it seemed like it had the most features and fewest drawbacks/incompatibility issues. Do the following while chrooted into root partition (/mnt). Not doing this could be what caused *Error 1* described  below.
-Install with `pacman -S grub efibootmgr`
- Make sure EFI system partition is mounted (/mnt/boot)
- Run this to install in EFI system partition `grub-install --target=x86_64-efi --efi-directory=/boot --bootloader-id=GRUB`
- Generate main configuration file: `grub-mkconfig -o /boot/grub/grub.cfg` 
- `exit` and `reboot`

## Post Wiki Installation Guide
### Network configuration
I ran into networking issues on my first GRUB install, as described below in *Error 2*. After reverting to pre-GRUB snapshot, reinstalling, and rebooting, I used the following commands to connect to the internet. 
- `systemctl start dhcpcd.service`enables dhcpcd to start when system boots
- `pacman -Syu` update system and sync databases
- `dhcpcd` 
### Add users
How I added users, assigned them passwords, gave sudo permissions, etc.
 - `useradd -m `*username*  to add user
 - `passwd `*username* to set password for a user
- `chage -d 0 codi` to require user *codi* to change password after first login
- Sudo Permissions
    - `sudo visudo` to edit `/etc/sudoers` file, which contains info on users/groups with sudo permissions among other sudo info. This uses `vi` as the text editor by default, which I have no experience with. Instead, I ran this command: 
        - EDITOR=nano visudo
    - Scroll down to the line with `root ALL=(ALL:ALL) ALL` and copy that line below itself, replacing `root` with *username* of user to receive sudo permissions. 
### Desktop Environment
 I originally tried to use Deepin as my DE. This was *Error 3*, described below. After this error, I decided to use LXDE for the ease of installation, fast and energy-saving  performance, and small size. 
- `pacman -S lxde` 

That's it, that was all the steps to get LXDE working.

### Aliases
- When I run `ls`, it will have coloring because `alias ls='ls --color=auto'` 


### SSH into class terminal
- `pacman -S openssh`
- turn on vpn
- `ssh -p 22 sysadmin@10.10.1.103` to ssh into my terminal 

### Installing  shell
I chose to install `fish` as I read good things about it as an alternative shell and liked that it is "interactive and user-friendly" since I'm new to linux. Did the following logged in as root:
- `pacman -S fish`
- type `fish` to enter fish shell

### Web browser
I installed links because it is lightweight and I thought it would be fun to have a text-only navigator.
- `pacman -S links` 



# Errors
## Error 1
### Trying to do full install in BIOS and not having snapshots. 
### For my first arch install attempt I booted in BIOS mode. Everything went okay until I tried to reboot after installing syslinux bootloder and I was stuck on the bootloader screen. There was an automatic boot countdown but whenever it reached 0 it would start back over at 5. I'm pretty sure I either installed the bootloader into /mnt instead of /mnt/boot or I installed it as root in the iso. Not entirely sure what happened but it totally broke my install. I wound up completely starting over on a new VM, booting through EFI, and using GRUB.

## Error 2
### After installing base package, I neglected to install anything else (besides nano) before installing GRUB bootloader. After rebooting with GRUB, I had no internet connection. I set my interface to be UP but was unable to `ping` any hosts nor `pacman` any packages. I tried setting `nameserver=8.8.8.8` in a network configuration file with no success. After 3 hours trying to troubleshoot the issue, I wound up reverting to a snapshot pre-GRUB install and installed dhcpd, netctl, and iproute2. This my or may not have helped. After reinstalling GRUB and rebooting, I still had no internet connection. This time however, I was able to fix the issue by running:
- `systemctl start dhcpcd.service` 
    - This changed my ping error message from  "Destination Host unreachable " to "Network is unreachable"
- `pacman -Syu`
    - This synchronized package databases
- Ran `dhcpcd` 
### After running those, I was able to `ping google.com` and use `pacman`. I am not sure what the exact issue was but I think it had something to do with my IP configuration and/or routing table.

## Error 3
### My third error was installing Deepin as my DE. It seemed to be a nice looking DE, and the install was easy. I also didn't want to use LXDE since that was the DE mentioned on the assignment instructions and I wanted to do something different. Anyways, once I booted into Deepin, 
- I was unable to connect to the internet (once again). This is because NetworkManager is integrated in Deepin network administration and has to be enabled to use. Which I did not do and couldn't because;
- I was unable to open a terminal.
-  I also found out that Deepin has "known security vulnerabilities... reported by developers of the openSUSE distribution to the DDE [Deepin Desktop Environment] developers" who have not responded to the reports.
### All in all , Deepin was not  good choice for DE and I should've paid more attention to the "Known Issues" column on the Deepin Arch Wiki page. I ended up reverting to a pre-Deepin snapshot and installing LXDE.  
