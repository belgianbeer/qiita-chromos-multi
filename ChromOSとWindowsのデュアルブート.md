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
1. ChromeOS Flexのインストール用USBメモリ(8GB以上)
1. Windows 10か11、またはmacOSのインストール用USBメモリ(Windowsであれば8GB以上、macOSでは16GB以上)
1. Debian Liveを書き込んだUSBメモリ(2GB以上)
1. 記録用のUSBメモリ(100KB以上の空きがあるもの)

UEFIをサポートしていないPCでもChromeOS Flex自体は起動できますが、他OSとのブートの切り替えにUEFIに用意されているブートセレクタを使うため、GPT(Guid Partition Table)を扱えるUEFI対応のPCが必須となります。PCがIntel CPUのMacであれば条件を満たしています。

### [ChromeOS Flexの認定モデル](https://support.google.com/chromeosflex/answer/11513094 "認定モデルリスト")に該当してなくても、よほどでなければChromeOS Flexが動作すると思います。

マルチブート環境を作るのですから、OSインストール用のUSBメモリがChromeOS FlexとWindowsまたはmacOSのものが必要なのは当然として、ディスクパーティションの構成を編集するためにDebian LiveのUSBメモリも必要になります。Debian Liveでなくても良いのですがシェルが起動できてsfdiskとエディタ等が利用できるLinuxの起動用USBメモリが条件となります。

この他に記録用のUSBメモリも必要になります。Debian LiveのUSBメモリに記録できれば済むのですが、Debian Liveではファイルシステムの肝心な部分はRead Onlyでマウントされているため書き込むことができません。パーティション構成のテキストファイルを数個保存するだけなので100Kバイトもあれば間に合います。手頃なものを用意してください。

## ChromeOS Flexのインストール

最初にChromeOS Flexをインストールするのですが、インストール前にBIOSの設定を確認します。少し古いPCでUEFIに対応したBIOSであってもUEFIが無効になっていることがあるので、必ず確認してください。Macの場合は事前に設定することはありません。

