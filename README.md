# gavinn0715-ui.github.io

## Description

This is the basic documentation for my Arch Installation project in System Administation. This is not a full guide to installing Arch, but a brief overview of what I did to get Arch to run on my system. 

## Documentation

1. Get .iso from https://release.archboot.com/aarch64/latest/iso/ because I am on a Mac 
2. Create a New Virtual Machine in VMware using the iso
3. Select "Other Linux 6.x kernel 64-bit Arm" on Choose Operating System window
4. Customised settings
	1. 5 Cores
	2. 4096 RAM
	3. 100 GB Disk size
	4. Accelerate 3D Graphics
	5. Use full resolution for Retina display
5. Give up and try qemu/UTM instead of vmware due to black screen issues
	1. https://www.reddit.com/r/archlinux/comments/1ex1pit/archlinux_black_screen_after_install_inside/
	2. Happened to have homebrew, so installation was fast to test
6. Make VM in UTM
	1. Virtualise, not emulate
7. Control-C to enter into bash terminal once VM loads in
8. Begin partitioning the disk
	1. ```fdisk /dev/vda```
	2. GPT
	3. Boot partition 
		1. Size 1G
		2. Type EFI System
	4. Root partition
		1. Size 99G
		2. Type Linux filesystem (default.
	5. Write, confirm, and quit
9. Format partitions
	1. ```mkfs.fat -F 32 /dev/vda1```
	2. ```mdfk.ext4 /dev/vda2```
10. Mount file systems
	1. ```mount /dev/vda2 /mnt```
	2. ```mount --mkdir /dev/vda1 /mnt/boot```
11. Install packages
	1. ```pacstrap -K /mnt base linux linux-firmware```
12. Fstab
	1. ```genfstab -U /mnt >> /mnt/etc/fstab```
	2. Determines how disk partitions should be mounted 
13. Chroot
	1. ```arch-chroot -S /mnt```
14. Localization
	1. ```ln -sf /usr/share/zoneinfo/America/Chicago /etc/localtime```
	2. ```hwclock --systohc```
15. Root Password
	1. ```passwd```
	2. Enter password for root
16. Set up DHCP upon prompt
17. Asks for mirror, just went for the first one
18. Chose Gnome and Chromium upon prompt
19. Make 2 new user accounts with sudo perms
	1. ```useradd -m -G wheel -s /bin/bash gavin```
	2. ```passwd gavin```
	3. Repeated for codi
20. Realise I should have probably gotten a bootloader
	1. Go back to installer
	2. Mount partitions again
	3. ```pacman -S grub efibootmgr```
	4. ```grub-install --target=arm64-efi --efi-directory=/boot --bootloader-id=GRUB --removable```
		1. Note: target is arm64-efi because I am using an arm based version
	5. ```grub-mkconfig -o /boot/grub/grub.cfg```
	6. exit and unmount
	7. Restart
		1. Clear the iso as it reboots
	8. Login as root using previous password
	9. Redo the user accounts
21. forgot to add sudo
	1. ```pacman -S sudo```
	2.  ```usermod -aG wheel gavin```
		1. Repeat for codi
	3. ```EDITOR=nano visudo```
		1. uncomment ```%wheel ALL=(ALL:ALL. ALL```
22. Install fish
	1. ```sudo pacman -S zsh```
23. Install firefox
	1. ```sudo pacman -S chromium```
24. Go to https://robotmoon.com/bash-prompt-generator/ and customize terminal colors
	1. ```sudo nano ~/.bashrc```
	2. Replace the export PS1... with the output from the website
25. Install and enable SSH 
	1. ```sudo pacman -Syu openssh```
	2. ```sudo systemctl enable sshd```
26. Make GRUB go straight to GUI instead of boot menu
	1. ```sudo nano /etc/default/grub```
	2. Change GRUB_TIMEOUT to 0
	3. Change GRUB_TIMEOUT_STYLE to hidden
	4. ```sudo grub-mkconfig -o /boot/grub/grub.cfg```
		1. Reloads config
27. Make aliases
	1. ```alias c ='clear'```
	2.  ```alias ping ='ping -c 5'```
	3.  ```alias``` to check which aliases are active
