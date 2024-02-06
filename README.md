# arch-btw
How to install Arch btw
or just use archinstall who cares amirite?

# List Disks
lsblk > list all drives
    >choose a drive to that arch will be installed then type gdisk /dev/sd[x]
        >x (expert)
        >z (zap)
        >y
        >y

# Create Partition
```
cgdisk /dev/sd[x]
    >(boot drive) New > default > 1024MiB > EF00 > "boot"
    >(swap memory) New > default > [10% of the whole drive]  > 8200 > "swap
    >(root partition) New > default > [10% of the whole drive] > 8300 >    "root"
    >(drive memory) New > default > default > 8300 > "home"
```

# Reformat Disk and filesystem (FAT32)
```
mkfs.fat -F32 /dev/sd[x]1
mkswap /dev/sd[x]2
swapon /dev/sd[x]2
mkfs.ext4 /dev/sd[x]3
mkfs.ext4 /dev/sd[x]4
```

# Install Linux
```
mount /dev/sd[x]3 /mnt
mkdir /mnt/boot
mkdir /mnt/home
mount /dev/sd[x]1 /mnt/boot
mount /dev/sd[x]4 /mnt/home
```
# Install packages
```
cp /etc/pacman.d/mirrorlist /etc/pacman.d/mirrorlist.backup
sudo pacman -Sy pacman-contrib
    >pacman-key --init
    >pacman-key --populate
rankmirrors -n 6 /etc/pacman.d/mirrorlist.backup > /etc/pacman.d/mirrorlist
pacstrap -K /mnt base linux linux-firmware base-devel
```

# Generate fstab
```
genfstab -U -p /mnt >> /mnt/etc/fstab
```

# Access root and configure timezone
```
arch-chroot /mnt
sudo pacman -S nano bash-completion
nano /etc/locale.gen
    >uncomment region
locale-gen
echo LANG=en_PH.UTF-8 > /etc/locale.conf
export LANG=en_PH.UTF-8
ls /usr/share/zoneinfo
ln /usr/share/zoneinfo/Singapore > etc/localtime
hwclock --systohc --utc
```

# create user
```
echo [username] > etc/hostname
```
# enable trim support for ssd
```
systemctl enable fstrim.timer
```

# enable 32 bit support
```
nano etc/pacman.conf
    >find multilib and uncomment [multilib] and Include = /etc/pacman.d/mirrorlist
    >save
sudo pacman -Sy
```

# set root pw
```
passwd
```

# add user
```
useradd -m -g users -G wheel,storage,power -s /bin/bash [un]
passwd [un]
```

# add wheels
```
EDITOR=nano visudo
    >uncomment %wheel ALL=(ALL:ALL) ALL
    >add "Defaults rootpw" eol
```
# bootloader install
```
	#check if on efi
	mount -t efivarfs efivarfs /sys/firmware/efi/efivars
	ls /sys/firmware/efi/efivars
bootctl install
```

# if encountered a warning
```
nano /etc/fstab
	>edit fmask=0137,dmask=0027
 	>umount /boot
	>mount /boot
 	>bootctl install
```

# write bootentry
```
nano /boot/loader/entries/arch.conf
title [anytitle]
linux /vmlinux-linux
initrd /initranfs-linux.img
echo "options root=PARTUUID=$(blkid -s PARTUUID -o value /dev/sd[x]3) rw" >> /boot/loader/entries/arch.conf
```
# enable dhcpcd
```
ip link
sudo pacman -S dhcpcd
sudo systemctl enable dhcpcd@[iplink].service
```
# install network manager
```
sudo pacman -S networkmanager
sudo systemctl enable NetworkManager.service
```
# install nvidia driver(optional)
```
	!important
	sudo pacman -S linux-headers
sudo pacman -S nvidia-dkms libglvnd nvidia-utils opencl-nvidia lib32-libglvnd lib32-nvidia-utils lib32-opencl-nvidia nvidia-settings
sudo nano /etc/mkinitcpio.conf
	>MODULE=(nvidia nvidia_modeset nvidia_uvm nvidia_drm)
	>save
sudo nano /boot/loader/entries/arch.config
	>add "nvidia-drm.modeset=1" right beside rw
sudo mkdir /etc/pacman.d/hooks
sudo nano /etc/pacman.d/hooks/nvidia.hook
	>
	```
	[Trigger]
	Operation=Install
	Operation=Upgrade
	Operation=Remove
	Type=Package
	Target=nvidia

	[Action]
	Depends=mkinitcpio
	When=PostTransaction
	Exec=/usr/bin/mkinitcpio -P
	```
```
# Reboot
```
exit
umount -R /mnt
reboot
```

# install xorg/wayland (xorg)
```
sudo pacman -S xorg-server xorg-apps xorg-xinit xorg-xinit xorg-twm xorg-xclock xterm
	>default
	>y
startx
```

# install plasma
```
sudo pacman -S plasma sddm
	>defaults all the way
	>y
sudo systemctl enable sddm.service
reboot
```
