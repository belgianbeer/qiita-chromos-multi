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
