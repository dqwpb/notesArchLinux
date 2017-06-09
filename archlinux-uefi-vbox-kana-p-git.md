# 安裝Virtualbox客體Arch Linux與一個極簡但功能完全的桌面:

**說明**:以下都在客戶端執行,主體端以octopi裝完後會自動啟動該有服務.

## 1. 安裝基本系統
### 1.a. 磁碟分割:
```bash
cfdisk /dev/sda
/dev/sda1:256M
/dev/sda2
```
### 1.b. 格式化:
```bash
mkfs.vfat -v -F 32 /dev/sda1
mkfs.ext4 /dev/sda2
```
**ps**: 如果有必要,建立swap分割:
```bash
mkswap /dev/sda3
swapon /dev/sda3
```
### 1.c. 設定網路:
```bash
ip route add <閘道> dev <設備>
ip route add default via <閘道> dev <設備>
ip addr add <位址>/<遮罩> broadcast <廣播位址> dev <設備>
echo 'nameserver 8.8.8.8' >> /etc/resolv.conf
echo 'nameserver 8.8.4.4' >> /etc/resolv.conf
```
### 1.d. 設定快速更新鏡像:
```bash
nano /etc/pacman.d/mirrorlist
---
Server = http://archlinux.cs.nctu.edu.tw/$repo/os/$arch
---
```
### 1.e. 掛載硬碟:
```bash
mount /dev/sda2 /mnt
mkdir /mnt/boot
mount /dev/sda1 /mnt/boot
pacstrap -i /mnt base base-devel intel-ucode grub
```
### 1.f. 產生fstab:
```bash
genfstab -U -p /mnt > /mnt/etc/fstab
```
**ps**: 以後要永久掛載某硬碟:
先lsblk再sudo blkid後把UUID補上/etc/fstab;
為了不要開機檢查所有磁碟分割,可以把pass設為0;
若是ssd還須加參數:discard,noatime.
### 1.g. 進入系統:
```bash
arch-chroot /mnt /bin/bash
```
### 1.h. 設定語言: 設為英文,目標為可以顯示中文,安裝中文輸入法
```bash
echo LANG=en_US.UTF-8 > /etc/locale.conf
nano /etc/locale.gen
---
en_US.UTF-8 UTF-8
zh_TW.UTF-8 UTF-8
---
locale-gen
```
### 1.h. 設定時區:
```bash
ln -sf /usr/share/zoneinfo/Asia/Taipei /etc/localtime
hwclock --systohc --utc
```
**ps**: 可以用date檢查時間是否正確.
### 1.i. 建立使用者:
```bash
passwd # 設定root的密碼
useradd -m -g users -s /bin/bash <使用者名稱>
passwd <使用者名稱>
```
**ps**: 安裝完重開機後要先設定<使用者名稱>的sudo權限:
```bash
sudo su
visudo
---
<使用者名稱> ALL=(ALL) ALL
---
```
### 1.j. 設定電腦名稱:
```bash
echo <電腦名稱> > /etc/hostname
```
### 1.k. 安裝EFI開機檔案:
```bash
pacman -S efibootmgr
grub-install --recheck /dev/sda --efi-directory=/boot
grub-mkconfig -o /boot/grub/grub.cfg
```
### 1.l. 建立網路快速啟用腳本:
```bash
nano /home/<username>/<腳本名稱>.sh
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
#echo -n 'dns: '
#read dns
dns1="8.8.8.8"
dns2="8.8.4.4"

ip link set dev $d up
ip route add $g dev $d
ip route add default via $g dev $d
ip addr add $ip/$m broadcast $b dev $d

echo 'nameserver '$dns1 > /etc/resolv.conf
echo 'nameserver '$dns2 >> /etc/resolv.conf
---
```
**ps**: 用ip addr查出<設備名稱>;
```bash
sudo bash <腳本名稱>.sh
```
### 1.m. 設定Virtualbox專屬開機指令稿:
```bash
echo "fs0:\EFI\arch\grubx64.efi" > /boot/startup.nsh
```
### 1.n. 退出重新啟動:
```bash
exit
umount -R /mnt
reboot now
```
## 2. 安裝Virtualbox客戶端元件:
### 2.a. linux檔頭: 此套件會影響guest-addition安裝
```bash
sudo pacman -S linux-headers
```
### 2.b. guest-addition映像檔: 此元件安裝成功與否會影響桌面啟動
```bash
sudo pacman -S virtualbox-guest-iso
sudo mount -o loop /usr/lib/virtualbox/additions/VBoxGuestAdditions.iso /mnt
cd /mnt
sudo ./VBoxLinuxAddition.sh
sudo reboot now
```
## 3. 安裝kana-p-git桌面:
### 3.a. penlight: 會影響awesome-git的build
```bash
sudo pacman -S luarocks
luarocks install penlight
```
### 3.b. 安裝AUR Helper: yaourt (安裝AUR套件不須sudo)
```bash
sudo nano /etc/pacman.conf
---
[archlinuxfr]
SigLevel = Never
Server = http://repo.archlinux.fr/$arch
---
sudo pacman -Sy yaourt
```
**ps**: 手動安裝AUR套件:
```bash
tar xzvf <套件名稱+版號>.xz
cd <套件名稱>
makepkg -c
sudo pacman -U 套件名稱
```
### 3.c. awesome-git: 重要相依套件
```bash
yaourt -S awesome-git
```
### 3.d. kana-p-git:
```bash
yaourt -S kana-p-git
sudo reboot now
```
(待續)

1. fcitx-chewing
2. vim: PlugInstall
3. openjdk
4. ...
