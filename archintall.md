# Installing ArchLinux on a VM

## Step 1. Download the ISO file

I got my ISO file from the arizona.edu link

## Step 2. Setup the VM

Gave 4GB of RAM and 25 GB of storage. Click UEFI in advanced options of VM. 

## Step 3. Check Internet Connectivity

VMware should already have VM connected to internet. Verify connection by pinging. 

## Step 4. Update System Clock

```bash
timedatectl set-timezone America/Chicago
```

## Step 5. Partition the disks

```bash
fdisk -l
fdisk /dev/sda
```
g - creates GPT partition table

### Creates boot partition

n - adds new partition
    Partition Number: 1
    First Sector: Enter
    Last Sector: +1G
t - change a partition type
    Choose 1(EFI System)

### Creates swap partition

n - adds new partition
    Partition Number: 2
    First Sector: Enter
    Last Sector: +2G
t - change a partition type
    Choose 2
    Select 19(Linux Swap)

### Creates root partition

n - adds new partition
    Partition Number: 3
    First Sector: Enter
    Last Sector: Enter
w - write table to disk then exit

## Step 6. Format Partitions

### EXt4 File system
```bash
mkfs.ext4 /dev/sda3
```
### Initialize swap partition
```bash
mkswap /dev/sda2
```
### Format EFI system to FAT32
```bash
mkfs.fat -F 32 /dev/sda1
```

## Step 7. Mount File System

### Mount root to /mnt
```bash
mount /dev/sda3 /mnt
```

### Mount EFI system partition
```bash
mount --mkdir /dev/sda1 /mnt/boot
```

### Enable Swap partition
```bash
swapon /dev/sda2
```

## Step 8. Select Mirrors 

Mirrors are located in /etc/pacman.d/mirrorlist

## Step 9. Install essential packages

```bash
pacstrap -K /mnt base linux sudo nano man-db man-pages networkmanager
```

## Step 10. Fstab
```bash
genfstab -U /mnt >> /mnt/etc/fstab
```
## Step 11. Chroot
```bash
arch-chroot -S /mnt
```

## Step 12. Time 
```bash
ln -sf /usr/share/zoneinfo/America/Chicago /etc/localtime
hwclock --systohc
```

## Step 13. Localize

Uncomment the UTF-8 locales you will use in /etc/locale.gen. I am using en_US.UTF-8 UTF-8. 
Then generate locales,
```bash
locale-gen
```
Create locale.conf and set LANG variable
```bash
nano /etc/locale.conf
LANG=en_US.UTF-8
```

## Step 14. Network Config
Create your hostname file
```bash
echo "archlinux" > /etc/hostname
```

## Step 15. Root password
Set a password for the root user
```bash
passwd
```

## Step 16. Boot Loader
Choose a boot loader. I am using GRUB
First install packages grub and efibootmgr
```bash
pacman -S grub efibootmgr
```
Then install GRUB EFI application
```bash
grub-install --target=x86_64-efi --efi-directory=/boot --bootloader-id=GRUB
grub-mkconfig -o /boot/grub/grub.cfg
```

## Step 17. Reboot
Exit the chroot then run the following commands
```bash
umount -R /mnt
reboot
```
Then remove the Arch ISO from the VM

## Step 18. Add Users
Adding a User and giving sudo permissions
```bash
useradd -m -G wheel -s /bin/bash trinder
useradd -m -G wheel -s /bin/bash codi
```
Then set passwords
```bash
passwd trinder
passwd codi
```
Next you enable sudo perms
```bash
EDITOR=nano visudo
```
Uncomment the line # %wheel ALL=(ALL:ALL) ALL

## Step 19. Installing another shell
Network was unreachable so I ran
```bash
sudo ip link set ens33 up
sudo systemctl enable --now NetworkManager
```
Then I installed zsh
```bash
sudo pacman -S zsh
```
Then change the shell of your users
```bash
chsh -s /usr/bin/zsh trinder
chsh -s /usr/bin/zsh codi
``` 
verify changes
```bash
echo $SHELL
```

## Step 20. Install Desktop Environment
```zsh
sudo pacman -S gnome
sudo systemctl enable gdm.service
```

## Step 21. Install SSH
```zsh
sudo pacman -S openssh
```
Enable SSH
```zsh
sudo systemctl enable --now sshd
```

## Step 22. Customizing terminal
The first thing I did was set colors and changed the prompt colors
```zsh
sudo nano ~/.zshrc
BLUE='%F{blue}'
GREEN='%F{green}'
RESET='%f'
PROMPT="${GREEN}%n@%m ${BLUE}%~ ${RESET}$ "
```
Install zsh plugin
```zsh
sudo pacman -S zsh-syntaz-highlighting
```
Add to .zshrc
```zsh
source /usr/share/zsh/plugins/zsh-syntax-highlighting/zsh-syntax-highlighting.zsh
```

## Step 23. Add aliases
Next I added aliases to my .zshrc file some were
```zsh
alias c='clear'
alias ll='ls -lah'
alias ping4='ping -c 4'
```
