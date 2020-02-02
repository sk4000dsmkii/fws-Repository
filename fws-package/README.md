# 技術書

## 1.はじめに
　近年では、IaC(Infrastracture As Code)が常識の時代になってきている。IaCとは開発したコードをマニュアルなどに書き出しておいて、誰がそのコードを書いても同じような動作をするといった冪等性を保つといった概念である。 その中で、クラウドサービスを利用することが当たり前の時代となってきている。 代表的なクラウドサービスの例としてAWS(Amazon Web Service)がある。 必要なリソースを入力すれば仕組みを理解していなくても誰でも利用することが可能である。 利用するだけであれば問題ないが、このような概念が存在する中、システムの仕組みを理解していないために先ほど記載したようなマニュアルなどの構築を行うことが不可能になってしまい、ブラックボックスとなってしまうのである。

 この技術書は最小クラウドサービスのホスト基盤構築を行ってもらうことで、クラウドサービスの裏側の仕組みを理解してもらうことを目的とする。

## 2.既存のシステムについて
クラウドサービスの仕組みを理解するうえで、既存のVMWareやHyperVなどが存在するが商用化されているため、裏側のコードを閲覧することが不可能である。またOpenStackやProxmoxも存在するが、コード量が多すぎるため理解をすることが難しい。したがって学習用には不向きである。

## 3.VM貸し出しサービスの構成

 VM貸し出しサービスの構成は以下の図のようになっている。

 動画フローとしては、
 ユーザが学内PCからWebコントロールパネルにアクセス　⇩

 VMに必要なCPU Core数、RAM容量、OSを入力してもらい送信　⇩

 管理サーバに値が受け渡されデータベースに保存され、ホストサーバにコマンドが送られる　⇩

 受け取ったリソースをもとにVMが作成される　⇩

 コントロールパネルに表示されたコマンドをユーザーが実行してvncでの操作が可能となる



