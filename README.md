# gavinn0715-ui.github.io

## Arch Install Description

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

 
## Docker

## Documentation

1) Before making any changes, go to Proxmox and create a snapshot named "pre-docker"
2) SSH into Virtual Machine (```ssh user@ipaddress```)
	1) Input password when prompted
3) Type the following commands to install Docker Engine
	1) ```$sudo apt update && sudo apt install -y ca-certificates curl  
$ sudo install -m 0755 -d /etc/apt/keyrings  
$ sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg \  
-o /etc/apt/keyrings/docker.asc  
$ echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] \  
https://download.docker.com/linux/ubuntu $(. /etc/os-release && echo $VERSION_CODENAME) stable" \  
| sudo tee /etc/apt/sources.list.d/docker.list  
$ sudo apt update  
$ sudo apt install -y docker-ce docker-ce-cli containerd.io \  
docker-buildx-plugin docker-compose-plugin
	   ```
4) Post Install checks
	1) ```$ sudo systemctl enable --now docker  
	   $ systemctl is-enabled docker # enabled  
	   $ sudo docker run --rm hello-world  
	   $ docker compose version```
5) Choose application to install, I chose Uptime Kuma (https://github.com/louislam/uptime-kuma/wiki/%F0%9F%94%A7-How-to-Install)
	1) ```sudo mkdir -p /opt/uptime-kuma```
	2) ```cd /opt/uptime-kuma```
	3) ``````sudo nano compose.yaml```
	4) Write: 
	   ```services:
  uptime-kuma:
    image: louislam/uptime-kuma:2
    container_name: uptime-kuma
    restart: unless-stopped
    ports:
      - "3001:3001"
    volumes:
      - ./data:/app/data
```
6) Deploy using ```sudo docker compose up -d```
7) Check status using ```sudo docker ps```
8) Go to http://[ipaddress]:3001 
	1) I chose embedded MariaDB
	2) Make admin account
9) Test if it is persistant
	1) ```sudo reboot``` and SSH back in
	2) ```sudo docker ps``` (should see Uptime Kuma still up)
10) Key-only SSH (https://askubuntu.com/questions/346857/how-do-i-force-ssh-to-only-allow-users-with-a-key-to-log-in)
    1) ```ssh-keygen -t ed25519``` (in another tab not SSH'd into VM)
    2) ```ssh-copy-id username@VM-IP```
    3) Open another terminal tab and SSH into the VM again
    4) Back in the VM, ```sudo nano /etc/ssh/sshd_config```
    5) Set ```PubkeyAuthentication yes
PasswordAuthentication no
UsePAM no```
	6) ```sudo sshd -t```
	7) ```sudo systemctl restart ssh```
11) Firewall
	1) ```
sudo apt update
sudo apt install ufw
sudo ufw allow OpenSSH
sudo ufw allow 3001/tcp
sudo ufw enable
sudo ufw status verbose
```
12) Automatic security updates (https://askubuntu.com/questions/9/how-do-i-enable-automatic-updates)
    1) ```sudo apt install unattended-upgrades```
    2) ```sudo dpkg-reconfigure -plow unattended-upgrades``` 
    3) ```sudo systemctl status unattended-upgrades```
13) Make backup directory
	1) ```sudo mkdir -p /opt/backups```
	2) ```sudo nano /usr/local/bin/backup-uptime-kuma.sh```
	3)Write: ```
#!/bin/bash
set -Eeuo pipefail

APP_DIR="/opt/uptime-kuma"
BACKUP_DIR="/opt/backups"
CONTAINER="uptime-kuma"

mkdir -p "$BACKUP_DIR"

WAS_RUNNING=0

if docker inspect -f '{{.State.Running}}' "$CONTAINER" 2>/dev/null | grep -q true; then
    docker stop "$CONTAINER"
    WAS_RUNNING=1
fi

# restart the container
restart_container() {
    if [ "$WAS_RUNNING" -eq 1 ]; then
        docker start "$CONTAINER"
    fi
}

trap restart_container EXIT

# Create timestamped archive
ARCHIVE="$BACKUP_DIR/uptime-kuma-$(date +%Y%m%d-%H%M%S).tar.gz"

tar -czf "$ARCHIVE" -C "$APP_DIR" data

echo "Created backup: $ARCHIVE"

# restart application
restart_container
trap - EXIT

# 7 newest archives only
mapfile -t BACKUPS < <(
    find "$BACKUP_DIR" -maxdepth 1 -type f \
        -name 'uptime-kuma-*.tar.gz' -printf '%f\n' |
        sort -r
)

for ((i=7; i<${#BACKUPS[@]}; i++)); do
    rm -- "$BACKUP_DIR/${BACKUPS[$i]}"
done

echo "Backup complete. Retained ${#BACKUPS[@]} archives."
```
	4) ```sudo chmod +x /usr/local/bin/backup-uptime-kuma.sh```
	5) Execute backup ```sudo /usr/local/bin/backup-uptime-kuma.sh```
	6) sudo docker ps
14) Backup on Proxmox
15) Testing backups
	1) ```echo "RESTORE TEST SUCCESS" | sudo tee /opt/uptime-kuma/data/restore-test.txt```
	2) ```cat /opt/uptime-kuma/data/restore-test.txt```
	3) Backup ```sudo /usr/local/bin/backup-uptime-kuma.sh```
	4) ```ls -lt /opt/backups/```
	5) Delete test file ```sudo rm /opt/uptime-kuma/data/restore-test.txt```
	6) Verify removal ```ls /opt/uptime-kuma/data/restore-test.txt```
	7) ```sudo tar -xzf /opt/backups/[latest filename from step 15.4].tar.gz -C /opt/uptime-kuma/```
	8) ```cat /opt/uptime-kuma/data/restore-test.txt```
