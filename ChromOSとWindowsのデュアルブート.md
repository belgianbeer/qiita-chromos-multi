# ChromeOS FlexとWindowsやmacOS等のマルチブート環境を作る

## はじめに

ChromOS Flexは1台のPCでの他のOSとのマルチブートをサポートしていません。実際にChromOS Flexをインストールしたことのある人なら分かりますが、別のOSが動いているPCにChromOS Flexをインストールすると問答無用で既存のOSの領域まで含めてすべてのストレージをChromeOS Flexに割り当ててしまい、使用容量を調整する手段は用意されていません。ですから通常の方法では他のOSを同時にインストールするためのストレージが確保できないわけです。

この記事は裏技的な手法を使って、1台のPCにChromeOS Flexと他のOSとのマルチブート環境を構築する方法について説明します。あくまでも裏技なので将来にわたってこの方法でChromeOS Flexとのマルチブート環境が作れるかどうかはわかりませんが、当面は大丈夫そうです。

## ChromeOS Flexってどうなの？

ChromeOS Flexってどうなの？と聞かれることが有りますが、WindowsやmacOSがストレスなく動くPCであればそれらを使った方が便利なのは間違いありません。しかし実際にChromeOS Flexを使ってみると内部で余計なアプリが動かないこともあって、動作は非常に軽く起動や終了の速さは特筆ものです。おかげで少々非力なPCであっても快適に利用できます。またWebを中心とした今時の作業であれば、オンラインミーティングやWord, Excelファイルの編集など様々なアプリケーションを動作させられるため、一般的な利用であればほとんど困らないのでは無いかと思います。

## お勧めしません

いきなりですが、**ChromeOS Flexと他のOSのデュアルブート環境を作るのはお勧めとは言えません**。理由は簡単でChromeOS Flexとのマルチブート環境のPCでWindowsやmacOSからシステムディスクに対する何らかの操作を行うと、ChromeOS Flexのパーティション情報を破壊する為です。これはChromeOS Flexのディスクパーティションが、一般的な構成と違って少々トリッキーになっていることによります。破壊されてもパーティション情報のバックアップがあれば復帰できるのですが、手間がかかりますし、あれ？っと思ったときに速やかに対処を行うセンスが要求されます。とは言えこの記事にその方法を記載しますから、本記事が理解できる人であれば困ることは無いと思います。

## 用意するもの

ChromeOS Flexとのマルチブート環境を作るのに必要なものは次の通りです。

1. UEFIをサポートしたPCまたはMac(Intel CPU)本体
1. ChromeOS Flexのインストール用USBメモリ
1. Windows 10か11、またはmacOSのインストール用USBメモリ
1. Debian Liveを書き込んだUSBメモリ
1. 記録用のUSBメモリ