![Photo](https://i.imgur.com/GOHbHsO.png)

# qemu-system-x86_64を用いたVM貸し出しサービスのホスト基盤構築手順

  ここからはqemu-kvmを用いたVM貸し出しサービスのホスト基盤の構築を行ってもらう。
  この演習は実際に自分でVM(仮想マシン)を構築し、VM貸し出しサービスのホスト基盤部分の裏側の仕組みがどのように動いているのかを理解することが目的である。

  今回演習に用いるPCの仕様、OSは以下の通りである。

## ホスト & ゲストに用いるOS

  Debian-live-10.0.0-amd64-gnome

## 構築演習用PC仕様

    メーカー:Hewlett-Packard(Hp)

    型番:XL51DAV

    CPU:Intel i3-2120 3.30GHz (2Core/4Thread)

    メモリ:DDR3 4GB(2GBx2)

    HDD:250GB

### CPUについて

CPUはコア数とスレッド数によって並列処理を行える数が変わってくる、そのためコア数とスレッド数が増えるほど処理の速度が向上する。
しかしアプリケーションによって対応しているコア数などが決まっているため、一概にすべての処理が必ずしも向上するとは限らない。

### RAM(メモリ)について

RAMは容量によって処理ができる速度が決まる。容量が増えるほど処理速度が向上するが、CPUとOSのbit数によって認識できるメモリの容量が決まっているため、注意が必要である。


## 大まかな流れについて

### ※注意  ここからはVMを動かす大元であるPC(XL51DAV)をホストマシン、ホストマシンの上で動かすVMのことをゲストマシンと呼ぶ。
      
### また、コマンドを打つ際に

```
$ 
```
### と書いてある場合はユーザ権限


```
# 
```
### と書いてある場合はroot権限であるため注意すること

### ※コピペは禁止です

構築の流れは以下の通りである。

  1.BIOS設定でIntelVirtialization Technology(IntelVT)の有効化を行う。

  2.ホストマシンにOSのインストールを行う。

  3.ホストマシンにVM構築の際に必要なソフトウェアをインストールを行う。

  4.ホストマシンのネットワーク設定を行う。

  5.ゲストマシンを作成しゲストマシンにOSをインストールする。

## 1.BIOS設定でIntel Virtialization Technology(Intel VT)を有効化する。

### ========================================

### Intel Virtialization Technology(Intel VT)とは

  IntelVTとはインテルによって開発された仮想化支援技術のことである。
  この機能を有効にしなければ、KVM(Kernel-based Virtual Machine)を利用することができない。

### KVM(Kernel-based Virtual Machine)とは

  KVMとはLinuxカーネルをハイパーバイザとして機能させるための仮想化モジュールである。

### ハイパーバイザとは

  ハイパーバイザーとはコンピュータを仮想化し、1台の物理サーバ上で複数の仮想的なコンピュータを動かせるようにする、制御ソフトウェアのことである。

### ========================================

  したがってBIOS設定でIntelVTを有効にしなければ、KVMを利用することができなく、ハイパーバイザーも使用することが不可能なため、この機能を有効にしなくてはならない。

   今回の演習ではXL51DAVのBIOSを例に説明を行う。
   まず電源を起動した際に、F10を押しBIOSに入る。

   BIOSの設定項目の

       Security -> System Security -> 【Virtialization Technology (VTx)】

  を選択。

   デフォルトでは【Disabled】になっているため、これを【Enabled】に変更し【F10(Accept)】し、【Save Change and Exit】でホストマシンを再起動する。

   ここまでで、IntelVTの設定は終了である。次はホストマシンにOSのインストールを行う。

## 2.ホストマシンにOSをインストールする。

  今回はLinuxディストリビューションのひとつであるDebianを用いる。
  LinuxとはUnix系オペレーティングシステムカーネルであり、OSS(Open Sorce Software)のひとつである。

   ホストマシンにDebian-live-10.0.0-amd64-gnomeをインストールする。インストール画面に沿って各種設定をしていく。
   
  ## ホスト名とユーザ名
  ### ユーザ名
  ```
  test-user
  ```
  
  ### パスワード
  ```
  test1234
  ```
とする
 Debianのインストールについては[こちら](https://github.com/nsrg-fmlorg/students-2019/blob/master/VirtualMachineTeam/qemu-kvm/DebianInstallManual_(qemu.ver).md)を参照する。

インストールが完了したらホストマシンに設定をしていく。

### ※ここからの作業はすべてroot権限で行う
 ```
 $ su 
 ```
 passwdを入力

 $が#に変化したのを確認し、設定が終わったら次の項目に進む

## 3.ホストマシンにゲストマシン構築に必要なソフトウェアをインストールする。

### 3-1.ホストマシンアプリケーションをインストールする。

インストールするソフトウェア以下のとおりである。

 【qemu】,【qemu-kvm】,【bridge-utils】,【gvncviewer】

#### ========================================

 #### qemuとは
 ホストPC上で動作する、仮想マシンのエミュレータである。 またQEMUはCPUだけではなく、各種の周辺ハードウェアもエミュレートしていることが特徴である

 #### qemu-kvmとは
 KVMについてははじめの方で解説したので省略する。KVM単体では動作しないが、先ほどのqemuと組み合わせることで高速に完全仮想化を実現することができる。

 #### bridge-utilsとは
 ブリッジを管理、構築するのに必要なソフトウェアである。ブリッジは仮想ネットワークスイッチのように動作する。ブリッジには実際のデバイス (例: eno1)と仮想デバイス (例: tap0) を接続することが可能である。

 #### gvncviewerとは
 ネットワークを通じて別のコンピュータに接続し、そのデスクトップ画面を呼び出して操作することができるリモートデスクトップソフトの一つである。今回はこのソフトを利用してVMの画面にアクセスする。

#### ========================================

以上の4つをインストールするため、以下のコマンドを実行する。

    # apt-get install qemu qemu-kvm bridge-utils gvncviewer autoconf ssh


 次は仮想ブリッジ構築のためのネットワーク設定を行う。

## 4.ホストマシンのネットワーク設定

### 4-1.ホストマシンのresolv.confファイルを書き換えてDNSサーバのアドレスを設定する
以下のディレクトリのファイルを編集し、viを用いて書き換えを行う。

    # vi /etc/resolv.conf

  resolv.conf

    nameserver 指定のDNSを入力

#### ========================================

#### DNSサーバとは

  ネットワーク上で ドメイン名を管理・運用するために開発されたシステムであり、ドメイン名とIPアドレスを対応づけるしくみのことである

#### ========================================

設定を完了したら次は仮想ブリッジを構築するためのネットワークインターフェースの設定を行う

### 4-2.　仮想ブリッジを構築するためにホストPC内の/etc/network/interfacesを書き換える


#### ========================================

#### 仮想ブリッジ

ブリッジとは複数のネットワーク間を直接接続する方式である。仮想ブリッジはホストLinux上に仮想的なL2スイッチを構成する機能であり、複数の仮想ブリッジを構成することも可能である。

#### ネットワークタップ

ネットワークタップとは、ネットワーク上の信号を分岐させて取り出すものである。イーサネットのネットワーク信号をネットワークに影響を与える事なく分岐をすることが可能である

ゲストOSはこの仮想ブリッジにタップで繋がることでネットワーク接続を行うことが可能になる。

#### ========================================

以下のディレクトリのファイルを編集し、viを用いて書き換えを行う。

    # vi /etc/network/interfaces

  interfaces

    source /etc/network/interfaces.d/*
    auto lo
    iface lo inet loopback
    iface eno1 inet manual
    auto br0
    iface br0 inet static
    address ipアドレスを入力
    netmask サブネットマスクを入力
    gateway ゲートウェイを入力
    dns-nameservers DNSを入力
    bridge_ports eno1 all
    bridge_maxwait 0
    bridge_df 0
    bridge_stp off



### 4-3.　変更したネットワーク設定を反映させるために、以下のコマンドを実行する

    # /sbin/ifup eno1 && /sbin/ifup br0

## 5.fws_packageをmake install する。

### 5-1. fws_packageの展開

ない場合はこちらからダウンロードする(通常はもう置いてあります)
[fws_package_v1.2](https://github.com/nsrg-fmlorg/students-2019/raw/master/VirtualMachineTeam/fws_package/fws_hostpackage_v1.2.zip)

zipを展開し、autoconfコマンドとmake installコマンドを使用しパッケージをインストールする。

    # cd /home/test-user/

    # unzip fws_hostpackage_v1.2

    # cd fws_package_v1.2

    # autoconf

    # ./configure

    # make install

以上を行うとパッケージが/usr/local/fws/　配下にインストールされる。
パッケージに入っているスクリプトはそれぞれ以下の通りである。


# ※コード解説

### 00_vmDiskCreate.sh

```
#!/bin/sh

disk_dir="/usr/local/fws/vmDiskImages/" #ディスク格納ディレクトリの指定

echo "ディスク名(ホスト名)を入力してください:"

read diskname                           #ディスク名の入力

echo "ディスク容量を入力してください(GB単位)"
read diskcapacity                       #ディスク容量の入力

qemu-img create -f qcow2 ${disk_dir}${diskname}.qcow2 ${diskcapacity}G

                                        #qemu-imgコマンドを用いて指定したディレクトリに仮想ディスクイメージファイルの作成

if [ -f $disk_dir${diskname}.qcow2 ]; then #ディスクが作成されたかどうかの確認

    echo "ディスクイメージが作成されました。"
    ls -l $disk_dir

else
    echo "Error"
fi
```
#### === ゲストマシン 仮想ディスクイメージ作成スクリプト ===
 
 00_vmDiskCreate.shは仮想ディスクイメージの作成スクリプトである。
 ディスク名とディスク容量を決めるとそのリソースをもとに、仮想ディスクイメージが作成される。
 次以降のインストールスクリプトや起動スクリプトなどで、この仮想ディスクイメージは使用される。
 
 仮想ディスクイメージとは簡単に言うと仮想のハードディスクである。この仮想のハードディスクを補助記憶装置として、ゲストOSはOSをインストール、データの読み書きをする。


### 01_vmInstallStart.sh

```
#!/bin/sh

disk_dir=/usr/local/fws/vmDiskImages/ #ディスク格納ディレクトリの指定
image_dir=/usr/local/fws/installImages/   #インストールイメージディレクトリの指定
database=/usr/local/fws/db/database       #データベースディレクトリの指定

echo "ディスクイメージを入力してください(Not Input .qcow2)"
ls -l $disk_dir                    #指定したディレクトリ内にある仮想ディスクイメージの閲覧

read qcowimage                     #ディスクイメージ名の入力

echo "VNC番号を選択してください"
read vncnumber                     #VNC番号の入力

echo "インストールイメージを選択してください(Not Input .iso)"
echo "=============================="
ls -l $image_dir                   #指定したディレクトリ内にあるインストールイメージの閲覧
echo "=============================="
read installimage                  #インストールしたいイメージ名の入力

echo "仮想CPUのコア数を設定してください"
read cpucore                       #仮想CPUのコア数の入力

echo "メモリ容量を入力してください"
read raminput                      #仮想メモリの容量の入力

macaddr=$(python 03_vm-macaddr-hasher.py ${qcowimage}${vncnumber}) #pythonのコードを用いてホスト名からMACアドレスの生成

dataset="$disk_dir $vncnumber $cpucore $raminput $macaddr $installimage"
echo $dataset >> $database                                  # 入力したデータをデータベースに保存


qemu-system-x86_64 -enable-kvm \  
-machine type=pc,accel=kvm \                      #kvmの有効化
-vga std -nographic -vnc :$vncnumber \　           #-vncオプションを使用するために-nographicsオプションを使用
-hda ${disk_dir}${qcowimage}.qcow2 \               #-hda 仮想ディスクイメージの指定
-cdrom ${image_dir}${installimage}.iso \　         #インストールイメージの指定
-boot once=d \                                     #ブート方式の指定 once=dは最初の起動だけ-cdromを読み込む
-m $raminput \                                     #仮想メモリのサイズ指定
-cpu host \                                        #qemuにホストPCのCPUを正確にエミュレートさせるためのオプション
-smp $cpucore \                                    #仮想CPUのコア数の指定
-k ja \                                            #キーボード配列の指定
-net nic,macaddr=$macaddr -net tap,ifname=vtap${vncnumber} &
                                                   #仮想NICにMACアドレスの設定、tapモードを使用したブリッジへの接続設定
```
#### === ゲストマシン OSインストールスクリプト ===
 
 01_vmInstallStart.shはOSインストールスクリプトである。
 先ほど作成したディスクイメージにOSをインストールするために使われる。ディスクイメージ名、vnc番号、インストールするOS、仮想CPUのコア数、仮想メモリの容量をそれぞれ決めるとディスクイメージ名と、vnc番号を用いてMACアドレスが割り振られ、ゲストマシンが立ち上がる。
リモートアクセスするにはvncviewerアプリケーションを用いる。

### 02_vmStart.sh

```
#!/bin/sh

disk_dir=/usr/local/fws/vmDiskImages/      #ディスク格納ディレクトリの指定

echo "ディスクイメージの場所を入力してください"
ls -l $disk_dir                                #指定したディレクトリ内にある仮想ディスクイメージの閲覧
read qcowimage                                 #ディスクイメージ名の入力

echo "VNC番号を選択してください"
read vncnumber                                 #VNC番号の入力

echo "仮想CPUのコア数を設定してください"
read cpucore                                   #仮想CPUのコア数の入力

echo "メモリ容量を入力してください"
read raminput                                  #仮想メモリの容量の入力

macaddr=$(python 03_vm-macaddr-hasher.py ${qcowimage}${vncnumber})

qemu-system-x86_64 -enable-kvm \
-machine type=pc,accel=kvm \                   #ここまでkvmの有効化
-vga std -nographic \
-vnc :$vncnumber \                             #vncオプションを使用するために-nographicsオプションを使用
-hda ${disk_dir}${qcowimage}.qcow2 \           #インストールイメージの指定
-boot c \                                      #ブート方式の指定
-m $raminput \                                 #仮想メモリのサイズ指定
-smp $cpucore \                                #仮想CPUのコア数の指定
-k ja \                                            #キーボード配列の指定
-net nic,macaddr=$macaddr -net tap,ifname=vtap${vncnumber}
                                               #仮想NICにMACアドレスの設定、tapモードを使用したブリッジへの接続設定
```
#### === ゲストマシン 起動スクリプト ===

### 03_vm-macaddr-hasher.py

02_vmStart.shは作成したゲストマシンを起動させるスクリプトである。ゲストマシンをシャットダウンした場合や、なにかしらのバグなどで
マシンの電源が落ちてしまった場合はこのスクリプトを用いて起動をさせる。
起動方法は基本はインストールの時と同じであり、ディスクイメージ名、vnc番号、仮想CPUのコア数、仮想メモリの容量をそれぞれ決めるとディスクイメージ名と、vnc番号を用いてMACアドレスが割り振られ、ゲストマシンが立ち上がる。

```
#!/usr/bin/env python

import sys
import zlib

if len(sys.argv) != 2:
        print("usage: %s <diskname>" % sys.argv[0])
        sys.exit(1)

crc = zlib.crc32(sys.argv[1].encode("UTF-8")) & 0xffffffff
crc = str(hex(crc))[2:]
print("52:54:%s%s:%s%s:%s%s:%s%s" % tuple(crc))

```


## 5-3.　ゲストマシンの仮想イメージファイルの作成

1.ターミナルで/usr/local/fws/srcに移動する。

    # cd /usr/local/fws/src

2.仮想ディスクイメージの作成

例:仮想ディスクの容量が30GBのゲストマシンを作成する場合は、以下のコマンドを用いて先ほど作成したスクリプトを実行する。

  作成するストレージイメージファイルは「test」とする。

        # sh 00_vmDiskCreate.sh

実行すると指示が出るので、指示に従って値を入力していく。

#### ディスク名:test ディスク容量:30

完了すると

```
ディスクイメージが作成されました。

-rw-r--r-- 1 root   rooot   197088 日付  test.qcow2

```
と表示され、正しく作成できていない場合は
```
error
```
と表示される。
エラーの場合はもう一度最初からやり直してください。

### 5-4.　ゲストマシンにOSをインストールする。

#### ※インストールを行う前に事前にインストール用のiso(Debian、CentOS、Ubuntu...etc)を準備してください。isoをダウンロートしてくたら、下記のコマンドを使用して先ほどインストールしたディレクトリにコピーしてください。
 
 ```
 例
 cp /home/test-user/ダウンロード/Debian10.iso /usr/local/fws/installImages/ 
 ```

先ほど作った仮想ディスクイメージにOSをインストールし、ゲストマシンを立ち上げる。
以下のコマンドを用いて、実行する。

        # sh 01_vmInstallStart.sh
        
実行すると命令が出力されるので、指示に従って値を入力していく。

※今回は仮想コア数2core、メモリ2048MB、vnc番号を1とする。

### ディスクイメージ:test VNC番号:1 インストールイメージ:Debian10 仮想CPUのコア数:2 メモリ容量:2048

完了するとqemuが起動する。

### 5-5.別ターミナルからgvncviewerを起動し、先程立ち上げたゲストマシンにホストマシンからアクセスし手順に沿ってインストールを進める。

#### VNCとは

VNCとは、ネットワークに接続しているコンピュータをリモート操作するソフトウェアのことである。オリジナルのソースコードは公開されており、オリジナルのほかにも様々な派生バージョンが公開されている。代表例としてRealVNC、TigerVNC、noVNCなどがある。

#### 操作中のPCからアクセスする場合

    # gvncviewer 127.0.0.1:1
    
    ※ipアドレスの後ろの:xはvnc番号である

#### 他PCからアクセスする場合

    # gvncviewer 10.129.xxx.xxx:1
    
    ※ipアドレスの後ろの:xはvnc番号である

Debianのインストールについては[こちら](https://github.com/nsrg-fmlorg/students-2019/blob/master/VirtualMachineTeam/qemu-kvm/DebianInstallManual_(qemu.ver).md)


  #### ※既存のゲストOSを起動する場合

  以下のコマンドを用いて、実行する。

    # 02_vmStart.sh
    
実行すると画面が出力されるので、以前設定した値を画面の指示に従って入力していく。

先ほど作成したtestを起動する場合

### ディスクイメージ:test VNC番号:1 仮想CPUのコア数:2 メモリ容量:2048

### 5-6.別ターミナルからgvncviewerを起動し、先程立ち上げたゲストマシンにホストマシンからアクセスし起動を確認する。

    # gvncviewer localhost:1

他ローカルPCからアクセスする場合

    # gvncviewer 10.129.xxx.xxx:1

インストールが完了し、再起動後にOSが起動したらゲストマシン構築の完了とする。
以上で演習を終了する。

終了後もう一度アンケートに答えてもらうので、終了したら川村を呼んでください。


### コマンド集とその意味

<details><summary>qemu-system-x86_64 のオプション</summary><div>


-enable-kvm ...KVMの有効

-machine type=pc,accel=kvm ...KVMを使用する

-vga std -nographic -vnc :$vncnumber ...VNCの設定

-hda ...ストレージの指定

-cdrom ...メディアの指定

-boot once=d ...ブート方式の設定,once=dの他にもc,dなどがある

-m ...RAMの容量設定

-cpu host ...仮想CPUの安定動作

-smp ...仮想CPUのコア数の設定

-netdev user,id=${image}.$vncnumber -device e1000,netdev=${image}.$vncnumber ...Tapインタフェースの設定

-net nic,macaddr -net tap,ifname=vtap0  ...NIC,Tapの設定,MACアドレスの設定

</div></details>

### 参考文献

[qemu-system-x86_64 コマンド集](https://wiki.archlinux.jp/index.php/QEMU)
