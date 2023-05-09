# ChromeOS FlexとWindowsやmacOS等のマルチブート環境を作る

## はじめに

ChromOS Flexは1台のPCでの他のOSとのマルチブートをサポートしていません。実際にChromOS Flexをインストールしたことのある人なら分かりますが、別のOSが動いているPCにChromOS Flexをインストールすると問答無用で既存のOSの領域まで含めてすべてのストレージをChromeOS Flexに割り当ててしまい、ストレージ使用量を調整する手段は用意されていません。ですから通常の方法では他のOSを同じストレージにインストールするためのストレージエリアを確保できないわけです。

この記事は裏技的な手法を使って、1台のPCにChromeOS Flexと他のOSとのマルチブート環境を構築する方法について説明します。あくまでも裏技なので将来にわたってこの方法でChromeOS Flexとのマルチブート環境が作れるかどうかはわかりませんが、当面は問題無いでしょう。

## お勧めしません

いきなりですが、**ChromeOS Flexと他のOSのデュアルブート環境を作るのはお勧めとは言えません**。理由は簡単でChromeOS Flexとのマルチブート環境のPCでWindowsやmacOSからシステムストレージに対する何らかの操作を行うと、ChromeOS Flexのパーティションテーブルを破壊する為です。これはChromeOS Flexのパーティションテーブルが、一般的な構成と違って少々トリッキーになっていることによります。もし破壊されてもパーティションテーブルのバックアップがあれば復帰できるのですが、あれ？っと思ったときに速やかに対処を行うセンスが要求されます。

# このセクションは書き直し必要

この記事を読めばそんな時に対処できるノウハウその方法を解説するので、本記事が理解できる人であれば困ることは無いと思います。とは言えhromeOS Flexを試す程度であれば、ChromeOS FlexのインストールUSBメモリでも十分試せます。また別にPCを用意するのであれば単独で動くようにしたほうが簡単なのは間違いありません。

## 用意するもの

ChromeOS Flexとのマルチブート環境を作るのに必要なものは次の通りです。

1. UEFIをサポートしたPCまたはMac(Intel CPU)本体
1. ChromeOS Flexのインストール用USBメモリ(8GB以上)
1. Windows 10か11、またはmacOSのインストール用USBメモリ(Windowsであれば8GB以上、macOSでは16GB以上)
1. Debian Liveを書き込んだUSBメモリ(2GB以上)
1. 記録用のUSBメモリ(100KB以上の空きがあるもの)

UEFIをサポートしていないPCでもChromeOS Flex自体は起動できますが、この記事では他OSとのブートの切り替えにUEFIで用意されているブートセレクタを使うことを前提としているため、GPT(Guid Partition Table)を扱えるUEFI対応のPCが必須となります。PC本体がIntel CPUのMacであれば条件を満たしています。

