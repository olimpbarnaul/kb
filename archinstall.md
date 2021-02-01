# Arch Linux Install

yes | pacman -S terminus-font && setfont ter-v32b

#### To verify the boot mode, list the efivars directory:
ls /sys/firmware/efi/efivars
 
timedatectl set-ntp true

lsblk

gdisk /dev/sda

#####.... create partitions table

#### root
mkfs.ext4 /dev/sda2

mount /dev/sda2 /mnt

#### home
cryptsetup -y -v luksFormat /dev/sda4

cryptsetup open /dev/sda4 home

mkfs.ext4 /dev/mapper/home

mkdir /mnt/home

mount /dev/mapper/home /mnt/home

#### efi boot
mkdir /mnt/boot

mkfs.vfat /dev/sda1

mount /dev/sda1 /mnt/boot

#### swap
mkswap /dev/sda3

swapon /dev/sda3

### final mounted table
sda        8:0    0 223.6G  0 disk  

├─sda1     8:1    0   512M  0 part  /mnt/boot

├─sda2     8:2    0    40G  0 part  /mnt

├─sda3     8:3    0     8G  0 part  [SWAP]

└─sda4     8:4    0 179.6G  0 part  

  └─home 254:0    0 179.6G  0 crypt /mnt/home


pacstrap /mnt base base-devel linux linux-firmware networkmanager terminus-font vim mc ranger

genfstab -U /mnt >> /mnt/etc/fstab

arch-chroot /mnt

systemctl enable NetworkManager

ln -sf /usr/share/zoneinfo/Asia/Barnaul /etc/localtime

hwclock --systohc
#### Edit /etc/locale.gen and uncomment en_US.UTF-8 UTF-8 and other needed locales. Generate the locales by running:
locale-gen

vim /etc/locale.conf
#### write 
LANG=en_US.UTF-8

LC_TIME=en_GB.UTF-8

vim /etc/vconsole.conf
#### write 
KEYMAP=ruwin_cplk-UTF-8

FONT=ter-v28b

vim /etc/hostname
#### write 
myhostname
 
vim /etc/hosts
#### write 
127.0.0.1 localhost

::1 localhost

127.0.1.1 nb

vim /etc/fstab
# home partition
/dev/mapper/home	                        /home     	ext4      	defaults	0 2

vim /etc/crypttab
# home partition
home           UUID=fb10f41d-42e0-4b3a-b362-b67b5c49487a    none                    luks,timeout=180

vim /etc/mkinitcpio.conf
#### write in HOOKS section
HOOKS=(base udev autodetect modconf block filesystems keyboard encrypt fsck)


#### root password
passwd

### install microcode
pacman -Sy intel-ucode
OR
pacman -Sy amd-ucode

### EFI systemd-boot
bootctl install

cd /boot/loader

vim loader.conf
#### write
default  arch.conf

timeout  4

console-mode max

editor   no

vim entries/arch.conf
#### write
title   Arch Linux

linux   /vmlinuz-linux

initrd  /intel-ucode.img

initrd  /initramfs-linux.img

options        root=PARTUUID=14420948-2cea-4de7-b042-40f67c618660 rw

#### get PARTUUID
blkid -s PARTUUID -o value /dev/sdxY

exit

umount -a

reboot



pacman -Sy xorg-server xorg-xinit i3-wm dmenu git guake picom pulseaudio

useradd -m -G wheel,adm -s /bin/bash nn

passwd nn
#### uncomment wheel group in /etc/sudoers

###Automatic login to virtual console
####Edit the provided unit either manually by creating the following drop-in snippet, or by running **systemctl edit getty@tty1** and pasting its content
[Service]

ExecStart=

ExecStart=-/usr/bin/agetty --autologin username --noclear %I $TERM

### Autostart X at login
#### Place the following in your shell initialization file .bashrc or .zshrc
[[ -z $DISPLAY && $XDG_VTNR -eq 1 ]] && exec startx