UEFIをサポートしていないPCでもChromeOS Flex自体は動くんじゃないかと思いますが(未確認)、他のOSとのマルチブートを行うためにGPT(Guid Partition Table)を扱えるUEFI対応のPCが必須となります。Intel CPUのMacであれば条件を満たしています。[ChromeOS Flexの認定モデル](https://support.google.com/chromeosflex/answer/11513094 "認定モデルリスト")に該当してなくても、よほどでなければChromeOS Flexが動作すると思います。

マルチブート環境を作るのですから、OSインストール用のUSBメモリがChromeOS FlexとWindowsまたはmacOSのものが必要なのは当然として、ディスクパーティションの構成を編集するためにDebian LiveのUSBメモリも必要になります。Debian Liveでなくても良いのですがシェルが起動できてsfdiskとエディタ等が利用できるLinuxの起動用USBメモリが条件となります。

この他に記録用のUSBメモリも必要になります。Debian LiveのUSBメモリに記録できれば済むのですが、そのままではできないような気がしています(筆者が知らないだけかも)。パーティション構成のテキストファイルを2, 3個保存するだけなので大した容量は要りません。

## ChromeOS Flexのインストール

まずPCを用意してChromeOS Flexをインストールします。ChromOS Flexのインストール用USBメモリで起動してメニューにしたがって操作すればインストールで難しいところは無いでしょう。Googleも[ChromOS Flexインストールガイド](https://support.google.com/chromeosflex/answer/11552529)を用意していて、インストール用のUSBメモリの作り方などもこちらにあります。

通常であればインストール後に起動してアカウント設定等を行うのですが、ここでは必要ありません。設定してもこの後の作業で消去されてしまい、改めて設定することになります。

## ChromeOS Flexのパーティション構成

ChromeOS Flexのインストールが終わったら、今度はDebian LiveのUSBメモリで起動します。Debian Li

Debian LiveのUSBメモリで起動する




## Debian Liveでのブート

```
sudo -i 
apt update
apt install dosfstools openssh-server efibootmgr 
systemctl start ssh
```


```
# sfdisk -l /dev/sda | tee p1-sda.list
Disk /dev/sda: 238.47 GiB, 256060514304 bytes, 500118192 sectors
Disk model: PALIT PSP256 SSD
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: gpt
Disk identifier: 9A788614-6CD4-1B46-9F6F-748287F5B228

Device        Start       End   Sectors   Size Type
/dev/sda1  17010688 500118143 483107456 230.4G Linux filesystem
/dev/sda2        69     32836     32768    16M ChromeOS kernel
/dev/sda3   8622080  17010687   8388608     4G ChromeOS root fs
/dev/sda4     32837     65604     32768    16M ChromeOS kernel
/dev/sda5    233472   8622079   8388608     4G ChromeOS root fs
/dev/sda6        65        65         1   512B ChromeOS kernel
/dev/sda7        66        66         1   512B ChromeOS root fs
/dev/sda8     69632    102399     32768    16M Linux filesystem
/dev/sda9        67        67         1   512B ChromeOS reserved
/dev/sda10       68        68         1   512B ChromeOS reserved
/dev/sda11       64        64         1   512B unknown
/dev/sda12   102400    233471    131072    64M EFI System

Partition table entries are not in disk order.
#

# sfdisk --dump /dev/sda | tee /tmp/p1-sda.dump
label: gpt
label-id: 9A788614-6CD4-1B46-9F6F-748287F5B228
device: /dev/sda
unit: sectors
first-lba: 34
last-lba: 500118158
sector-size: 512

/dev/sda1 : start=    17010688, size=   483107456, type=0FC63DAF-8483-4772-8E79-3D69D8477DE4, uuid=26FE530D-CCE5-2A4D-B5AA-248FA77B01C9, name="STATE"
/dev/sda2 : start=          69, size=       32768, type=FE3A2A5D-4F32-41A7-B725-ACCC3285A309, uuid=C5F52E17-1AA7-2144-BFC4-3E957668C4C5, name="KERN-A", attrs="GUID:48,53,54,56"
/dev/sda3 : start=     8622080, size=     8388608, type=3CB8E202-3B7E-47DD-8A3C-7FF2A13CFCEC, uuid=1604F89D-9533-8F45-A25F-95F18C69E3B9, name="ROOT-A"
/dev/sda4 : start=       32837, size=       32768, type=FE3A2A5D-4F32-41A7-B725-ACCC3285A309, uuid=9E34E184-4D12-C142-82A9-BA998B2A5968, name="KERN-B", attrs="GUID:52,53,54,55"
/dev/sda5 : start=      233472, size=     8388608, type=3CB8E202-3B7E-47DD-8A3C-7FF2A13CFCEC, uuid=4EB5B9D5-B4C0-254D-AC32-075D6E4BC90B, name="ROOT-B"
/dev/sda6 : start=          65, size=           1, type=FE3A2A5D-4F32-41A7-B725-ACCC3285A309, uuid=D66DA0CC-AB74-1E47-96DE-F1F172456FE8, name="KERN-C", attrs="GUID:52,53,54,55"
/dev/sda7 : start=          66, size=           1, type=3CB8E202-3B7E-47DD-8A3C-7FF2A13CFCEC, uuid=7B71E8C3-19C8-7B46-B3BA-D3BBD7F63FA7, name="ROOT-C"
/dev/sda8 : start=       69632, size=       32768, type=0FC63DAF-8483-4772-8E79-3D69D8477DE4, uuid=6A967904-870F-6647-A84D-9E0007F5C761, name="OEM"
/dev/sda9 : start=          67, size=           1, type=2E0A753D-9E48-43B0-8337-B15192CB1B5E, uuid=D0C724DD-A6C5-C74B-B887-B4A39E47987B, name="reserved"
/dev/sda10 : start=          68, size=           1, type=2E0A753D-9E48-43B0-8337-B15192CB1B5E, uuid=31F964AE-87D4-7345-9609-94FA965B9C7C, name="reserved"
/dev/sda11 : start=          64, size=           1, type=CAB6E88E-ABF3-4102-A07A-D4BB9BE3C1D3, uuid=A0DAB71B-CC02-FB41-867C-FD4372385941, name="RWFW"
/dev/sda12 : start=      102400, size=      131072, type=C12A7328-F81F-11D2-BA4B-00A0C93EC93B, uuid=72511358-8D4A-7B41-A256-57CFE6E97258, name="EFI-SYSTEM", attrs="LegacyBIOSBootable"
#
```

```
# cp p1-sda.dump p2-sda.dump
# uuidgen
8C678F1E-729F-484B-8368-92DF1332E62A
# vi p2-sda.dump


# sfdisk /dev/sda < p2-sda.dump

# sfdisk -l /dev/sda
Disk /dev/sda: 238.47 GiB, 256060514304 bytes, 500118192 sectors
Disk model: PALIT PSP256 SSD
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: gpt
Disk identifier: 9A788614-6CD4-1B46-9F6F-748287F5B228

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
```

ファイルシステムを作る
```
# mkfs.ext4 /dev/sda1
mke2fs 1.46.2 (28-Feb-2021)
/dev/sda1 contains a ext4 file system labelled 'H-STATE'
        last mounted on /mnt/stateful_partition on Fri Mar 17 04:05:56 2023
Proceed anyway? (y,N) y
Discarding device blocks: done
Creating filesystem with 5242880 4k blocks and 1310720 inodes
Filesystem UUID: cf0c6b34-d9d8-437e-8f4e-c1b2c939afc8
Superblock backups stored on blocks:
        32768, 98304, 163840, 229376, 294912, 819200, 884736, 1605632, 2654208,
        4096000

Allocating group tables: done
Writing inode tables: done
Creating journal (32768 blocks): done
Writing superblocks and filesystem accounting information: done

#
# mkdosfs  -F 32 -n EFI-SYSTEM /dev/sda13
mkfs.fat 4.2 (2021-01-31)
# 
```
## パーティションをマウントする
```
# mkdir /mnt/efi /mnt/dos
# mount /dev/sda12 /mnt/efi
# mount /dev/sda13 /mnt/dos
```

## EFIの内容をコピーする
```
# (cd /mnt/efi ; tar cf - * ) | (cd /mnt/dos ; tar xvf -)
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
# 
```

EFIのパーティションを今回作ったdosパーティションにおきかえる
```
cp p2-sda.dump p3-sda.dump
# diff -U0 p2-sda.dump p3-sda.dump
--- p2-sda.dump 2023-03-17 04:26:24.000000000 +0000
+++ p3-sda.dump 2023-03-17 04:47:08.000000000 +0000
@@ -20,2 +20 @@
-/dev/sda12 : start=      102400, size=      131072, type=C12A7328-F81F-11D2-BA4B-00A0C93EC93B, uuid=72511358-8D4A-7B41-A256-57CFE6E97258, name="EFI-SYSTEM", attrs="LegacyBIOSBootable"
-/dev/sda13 : start=    58953728, size=      524288, type=EBD0A0A2-B9E5-4433-87C0-68B6B72699C7, uuid=33ff6d04-0b5a-4c61-86bc-a8abf8e65428
+/dev/sda12 : start=    58953728, size=      524288, type=C12A7328-F81F-11D2-BA4B-00A0C93EC93B, uuid=72511358-8D4A-7B41-A256-57CFE6E97258, name="EFI-SYSTEM", attrs="LegacyBIOSBootable"
# umount /mnt/dos
# umount /mnt/efi
# sfdisk /dev/sda < p3-sda.dump
# sfdisk --list /dev/sda  | tee p3-sda.list
Disk /dev/sda: 238.47 GiB, 256060514304 bytes, 500118192 sectors
Disk model: PALIT PSP256 SSD
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: gpt
Disk identifier: 9A788614-6CD4-1B46-9F6F-748287F5B228

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
/dev/sda12 58953728 59478015   524288  256M EFI System

Partition table entries are not in disk order.
# 
```
## 変更点を確認してみます
```
# diff -U0 --ignore-space-change p1-sda.list p3-sda.list
--- p1-sda.list 2023-03-17 04:18:02.000000000 +0000
+++ p3-sda.list 2023-03-17 04:53:00.000000000 +0000
@@ -10 +10 @@
-/dev/sda1  17010688 500118143 483107456 230.4G Linux filesystem
+/dev/sda1  17010688 58953727 41943040   20G Linux filesystem
@@ -21 +21 @@
-/dev/sda12   102400    233471    131072    64M EFI System
+/dev/sda12 58953728 59478015   524288  256M EFI System
#
```
## 空きパーティションを隠す

```
# cp p3-sda.dump p4-sda.dump-temp
# diff -U0 p3-sda.list p4-sda.list-temp
```
## Window のインストール


## Debian Live で起動

```
# sfdisk --list /dev/sda > p5-sda.list
# sfdisk --dump /dev/sda > p5-sda.dump
# cp p3-sda.dump p6-sda.dump
# tail -2 p5-sda.dump >> p6-sda.dump
# sfdisk /dev/sda < p6-sda.dump
# sfdisk --list /dev/sda > p6-sda.list
```