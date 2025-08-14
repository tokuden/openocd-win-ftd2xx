[English version](./README.md)
# Welcome to OpenOCD-win-ftd2xx

OpenOCD-win-ftd2xxはOpenOCDを、Windowsで動作するようにビルドしたものです。
オープンソースのlibusbやlibftdiではなく、FTDI社のプロプライエタリなFTD2XXドライバを使って動作するように書き換えました。

## 特徴
* Microsoft Windows上で動く(仮想環境が不要)
* FTDI純正のFTD2XXで動くようにカスタマイズしているため、Zadigでドライバを入れ替える必要がない
* FTDI社のFT232HやFT2232D/H、FT4232HなどのMPSSEを使ったUSB-JTAGで汎用的に使える
* 最新のOpenOCD 0.12をベースにしている

ターゲットは、ZYNQ7000とUltraScale+、Raspberry Pi 4に接続できることを確認しています。
Windowsのネイティブアプリとして動作しますので、WSLなどの仮想環境、コンテナは不要です。
詳しくない人に「WSLを用意してください」と言ってを困らせる心配がありません。

## ダウンロードとインストール
https://github.com/tokuden/openocd-win-ftd2xx/OpenOCD-Win-FTD2XX.zip をダウンロードして解凍してください。

このレポジトリは本家OpenOCDを2025年8月14日にフォークしてきたものなので、様々なファイルがありますが、  
ユーザとして真に実行に必要なものは OpenOCD-Win-FTD2XX.zip に固めてあるものだけです。

![ZIPファイルを解凍したようす](https://github.com/tokuden/openocd-win-ftd2xx/blob/images/folder.png)

## 使い方
MSDOSプロンプトを開いて、

`openocd.exe -f ft2232h.cfg -f zynq7000.cfg`

のように入力します。

用意しているコンフィグファイルは、

* np1167.cfg - 特電NP1167A/Bケーブルを使うインタフェース用コンフィグファイル
* ft2232h.cfg - FT2232HのMPSSEを使う汎用的なインタフェース用コンフィグファイル
* TE0802.cfg - TrenzElectronic製TE0802用のインタフェース用コンフィグファイル
* zynq7000.cfg - ZYNQ7000を使ったターゲット側のコンフィグファイル
* xilinx_zynqmp.cfg - ZYNQ UltraScale+を使ったターゲット側のコンフィグファイル

です。

構成が決まっていたらバッチファイルを作ったほうがいいかと思います。  
例えば、[特電のMPSSE-JTAGケーブル](https://mitoujtag.jp/mpssejtag.html)を使用する場合は、`-f np1167.cfg`と指定してください。

起動すると以下のような画面になります。

![OpenOCDWinの起動画面](https://github.com/tokuden/openocd-win-ftd2xx/blob/images/openocdwin.png)

CPUのデバッグをするにはTELNETでlocalhost:4444にログインしたり、GDBでリモート接続する必要があります。
例えば、物理メモリ0番地の内容をダンプするには、TeraTermを使ってlocalhost:4444にTELNET接続し、

```
targets
halt
mdw phys 0 0x100
redume
```

とコマンドを打ちます。haltさせたらresumeしないとCPUが止まったままになります。

![メモリダンプ](https://github.com/tokuden/openocd-win-ftd2xx/blob/images/dump.png)

## OpenOCD-win-FTD2XXの最大の特徴
なひたふ版OpenOCDの最大の特徴は、**Windows環境で、libusbやWinUSBを使うことなくFTDIの純正デバイスドライバで動く**ということです。

FTDIのデバイスはWindows PCに刺した瞬間にデバイスドライバが自動的にダウンロードされて使用可能になるため、通常の使い方をする限りはデバイスドライバのインストールは不要です。刺したらすぐに使える大変優れたものです。

しかし、OpenOCDはオープンソースのソフトウェアで「GPL」でライセンスされています。
困ったことにGPLが大好きな人たちは「GPLのソフトは美しく正義であるので、邪悪なプロプライエタリ(ソースが公開されていないソフト)をリンクさせてはいけない」という狂った思想によって、FTDIの純正ライブラリを使わずにlibusbやlibftdi1といった有志作成のオープンソースのデバイスドライバを使うようにしてしまいました。OpenOCDからFTDIに関するコードが削除されたのは v0.10.0-rc1 からだと思います。

しかし、Windowsは１つのUSB IDに対して1つのドライバしか使えません。  

そこで、Zadigという危険ツールを使って、FTDIのデバイスドライバの代わりにlibusbを使うようにレジストリを改変してしまうことを推奨しているのです。

zadigを使ってFTDIドライバをlibusbに置き換えてしまうとそのPCではFTDIのドライバが使えなくなるので、FTDIを使う他のアプリが使えなくなってしまうという大きな犠牲を払う必要があったのです。
FTDIドライバが使えなくなった状態を解除するには、デバイスドライバをアンインストールして再インストールする必要がありますが、繰り返しているうちに、PCはだんだんおかしくなってきます。

そもそもOpenOCDがFTDIのドライバを使わないのは、オープンソースとプロプライエタリを混ぜないという理由によるものですが、ユーザにとっては何の利益にもなりません。むしろ毎回のドライバ入れ替えで危険を体験するデメリットしかありません。
そこで、私なひたふは、libusbとlibftdi1を使うように書かれたmpsse.cをFTDI純正ドライバで動くように書き換えました。

## ビルド情報
オリジナルのOpenOCD 0.12から [src/jtag/drivers/mpsse.c](/tokuden/openocd-win-ftd2xx/blob/master/src/jtag/drivers/mpsse.c) だけを改変しています。
configureの書式は

```
./configure --disable-werror --enable-stlink=no --enable-ti-icdi=no --enable-ulink=no --enable-angie=no --enable-usb-blaster-2=no --enable-ft232r=no --enable-vsllink=no --enable-xds110=no --enable-osbdm=no --enable-opendous=no --enable-armjtagew=no --enable-rlink=no --enable-usbprog=no --enable-esp-usb-jtag=no --enable-cmsis-dap-v2=no --enable-cmsis-dap=no --enable-nulink=no --enable-kitprog=no --enable-usb-blaster=no --enable-presto=no --enable-openjtag=no --enable-linuxgpiod=no --enable-dmem=no --enable-sysfsgpio=no --enable-remote-bitbang=no --enable-linuxspidev=no --enable-buspirate=no --enable-dummy=no --enable-vdebug=no --enable-jtag-dpi=no --enable-jtag-vpi=no --enable-rshim=no --enable-xlnx-xvc=no --enable-jlink=no --enable-parport=no --enable-amtjtagaccel=no --enable-ep93xx=no --enable-at91rm9200=no --enable-bcm2835gpio=no --enable-imx-gpio=no --enable-am335xgpio=no LDFLAGS="-lftd2xx"
```
です。  
いろいろオプションがついていますが、不要なケーブルの対応を削除しているだけです。

重要なオプションは `--disable-werror LDFLAGS="-lftd2xx" ` のみです。  
MSYS2 MINGW64環境でコンパイルしています。