ChromOS Flexのインストールに関しては、USBメモリで起動してメニューにしたがって順に操作するだけなので難しいところは無いでしょう。Googleも[ChromOS Flexインストールガイド](https://support.google.com/chromeosflex/answer/11552529)を用意していて、インストール用のUSBメモリの作り方などもこちらにあります。

通常であればインストール後に起動してアカウント設定等を行うのですが、ここでは必要ありません。設定してもこの後の作業で消去されてしまい、改めて設定することになります。とは言えChromeOS Flexが動作することを確認しておくことは重要なので、ある程度のことは試すのがよいでしょう。

## Debian Liveの起動

ChromeOS Flexのインストールが終わったら、今度はDebian LiveのUSBメモリで起動します。起動前にLANケーブルを接続してPCがインターネットにアクセスできるようにしておく必要があります。起動するとシェルが立ち上がりますが、少し古いバージョンのDebian Liveのイメージを使った場合はログインプロンプトになることがあります。そのときはユーザー名に「user」パスワードに「live」を指定してログインします。

そして、記録用のUSBメモリをPCに接続します。

多くの場合Debian環境下でのストレージデバイスの割り当ては次のようになると思います。

<table>
  <caption>ストレージとデバイス名の対応</caption>
  <thead>
    <tr>
      <th>ストレージの種類</th> <th>デバイス名</th>
    </tr>
  </thead>
  <tr>
    <td> 内蔵ストレージ </td> <td>/dev/sda</td>
  </tr>
  <tr>
    <td> Debian LiveのUSBメモリ</td> <td>/dev/sdb</td>
  </tr>
  <tr>
    <td> 記録用のUSBメモリ</td> <td>/dev/sdc</td>
  </tr>
</table>

ストレージの構成によっては違う割り当てになることもあるので、実際のストレージとデバイス名の割り当てを正しく把握してください。以後はこの割り当てを前提に説明します。

### Debianでの事前準備

`sudo -i`でrootユーザーになります。

```
user@debian:~$ sudo -i
root@debian:~# 
```

aptを使ってdosfstoolsをインストールします。

```
root@debian:~# apt update
root@debian:~# apt install dosfstools
root@debian:~# 
```

記録用のUSBメモリをマウントして、マウントポイントにcdします。これでUSBメモリに記録が残せるようになります。

```
root@debian:~# mount /dev/sdc1 /mnt
root@debian:~# cd /mnt
root@debian:/mnt# 
```

### ChromeOS Flexの不思議なパーティション構成

最初にChromeOS Flexのパーティション構成を確認します。ここでは`sfdisk --list`を使います。

```
root@debian:/mnt# sfdisk --list /dev/sda
Disk /dev/sda: 111.79 GiB, 120034123776 bytes, 234441648 sectors
Disk model: INTEL SSDSC2BW12
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: gpt
Disk identifier: AC161E76-BF4B-924D-9C72-06CE3C6EABCF

Device        Start       End   Sectors   Size Type
/dev/sda1  17010688 234441599 217430912 103.7G Linux filesystem
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
root@debian:/mnt#
```

パーティション構成を見たことのある人なら違和感を覚えると思いますが、ChromeOS Flexではパーティションのインデックスとディスクの物理的な順番が一致していません。たとえばsda1のスタートセクタは170010688で、次のsda2の69より大きくなっています。通常ディスクにパーティションを作成する場合ディスクの先頭から順に割り当てるので、パーティションインデックとスタートセクタの値はどちらも小さいものから順に並びます。ところがChromeOS Flexではこのようにバラバラの順番でパーティションが並んでいます。おかげで`Partition table entries are not in disk order.`というメッセージまで表示されています。

### ChromOS Flexのディスクパーティション構成のバックアップ

ディスクパーティションの物理と論理の順番が一致していないことが、この後WindowsやmacOSをインストールした後に問題となります。そこでChromeOS Flexインストール直後のパーティション構成のバックアップを保存しておき、後で使います。バックアップには`sfdisk --dump`を使い、ここでは`p1-sda-dump`というファイルに保存しています。

```
root@debian:/mnt# sfdisk --dump /dev/sda | tee p1-sda-dump
label: gpt
label-id: AC161E76-BF4B-924D-9C72-06CE3C6EABCF
device: /dev/sda
unit: sectors
first-lba: 34
last-lba: 234441614
sector-size: 512

/dev/sda1 : start=    17010688, size=   217430912, type=0FC63DAF-8483-4772-8E79-3D69D8477DE4, uuid=08FD7E32-17D4-7341-934F-06FE44B1237F, name="STATE"
/dev/sda2 : start=          69, size=       32768, type=FE3A2A5D-4F32-41A7-B725-ACCC3285A309, uuid=A8FC6629-5055-A040-A9A5-415A8EA50C1B, name="KERN-A", attrs="GUID:48,56"
/dev/sda3 : start=     8622080, size=     8388608, type=3CB8E202-3B7E-47DD-8A3C-7FF2A13CFCEC, uuid=358D4F01-CDE0-5D4C-BB65-E91160A04FE2, name="ROOT-A"
/dev/sda4 : start=       32837, size=       32768, type=FE3A2A5D-4F32-41A7-B725-ACCC3285A309, uuid=9F96728D-96A8-6745-892D-91084EAF2481, name="KERN-B", attrs="GUID:49,56"
/dev/sda5 : start=      233472, size=     8388608, type=3CB8E202-3B7E-47DD-8A3C-7FF2A13CFCEC, uuid=1030FFBB-8B4C-5049-9B22-20878DC32FE7, name="ROOT-B"
/dev/sda6 : start=          65, size=           1, type=FE3A2A5D-4F32-41A7-B725-ACCC3285A309, uuid=3AB9CA80-136D-1641-B222-3C0BBEC2B66D, name="KERN-C", attrs="GUID:52,53,54,55"
/dev/sda7 : start=          66, size=           1, type=3CB8E202-3B7E-47DD-8A3C-7FF2A13CFCEC, uuid=170F4F46-6D5F-ED48-82EB-39FEEE521751, name="ROOT-C"
/dev/sda8 : start=       69632, size=       32768, type=0FC63DAF-8483-4772-8E79-3D69D8477DE4, uuid=9E8B7A51-B372-5B44-B4D4-EB4ABE5F77CF, name="OEM"
/dev/sda9 : start=          67, size=           1, type=2E0A753D-9E48-43B0-8337-B15192CB1B5E, uuid=32CF3879-ABD4-B74D-A864-7E734A2DB89E, name="reserved"
/dev/sda10 : start=          68, size=           1, type=2E0A753D-9E48-43B0-8337-B15192CB1B5E, uuid=7433F1DE-C8AF-7D42-8A58-996671E0753D, name="reserved"
/dev/sda11 : start=          64, size=           1, type=CAB6E88E-ABF3-4102-A07A-D4BB9BE3C1D3, uuid=C147F3D7-07F4-0048-97B5-B4DAC1DF7120, name="RWFW"
/dev/sda12 : start=      102400, size=      131072, type=C12A7328-F81F-11D2-BA4B-00A0C93EC93B, uuid=FF81FED5-A756-3C44-9693-3E41EE823552, name="EFI-SYSTEM", attrs="LegacyBIOSBootable"
root@debian:/mnt# 
```

### WindowsやmacOS用の空き領域の確保

次のリストは最初に確認した`sfdisk --list /dev/sda`のパーティション構成を、Startの順番つまり物理的順番に並び変え、説明しやすいようにIDにパーティションの順番を追加してあります。

```
ID : Device        Start       End   Sectors   Size Type
 1 : /dev/sda6        65        65         1   512B ChromeOS kernel
 2 : /dev/sda7        66        66         1   512B ChromeOS root fs
 3 : /dev/sda9        67        67         1   512B ChromeOS reserved
 4 : /dev/sda2        69     32836     32768    16M ChromeOS kernel
 5 : /dev/sda11       64        64         1   512B unknown
 6 : /dev/sda10       68        68         1   512B ChromeOS reserved
 7 : /dev/sda4     32837     65604     32768    16M ChromeOS kernel
 8 : /dev/sda8     69632    102399     32768    16M Linux filesystem
 9 : /dev/sda12   102400    233471    131072    64M EFI System
10 : /dev/sda5    233472   8622079   8388608     4G ChromeOS root fs
11 : /dev/sda3   8622080  17010687   8388608     4G ChromeOS root fs
12 : /dev/sda1  17010688 234441599 217430912 103.7G Linux filesystem
```

現状ではディスクの全領域がChromeOS Flexに割り当てられているので、このままでは他のOSをインストールするための空き領域がありません。そこで注目するのが12番目にあるLinux filesystemのパーティションで、物理的には最後ですが論理的には`/dev/sda1`が示すように最初に割り当てられています。このパーティションがChromeOS Flexのユーザーデータ用の領域です。

**実はユーザーデータ用の領域のファイルシステムはEXT4で、サイズを変更してEXT4を作り直しても問題なくChoromOS Flexは立ち上がります。つまりこの領域を縮小することで他のOS用のスペースを確保します。**これが最初に書いた裏技ということになります。もちろんEXT4を縮小して作り直すと書き込み済みのデータを失いますが、この時点では使い込んでいるわけでは無いので問題ないわけです。


### EFIシステムパーティションの修正

ここで注目するのは9番目にあるEFIシステムパーティション(ESP)で、UEFIではESPにOSのブートコードが書き込まれるます。ChromOS Flexでは、ESPは64Mバイト割り当てられていて、9番目なのにデバイス名の`/dev/sda12`が示すように論理的には12番目ということになり、この中では論理的には一番最後ということになります。
物理的には9番目のパーティションということになります。



```
root@debian:/mnt# uuidgen
8C678F1E-729F-484B-8368-92DF1332E62A
root@debian:/mnt# cp p1-sda.dump p2-sda.dump
root@debian:/mnt# vi p2-sda.dump
.....(省略).....
root@debian:/mnt#
```

```
root@debian:/mnt# sfdisk /dev/sda < p2-sda.dump
.....(省略).....
root@debian:/mnt# 
```

```
root@debian:/mnt# sfdisk --list /dev/sda
Disk /dev/sda: 111.79 GiB, 120034123776 bytes, 234441648 sectors
Disk model: INTEL SSDSC2BW12
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: gpt
Disk identifier: AC161E76-BF4B-924D-9C72-06CE3C6EABCF

Device        Start      End  Sectors  Size Type
/dev/sda1  17010688 50565119 33554432   16G Linux filesystem
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
/dev/sda13 50565120 51089407   524288  256M Microsoft basic data

Partition table entries are not in disk order.
root@debian:/mnt# 
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

## ESPの内容をコピーする
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