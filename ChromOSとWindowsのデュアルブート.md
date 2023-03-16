# ChromeOS FlexとWindowsのデュアルブートPCを作る

# おことわり

用意するもの

1. UEFIをサポートしたPC
1. Debian Liveを書き込んだUSBメモリ
1. ChromeOS Flexのインストール用USBメモリ
1. Windows 10 or 11 のインストール用USBメモリ

まずChromeOS Flexをインストールする

Debian LiveのUSBメモリで起動する

sfdisk -dump /dev/sda
```

```

dosfstools を追加


```
root@debian:~/work/disk# fdisk /dev/sda

Welcome to fdisk (util-linux 2.36.1).
Changes will remain in memory only, until you decide to write them.
Be careful before using the write command.


Command (m for help): p
Disk /dev/sda: 238.47 GiB, 256060514304 bytes, 500118192 sectors
Disk model: PALIT PSP256 SSD
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: gpt
Disk identifier: ABFF6D06-0059-D44E-9FA3-B23D4EB13301

Device        Start      End  Sectors  Size Type
/dev/sda1  17010688 58953727 41943040   20G Linux filesystem
/dev/sda2        69    32836    32768   16M ChromeOS kernel
/dev/sda3   8622080 17010687  8388608    4G ChromeOS root fs
/dev/sda4     32837    65604    32768   16M ChromeOS kernel
/dev/sda5    233472  8622079  8388608    4G ChromeOS root fs
/dev/sda6        65       65        1  512B ChromeOS kernel
/dev/sda7        66       66        1  512B ChromeOS root fs
/dev/sda8     69632   102399    32768   16M Linux filesystem
/dev/sda9        67       67        1  512B ChromeOS reserved
/dev/sda10       68       68        1  512B ChromeOS reserved
/dev/sda11       64       64        1  512B unknown
/dev/sda12   102400   233471   131072   64M EFI System

Partition table entries are not in disk order.

Command (m for help): n
Partition number (13-128, default 13):
First sector (65605-500118158, default 58953728):
Last sector, +/-sectors or +/-size{K,M,G,T,P} (58953728-500118158, default 500118158): +256M

Created a new partition 13 of type 'Linux filesystem' and of size 256 MiB.

Command (m for help): t
Partition number (1-13, default 13): 13
Partition type or alias (type L to list all): 11

Changed type of partition 'Linux filesystem' to 'Microsoft basic data'.

Command (m for help): p
Disk /dev/sda: 238.47 GiB, 256060514304 bytes, 500118192 sectors
Disk model: PALIT PSP256 SSD
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: gpt
Disk identifier: ABFF6D06-0059-D44E-9FA3-B23D4EB13301

Device        Start      End  Sectors  Size Type
/dev/sda1  17010688 58953727 41943040   20G Linux filesystem
/dev/sda2        69    32836    32768   16M ChromeOS kernel
/dev/sda3   8622080 17010687  8388608    4G ChromeOS root fs
/dev/sda4     32837    65604    32768   16M ChromeOS kernel
/dev/sda5    233472  8622079  8388608    4G ChromeOS root fs
/dev/sda6        65       65        1  512B ChromeOS kernel
/dev/sda7        66       66        1  512B ChromeOS root fs
/dev/sda8     69632   102399    32768   16M Linux filesystem
/dev/sda9        67       67        1  512B ChromeOS reserved
/dev/sda10       68       68        1  512B ChromeOS reserved
/dev/sda11       64       64        1  512B unknown
/dev/sda12   102400   233471   131072   64M EFI System
/dev/sda13 58953728 59478015   524288  256M Microsoft basic data

Partition table entries are not in disk order.

Command (m for help): w
The partition table has been altered.
Failed to add partition 13 to system: Device or resource busy

The kernel still uses the old partitions. The new table will be used at the next reboot.
Syncing disks.

root@debian:~/work/disk# fdisk /dev/sda




# mkfs.ext4 /dev/sda1
mke2fs 1.46.2 (28-Feb-2021)
/dev/sda1 contains a ext4 file system
        created on Thu Mar 16 09:52:31 2023
Proceed anyway? (y,N) y
Discarding device blocks: done
Creating filesystem with 5242880 4k blocks and 1310720 inodes
Filesystem UUID: 338f0d1a-7d59-4de3-b2cd-31affe175685
Superblock backups stored on blocks:
        32768, 98304, 163840, 229376, 294912, 819200, 884736, 1605632, 2654208,
        4096000

Allocating group tables: done
Writing inode tables: done
Creating journal (32768 blocks): done
Writing superblocks and filesystem accounting information: done

#


root@debian:~# mkdir /mnt/old
root@debian:~# mkdir /mnt/new

Partition table entries are not in disk order.
root@debian:~# mkdosfs -F 32 -n EFI /dev/sd12
mkfs.fat 4.2 (2021-01-31)
mkdosfs: unable to open /dev/sd12: No such file or directory
root@debian:~# mkdosfs -F 32 -n EFI /dev/sda12
mkfs.fat 4.2 (2021-01-31)
root@debian:~# mount /dev/sda13 /mnt/old
root@debian:~# mount /dev/sda12 /mnt/new/
root@debian:~# ls /mnt/*
/mnt/new:


root@debian:~# (cd /mnt/old ; tar cf - * ) | (cd /mnt/new ; tar xvf -)
efi/
efi/boot/
efi/boot/bootx64.efi
efi/boot/bootia32.efi
efi/boot/grubx64.efi
efi/boot/grubia32.efi
efi/boot/grub.cfg
syslinux/
syslinux/syslinux.cfg
syslinux/default.cfg
syslinux/usb.A.cfg
syslinux/root.A.cfg
syslinux/root.B.cfg
syslinux/README
syslinux/vmlinuz.A
syslinux/vmlinuz.B
syslinux/ldlinux.sys
syslinux/ldlinux.c32
root@debian:~#
```
