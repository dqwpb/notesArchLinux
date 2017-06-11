# Note of install archlinux with uefi boot on RAID 1 hdd
### info:
```txt
/sda
 +-sda1 /boot   512M
 +-sda2
  +-md2 /       120G
 +-sda3
  +-md3 /qm     120G
 +-sda4
  +-md4 /db     233.5G
 +-sda5 /swap   1.5G

/sdb
 +-sdb1 /boot
 +-sdb2
  +-md2 /
 +-sdb3
  +-md3 /qm
 +-sdb4
  +-md4 /db
 +-sdb5 /swap
```

### 之前玩壞的，要重裝:
```bash
mdadm --stop /dev/md{1,2,3} # 之前玩壞的先關掉，要重裝
mdadm --zero-superblock /dev/sda{2,3,4}
```

### 分割表:
```bash
lsblk
cfdisk /dev/sda
---
gpt
===
/dev/sda1 esp
/dev/sda2 linux file system
/dev/sda3 linux file system
/dev/sda4 linux file system
/dev/sda5 linux swap
---
sgdisk /dev/sda -R=/dev/sdb # 把/dev/sda的分割表複製給/dev/sdb
sgdisk -G /dev/sdb
```

### 建立raid 1:
```bash
mdadm --create --verbose --level=1 --metadata=1.2 --raid-devices=2 /dev/md{2,3,4} /dev/sda{2,3,4} /dev/sdb{2,3,4}
echo 'DEVICE partitions' > /etc/mdadm.conf
mdadm --detail --scan >> /etc/mdadm.conf
mdadm --assemble --scan
```

### 格式化硬碟: 有建議計算公式:
```bash
cat /proc/mdstat
---
stride = chunk/block         md2和md3的stride = 65526/4 = 16384, md4的stride = 65536/8 = 8192
stripe = no(disk) * stride   md2和md3的stripe = 2*16384 = 32768, md4的stride = 2*8192 = 16384
---
mkfs.vfat -v -F 32 /dev/sda1
mkfs.ext4 -v -L <name{2,3,4}> -m -0 -b 4096 -E stride=<stride>,stripe-width=<stripe> /dev/md{2,3,4}
mkswap /dev/sda5
swapon /dev/sda5
```

接下來，把/md{2,3,4}看成/sda{2,3,4}做下去, /dev/sda5和/dev/sdb5都要swapon;

以下接著```pacstrap -i /mnt base base-devel grub```, 而且```genfstab```已經做了:

### 把raid表記下:

```bash
mdadm --detail --scan >> /mnt/etc/mdadm.conf
```

進入```arch-chroot```:

### 修改grub: 
```bash
nano /etc/default/grub
---
GRUB_DISABLE_LINUX_UUID=true
---
nano /etc/mkinitcpio.conf
---
HOOKS="bash ... block mdadm_udev filesystem ... "
...
BINARIES="/sbin/mdmon"
---
mkinitcpio -p linux
pacman -S efibootmgr
grub-install --recheck /dev/sda --efi-directory=/boot
grub-mkconfig -o /boot/grub/grub.cfg
```
