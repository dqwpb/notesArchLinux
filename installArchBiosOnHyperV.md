# Note of Note of install archlinux on hyperv with legacy boot(grub-bios) and lxqt desktop
## Add RemoteFX 3D 視訊卡: 
## 1.
```bash
lsblk
cfdisk /dev/sda
---
dos
===
/dev/sda1 linux file system
/dev/sda2 linux swap
---
mkfs.ext4 /dev/sda1
mkswap /dev/sda2
swapon /dev/sda2
```

## 2.
```bash
nano /etc/pacman.d/mirrorlist
---
Server = http://archlinux.cs.nctu.edu.tw/$repo/os/$arch
---
ip route add <gateway> dev eth0
ip route add default via <gateway> dev eth0
ip addr add <ip>/24 broadcast <broadcast> dev eth0
echo 'nameserver 8.8.8.8 >> /etc/resolv.conf'
echo 'nameserver 8.8.4.4 >> /etc/resolv.conf'
ping -c 4 tw.yahoo.com
```

## 3.
```bash
mount /dev/sda1 /mnt
pacstrap /mnt base base-devel intel-ucode grub-bios
```

## 4.
```bash
genfstab -U -p /mnt > /mnt/etc/fstab
```
(**ps**: genfstab -U -p /mnt | sudo tee /mnt/etc/fstab # archbox)

```bash
arch-chroot /mnt /bin/bash
```

## 5. 
```bash
echo LANG=en_US.UTF-8 > /etc/locale.conf 
echo archbiosv3 > /etc/hostname
nano /etc/locale.gen
---
en_US.UTF-8 UTF-8
zh_TW.UTF-8 UTF-8
---
locale-gen
ln -sf /usr/share/zoneinfo/Asia/Taipei /etc/localtime
hwclock --systohc --utc
timedatectl
timedatectl set-timezone Asia/Taipei
```

## 6. 
```bash
useradd -m -g users -s /bin/bash <username>
visudo
---
<username> ALL=(ALL) ALL
---
```

## 7. 
```bash
grub-install --target=i386-pc /dev/sda
nano /etc/default/grub
---
GRUB_CMDLINE_LINUX_DEFAULT="quiet splash video=hyperv_fb:800:600"
---
grub-mkconfig -o /boot/grub/grub.cfg
```

## 8. 
```bash
nano /home/<username>/setstaticnetwork.sh
---
#!/bin/bash

echo -n 'device: '
read d
echo -n 'gateway: '
read g
echo -n 'netmask: '
read m
echo -n 'broadcast: '
read b
echo -n 'ip: '
read ip
echo -n 'dns: '
read dns

ip link set dev $d up
ip route add $g dev $d
ip route add default via $g dev $d
ip addr add $ip/$m broadcast $b dev $d

echo 'nameserver '$dns > /etc/resolv.conf
---
bash /home/<username>/setstaticnetwork.sh
```

## 9.
```bash
pacman -S xorg-server xorg-xinit xorg-utils xorg-server-utils xorg-twm xterm xorg-xclock
pacman -S xf86-video-fbdev
pacman -S lxqt
pacman -S sddm slock
systemctl enable sddm
systemctl start sddm
pacman -S sakura pcmanfm
cp /etc/X11/xinit/xinitrc /home/<username>/.xinitrc
nano /home/<username>/.xinitrc
---
... &
exec startlxqt
---
nano /home/<username>/.bash_profile
---
[[ -z $DISPLAY && $XDG_VTNR -eq 1 ]] && exec startx
---
```

```bash
exit
umount -R /mnt
reboot now
```