もちろん対象PCでChromeOS FLexが動くことを事前に確認しておく必要があります。[ChromeOS Flexの認定モデル](https://support.google.com/chromeosflex/answer/11513094 "認定モデルリスト")に該当していれば文句なしですが、そうでない場合は事前にChromeOS FlexのインストールUSBメモリを使って動作確認を行うのが良いでしょう。

# ココ

macOS の起動可能なインストーラを作成する
windows ダウンロード

マルチブート環境を作るのですからインストール用のUSBメモリがChromeOS FlexとWindowsまたはmacOSのものが必要なのは当然として、パーティションテーブルを編集するためにDebian LiveのUSBメモリも必要になります。Debian Liveでなくても良いのですがシェルが起動できてsfdiskとエディタ等が利用できるLinuxの起動用USBメモリが条件となります。

この他に記録用のUSBメモリも必要になります。Debian LiveのUSBメモリに記録できれば済むのですが、Debian Liveではファイルシステムの肝心な部分はRead Onlyでマウントされているため書き込むことができません。パーティションテーブルのテキストファイルを数個保存するだけなので100Kバイトもあれば間に合います。手頃なものを用意してください。

## ChromeOS Flexのインストール

最初にChromeOS Flexをインストールするのですが、インストール前にBIOSの設定を確認します。少し古いPCではUEFIに対応したBIOSであってもUEFIでの起動が無効になっていることがあるので、必ず確認してください。Macの場合は事前に設定することはありません。

ChromOS Flexのインストールに関しては、インストール用のUSBメモリを作成し、それで起動してメニューにしたがって順に操作するだけなので難しいところは無いでしょう。Googleも[ChromOS Flexインストールガイド](https://support.google.com/chromeosflex/answer/11552529)を用意していて、インストール用のUSBメモリの作り方などもこちらにあります。

通常であればインストール後に起動してアカウント設定等を行うのですが、設定してもこの後の作業で消去されてしまい改めて設定することになります。とは言えChromeOS Flexが動作することを確認しておくことは重要なので、ある程度のことは試しておくのが良いでしょう。

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

## Debianでの事前準備

`sudo -i`でrootユーザーになります。

```
user@debian:~$ sudo -i
root@debian:~# 
```

後ほどmkdosfsコマンドを使うため、aptを使ってdosfstoolsをインストールします。

```
root@debian:~# apt update
root@debian:~# apt install dosfstools
root@debian:~# 
```

記録用のUSBメモリをマウントして、マウントポイントにcdします。これでUSBメモリにコマンドの出力結果などを残せるようになります。

```
root@debian:~# mount /dev/sdc1 /mnt
root@debian:~# cd /mnt
root@debian:/mnt# 
```

## ChromeOS Flexの不思議なパーティションテーブル

最初に`sfdisk --list`を使って、ChromeOS Flexのパーティションテーブルを確認します。この記事で使っているPCの内蔵ストレージは120GBのSSDです。また、後ほど比較するためにp1-sda-listというファイルに結果を保存しています。

```
root@debian:/mnt# sfdisk --list /dev/sda | tee p1-sda-list
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

パーティションテーブルを見たことのある人なら違和感を覚えると思いますが、ChromeOS Flexでは12ものパーティションがあり、更にパーティションの論理的順番と物理的順番が一致していません。たとえばsda1のスタートセクタは170010688ですが、次のsda2の69より大きくなっています。通常パーティションを作成する場合先頭から順に割り当てるので、`sfdisk --list`で見ればスタートセクタの値は小さいものから順に並びます。ところがどういうわけかChromeOS Flexではこのようにバラバラの順番でパーティションが並んでいます。そのため`Partition table entries are not in disk order.`というメッセージも表示されています。

## ChromOS Flexのパーティションテーブルのバックアップ

パーティションの物理と論理の順番が一致していないことが、この後WindowsやmacOSをインストールした後に問題になります。そのため後で利用できるようChromeOS Flexインストール直後のパーティションテーブルを保存します。ここでは`sfdisk --dump`でパーティションの構成を`p1-sda-dump`というファイルに保存しています。

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

GPTの各パーティションエントリはこのリストからわかるように、先頭のLBA、サイズ(実際に記録されているのはパーティションの最後のLBA)、パーティションのタイプを示すUUID(GUID)、各パーティション固有のUUID、パーティションの名前、アトリビュートから構成されています。パーティションのインデックス(論理的な順番)は、各エントリが並んでいる順番で、1から最大128までとなっています。

## WindowsやmacOS用の空き領域の確保

次のリストは最初に確認した`sfdisk --list /dev/sda`のパーティションテーブルをスタートセクタの小さい順つまり物理順に並び変え、説明のためパーティションの順番をIDとして追加してあります。

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

現状ではストレージの全領域がChromeOS Flexに割り当てられているので、このままでは他のOSをインストールするための空き領域がありません。そこでポイントとなるのが12番目にあるLinux filesystemのパーティションです。このパーティションはChromeOS Flexでのユーザーデータ用に割り当てられていてで、物理的には最後ですが論理的には`/dev/sda1`が示すように最初に割り当てられています。

実は**ユーザーデータ用のパーティションのファイルシステムはEXT4で、サイズを変更してEXT4を作り直しても問題なくChoromOS Flexが立ち上がることを確認しています。そこでこの領域を縮小することで他のOS用のスペースを確保します**。もちろんEXT4を縮小して作り直すと書き込み済みのデータを失いますが、この時点では保存しなければならないようなデータは無いので問題ないわけです。これが最初に書いた裏技ということになります。

## EFIシステムパーティションの変更

次にポイントとなるのが9番目にあるEFIシステムパーティション(ESP)で、UEFIではESPにOSのブートローダーやカーネルなどが保存されます。ChromOS FlexではESPに64Mバイト割り当てられていて、9番目なのにデバイス名の`/dev/sda12`が示すように論理的には12番目で一番最後になります。

いろいろテストした結果ChromeOS FlexのESPの設定には二つの問題がありました。一つ目が64MBの容量で、ChromeOS Flexだけなら問題ないのでしょうがWindowsとのマルチブートを行おうとすると残容量が不足気味です。実際Windowsのブートローダーと必要なファイルをインストールしてみると残りは2.7MB程度になりました(2023年4月時点)。もし今後OSのアップデート等によってESPに保存するファイルが増えたり何かのファイルサイズが大きくなったりすると容量不足に陥ることが容易に予測できます。そのため将来に備えて容量を増やす必要があります。

二つ目は論理と物理の順番が一致していないChromeOS Flexならでは問題で、macOS、WindowsともESPのストレージ上の配置は論理と物理の順番が一致したところにないといけません。

そこで、ESPをCheromOS Flexのパーティションの一番最後に移動します。論理的順番はすでに12で一番最後となっていますから、物理的にも12番目になるよう変更するわけです。移動させるといってもストレージ上のパーティションですから、一旦容量の大きなESP用パーティションを作成し、元のESPの内容をコピーしたのち削除します。

## パーティションテーブルの編集 その1

最初のパーティションテーブルの編集は、ChromeOS Flexのユーザーデータ用パーティションを縮小し、それによって空いた領域の先頭部分にESPを置き換えるためのFATのパーティションを作成します。

まず、先ほど保存したp1-sda-dumpを別のファイルにコピーします。そしてFAT用パーティションを作るためにuuidを1個作成します。

```
root@debian:/mnt# cp p1-sda-dump p2-sda-dump
root@debian:/mnt# uuidgen
f2b1b3fc-81da-4ef8-9494-32dd9c0b20a0
```

次にテキストエディタを使って、コピーしたp2-sda-dumpを編集します。元のp1-sda-dumpと編集後のp2-sda-dumpの差分は次のようになります。

```
root@debian:/mnt# vi p2-sda-dump
.....(省略).....
root@debian:/mnt# diff -u p1-sda-dump p2-sda-dump
--- p1-sda-dump 2023-04-08 22:51:03.041333000 +0900
+++ p2-sda-dump 2023-04-08 22:51:03.042949000 +0900
@@ -8,3 +8,3 @@

-/dev/sda1 : start=    17010688, size=   217430912, type=0FC63DAF-8483-4772-8E79-3D69D8477DE4, uuid=08FD7E32-17D4-7341-934F-06FE44B1237F, name="STATE"
+/dev/sda1 : start=    17010688, size=    33554432, type=0FC63DAF-8483-4772-8E79-3D69D8477DE4, uuid=08FD7E32-17D4-7341-934F-06FE44B1237F, name="STATE"
 /dev/sda2 : start=          69, size=       32768, type=FE3A2A5D-4F32-41A7-B725-ACCC3285A309, uuid=A8FC6629-5055-A040-A9A5-415A8EA50C1B, name="KERN-A", attrs="GUID:48,56"
@@ -20 +20,2 @@
 /dev/sda12 : start=      102400, size=      131072, type=C12A7328-F81F-11D2-BA4B-00A0C93EC93B, uuid=FF81FED5-A756-3C44-9693-3E41EE823552, name="EFI-SYSTEM", attrs="LegacyBIOSBootable"
+/dev/sda13 : start=    50565120, size=      524288, type=EBD0A0A2-B9E5-4433-87C0-68B6B72699C7, uuid=f2b1b3fc-81da-4ef8-9494-32dd9c0b20a0, name="DOS"
root@debian:/mnt# 
```

/dev/sda1の変更は、単純にサイズを小さくするだけです。元は217430912ブロック(1ブロックは512バイト)と100GB以上あるわけで、16GBに変更するため33554432に設定しています。ChromeOS Flexにどのぐらいの容量を割り当てるのかは使い方によって変わりますが、自分としてはChromeOS Flexのローカールストレージはほとんど使用しないので16GBで十分という判断です。

次に13番目のパーティションとして、ESPを置き換えるためのFATパーティションを用意します(ESPのファイルシステムはFAT16やFAT32)。sda13の始まりのLBAはsda1のスタートとサイズを加えたものになるので、`17010688 + 33554432 = 50565120`となります。ESPの容量は最低でも100MB程度は必要と思われますので、ここでは256MBを設定しています。次にこのパーティションのtypeを示すGUIDですが、Windowsの一般的なデータパーティションを示す`EBD0A0A2-B9E5-4433-87C0-68B6B72699C7`を指定します。そしてuuidには先ほど用意した`f2b1b3fc-81da-4ef8-9494-32dd9c0b20a0`を設定します。パーティションの名前は無くてもかまわないのですが、ここでは"DOS"とつけてあります。

なお新規にパーティションを用意する場合、startとsizeはかならず8の倍数になるよう設定します。現在のストレージでは、物理セクタサイズに4096バイトを採用しているものが一般的であるため、ブロックサイズの512で割った値が8となります。8の倍数にそろっていない場合、物理セクタとアクセスするセクタが合わないことになって、パフォーマンス面でペナルティが発生するので注意します。

p2-sda-dumpの変更内容が確認できたら、sfdiskコマンドを使ってストレージに書き込みます。

```
root@debian:/mnt# sfdisk /dev/sda < p2-sda-dump
```

変更後のパーティションテーブルは次のようになります。

```
root@debian:/mnt# sfdisk --list /dev/sda | tee p2-sda-list
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

次にsda1とsda13にファイルシステムを作成します。sda1はEXT4ですからmkfs.ext4コマンドを使います。

```
root@debian:/mnt# mkfs.ext4 /dev/sda1
mke2fs 1.46.2 (28-Feb-2021)
/dev/sda1 contains a ext4 file system labelled 'H-STATE'
        last mounted on /mnt/stateful_partition on Fri Mar 17 04:05:56 2023
Proceed anyway? (y,N) y
Discarding device blocks: done
Creating filesystem with 4194304 4k blocks and 1048576 inodes
Filesystem UUID: 5e407efa-1a3a-41f7-9d04-b036bb893ff6
Superblock backups stored on blocks:
        32768, 98304, 163840, 229376, 294912, 819200, 884736, 1605632, 2654208,
        4096000

Allocating group tables: done
Writing inode tables: done
Creating journal (32768 blocks): done
Writing superblocks and filesystem accounting information: done
root@debian:/mnt#
```

sda13はESP用ですからFAT32で作成します。

```
root@debian:/mnt# mkdosfs -F 32 -n EFI-SYSTEM /dev/sda13
mkfs.fat 4.2 (2021-01-31)
root@debian:/mnt# 
```

## ESPの内容のコピー

パーティションの準備ができたので、ESP(sda12)の内容をsda13にコピーします。ここでは双方のパーティションをマウントしてからtarを使ってコピーしていますが、コピーに関してはcp -rなど他の方法でもかまいません。コピーが完了したら、sda12とsda13をアンマウントします。

```
root@debian:/mnt# mkdir /mnt/efi /mnt/dos
root@debian:/mnt# mount /dev/sda12 /mnt/efi
root@debian:/mnt# mount /dev/sda13 /mnt/dos
root@debian:/mnt# (cd /mnt/efi ; tar cf - * ) | (cd /mnt/dos ; tar xvf -)
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
root@debian:/mnt# umount /mnt/dos
root@debian:/mnt# umount /mnt/efi
```

## ESPの置き換え

sda13に新たなESPが用意できたので、既存のsda12を削除してsda13の領域をsda12のESPに変更します。しかしそれだけでは元のsda12の領域(sda8とsda5の間)に64MBの空きができてしまいます。Windowsの場合インストール時に16MBの予約パーティションを作るようで、ここが空いたままでは予約パーティションがsda8の次に作成されてしまいます。そうなると論理と物理の順番を12で一致させたESPが13になってずれてしまいますので、それを防ぐためにsda8の領域をsda5の手前まで拡張します。

ということで今度はp2-sda.dumpをp3-sda-dumpにコピーして、p3-sda-dumpを編集します。

```
root@debian:/mnt# cp p2-sda-dump p3-sda-dump
root@debian:/mnt# vi p3-sda-dump
```

編集内容は次のようになります。

```
root@debian:/mnt# diff -U0 p2-sda-dump p3-sda-dump
--- p2-sda-dump 2023-04-08 22:51:03.042949000 +0900
+++ p3-sda-dump 2023-04-23 21:43:45.434028000 +0900
@@ -16 +16 @@
-/dev/sda8 : start=       69632, size=       32768, type=0FC63DAF-8483-4772-8E79-3D69D8477DE4, uuid=9E8B7A51-B372-5B44-B4D4-EB4ABE5F77CF, name="OEM"
+/dev/sda8 : start=       69632, size=      163840, type=0FC63DAF-8483-4772-8E79-3D69D8477DE4, uuid=9E8B7A51-B372-5B44-B4D4-EB4ABE5F77CF, name="OEM"
@@ -20,2 +20 @@
-/dev/sda12 : start=      102400, size=      131072, type=C12A7328-F81F-11D2-BA4B-00A0C93EC93B, uuid=FF81FED5-A756-3C44-9693-3E41EE823552, name="EFI-SYSTEM", attrs="LegacyBIOSBootable"
-/dev/sda13 : start=    50565120, size=      524288, type=EBD0A0A2-B9E5-4433-87C0-68B6B72699C7, uuid=f2b1b3fc-81da-4ef8-9494-32dd9c0b20a0, name="DOS"
+/dev/sda12 : start=    50565120, size=      524288, type=C12A7328-F81F-11D2-BA4B-00A0C93EC93B, uuid=FF81FED5-A756-3C44-9693-3E41EE823552, name="EFI-SYSTEM", attrs="LegacyBIOSBootable"
root@debian:/mnt#
```

sda8は、サイズを以前のsda8のサイズにsda12のサイズを加えた163840ブロック(80MB)に変更しています。sda12はtypeとuuidはそのままで、sda13のstartと
sizeの値に変更します。そしてsda13は不要なので単純に削除します。

以上の変更が正しくできてるのを確認したら、再びsfdiskコマンドでパーティションテーブルを書き換えます。

```
root@debian:/mnt# sfdisk /dev/sda < p3-sda-dump
```

変更後のパーティションテーブルは次のようになります。

```
root@debian:/mnt# sfdisk --list /dev/sda | tee p3-sda-list
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
/dev/sda8     69632   233471   163840   80M Linux filesystem
/dev/sda9        67       67        1  512B ChromeOS reserved
/dev/sda10       68       68        1  512B ChromeOS reserved
/dev/sda11       64       64        1  512B unknown
/dev/sda12 50565120 51089407   524288  256M EFI System

Partition table entries are not in disk order.
root@debian:/mnt# 
```

最初に保存したp1-sda-listと比較すると次のようになります。

```
root@debian:/mnt# diff -U0 --ignore-space-change p1-sda-list p3-sda-list
--- p1-sda-list 2023-04-08 22:51:03.042130000 +0900
+++ p3-sda-list 2023-04-08 22:51:03.044934000 +0900
@@ -10 +10 @@
-/dev/sda1  17010688 234441599 217430912 103.7G Linux filesystem
+/dev/sda1  17010688 50565119 33554432   16G Linux filesystem
@@ -17 +17 @@
-/dev/sda8     69632    102399     32768    16M Linux filesystem
+/dev/sda8     69632   233471   163840   80M Linux filesystem
@@ -21 +21 @@
-/dev/sda12   102400    233471    131072    64M EFI System
+/dev/sda12 50565120 51089407   524288  256M EFI System
```

これでWindowsやmacOSなどの他のOSをインストールする準備ができたので、PCをシャットダウンします。Debian LiveのUSBメモリとパーティションテーブルのメモをとった記録用のUSBメモリは後の作業で使用するのでそのまま保管してください。

```
root@debian:/mnt# poweroff
```

この後PCの電源を入れると、ChromeOS Flexの起動を確認できます。もし起動しなかった場合はどこかの手順に誤りがあったことになりますので、その場合ChromeOS Flexのインストールからやり直すか、Debian Liveでのパーティションテーブルの修正が必要になります。

## WindowsまたはmacOSのインストール

今度はWindowsやmacOSのインストールメディアを使ってPCを起動し、対象OSをインストールします。インストール時に注意するのは、今までの作業で用意したChromeOS Flexのパーティションを壊さないよう、ストレージの空き領域に正しくインストールすることです。あとは通常のmacOS、Windowsのインストールとなんら変わりありません。

インストールが終了したら、それぞれのOSが動くことを確認します。この状態では**WindowsやmacOS等インストールしたOSは起動しますが、ChromeOS Flexは起動できなくなっています**。理由はWindowsやmacOSがインストールのために新たなパーティションを作成すると、ChromeOS Flexのパーティションテーブルを書き換えてしまうためです。

## Debian Liveの2回目の起動

今度は起動しなくなっているChromeOS Flexのパーティションテーブルの修復です。再びDebian LiveのUSBメモリで起動し、前の作業でメモをとったUSBメモリを/mntにマウントします。なおaptコマンドを使うことはありません。

```
user@debian:~$ sudo -i
root@debian:~# mount /dev/sdc1 /mnt
root@debian:~# cd /mnt
root@debian:/mnt# 
```

起動しなくなったパーティションテーブルの状況を`sfdisk --list`で確認します。

```
root@debian:/mnt# sfdisk --list /dev/sda | tee p4-sda-list
Disk /dev/sda: 111.79 GiB, 120034123776 bytes, 234441648 sectors
Disk model: INTEL SSDSC2BW12
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: gpt
Disk identifier: AC161E76-BF4B-924D-9C72-06CE3C6EABCF

Device        Start       End   Sectors  Size Type
/dev/sda1        64        64         1  512B unknown
/dev/sda2        65        65         1  512B ChromeOS kernel
/dev/sda3        66        66         1  512B ChromeOS root fs
/dev/sda4        67        67         1  512B ChromeOS reserved
/dev/sda5        68        68         1  512B ChromeOS reserved
/dev/sda6        69     32836     32768   16M ChromeOS kernel
/dev/sda7     32837     65604     32768   16M ChromeOS kernel
/dev/sda8     69632    233471    163840   80M Linux filesystem
/dev/sda9    233472   8622079   8388608    4G ChromeOS root fs
/dev/sda10  8622080  17010687   8388608    4G ChromeOS root fs
/dev/sda11 17010688  50565119  33554432   16G Linux filesystem
/dev/sda12 50565120  51089407    524288  256M EFI System
/dev/sda13 51089408  51122175     32768   16M Microsoft reserved
/dev/sda14 51122176 234440703 183318528 87.4G Microsoft basic data
```

sda13とsda14がWindowsのインストールによって追加されたパーティションですが、さらにsda15も増えている場合があります。macOSの場合はsda13のみの1つです。

sda12は事前にパーティションインデックスと物理的な順番が一致するように対処してあったので変化はありませんが、sda1からsda11までが物理的な順にパーティションインデックスが変更されて綺麗にならんでしまっています。ようするにWindowsやmacOSでは、新規のパーティションを追加するためにパーティションテーブルを書き換える際、パーティションインデックスを物理順に並べ替えてしまいChromeOS Flexが起動できなくなっているわけです。

そこで、パーティションのインデックスを元のChromeOS Flexの順番に戻せば、ChromeOS Flexが起動できるようになります。

まず`sfdisk --dump`で現状のパーティションテーブルをp4-sda-dumpに保存します。

```
root@debian:/mnt# sfdisk --dump /dev/sda > p4-sda-dump
```

p4-sda-dumpを、前に保存したp3-sda-dumpと比べると、パーティションの大きさやtype, uuidの情報は変わっていないことがわかります。そこで今回追加されたsda13, sda14(あるいはsda15まで、Macではsda13のみ)のパーティション情報をp3-sda-dumpに追加してストレージに反映すれば、ChromeOS Flexのパーティションテーブルを修復できます。

修復するためのパーティションテーブルを作るのは、エディタを使うこともなく次のコマンドラインで完了します。

```
root@debian:/mnt# cp p3-sda-dump p5-sda-dump
root@debian:/mnt# tail -n 2 p4-sda-dump >> p5-sda-dump
root@debian:/mnt# diff -U1  p3-sda-dump p5-sda-dump
--- p3-sda-dump 2023-04-26 12:30:22.078533000 +0900
+++ p5-sda-dump 2023-04-26 12:30:22.078899000 +0900
@@ -20 +20,3 @@
 /dev/sda12 : start=    50565120, size=      524288, type=C12A7328-F81F-11D2-BA4B-00A0C93EC93B, uuid=FF81FED5-A756-3C44-9693-3E41EE823552, name="EFI-SYSTEM", attrs="LegacyBIOSBootable"
+/dev/sda13 : start=    51089408, size=       32768, type=E3C9E316-0B5C-4DB8-817D-F92DF00215AE, uuid=46FFCC48-8858-4FF6-BE06-E352C55E9FD1, name="Microsoft reserved partition", attrs="GUID:63"
+/dev/sda14 : start=    51122176, size=   183318528, type=EBD0A0A2-B9E5-4433-87C0-68B6B72699C7, uuid=2C6B3DFE-8DF4-4BF1-8C1F-01173A1042E1, name="Basic data partition"
```

 p3-sda-dumpと比較してsda13以降のパーティション情報が追加できていれば問題ありません。確認できたら修正したパーティションテーブルをストレージに反映します。

```
root@debian:/mnt# sfdisk /dev/sda < p5-sda-dump
```

以上でChromeOS Flexと他OSのマルチブート環境の設定は完了です。

Windows PCの場合は、起動時にUEFIの起動選択を選ぶとこんな感じで「ChromeOF Flex」と「Windows Boot Manager」が選べると思います。

Macの場合はOptionキーを押しながら起動すれば次の選択画面が表示されます。このうち「EFI Boot」がChromeOS Flexとなります。

## ChromeOS Flexってどうなの？

ChromeOS Flexってどうなの？と聞かれることが有りますが、WindowsやmacOSがストレスなく動くPCであればそれらを使った方が便利なのは間違いありません。しかし実際にChromeOS Flexを使ってみると内部で余計なアプリが動かないこともあって、動作は非常に軽く起動や終了の速さは特筆ものです。おかげで少々非力なPCであっても快適に利用できます。またWebを中心とした今時の作業であれば、オンラインミーティングやWord, Excelファイルの編集など様々なアプリケーションを動作させられるため、一般的な利用であればほとんど困らないのでは無いかと思います。
