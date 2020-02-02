# Debianのインストール手順の解説

## 1.インストーラが起動したら【Grapchical Debian Installer】を選択し、Enter.

![Grapchical Debian Installer](https://i.imgur.com/oA1qJhA.jpg)

## 2.言語の選択

### 【日本語】を選択し、Enter.(Jキーを押すと日本語に項目に飛ぶことができます。)

![Grapchical Debian Installer](https://i.imgur.com/EAv4Q3l.jpg)

## 3.在住国の選択

### 【日本】を選択し、Enter.

![Grapchical Debian Installer](https://i.imgur.com/I63vZ80.jpg)

## 4.キーボードのキー配列の選択

### 【日本語】を選択し、Enter.

![Grapchical Debian Installer](https://i.imgur.com/7hZ4kTH.jpg)

## 5.ネットワークインターフェースカードの設定

### 搭載されているNIC(Network Interface Card)が表示されます。ここでは【ens3】を選択し、Enter.

![Grapchical Debian Installer](https://i.imgur.com/zYqW6wl.jpg)

## 6.ネットワークの設定

### 6-1.自動設定に失敗しましたと言われるので、【続ける】を選択するか、Enter.

![Grapchical Debian Installer](https://i.imgur.com/phSfi0C.jpg)

### 6-2.【ネットワークを手動で設定】を選択し、Enter.

![Grapchical Debian Installer](https://i.imgur.com/gUpV9S7.jpg)

### 6-3.IPアドレスを入力し、Enter.

![Grapchical Debian Installer](https://i.imgur.com/4WEmH8E.jpg)

### 6-4.サブネットマスクを入力し、Enter.

![Grapchical Debian Installer](https://i.imgur.com/f8xEf5V.jpg)

### 6-5.デフォルトゲートウェイを入力し、Enter.

![Grapchical Debian Installer](https://i.imgur.com/TIgChqr.jpg)

### 6-6.ネームサーバーアドレスを入力し、Enter.

![Grapchical Debian Installer](https://i.imgur.com/C41Fi9I.jpg)

### 6-7.ホスト名を入力(事前に決められたものを使用する)し、Enter.

![Grapchical Debian Installer](https://i.imgur.com/RwzX0vw.jpg)

### 6-8.ドメイン名を入力し、Enter.

![Grapchical Debian Installer](https://i.imgur.com/FeukGPx.jpg)

## 7.root(管理者)用のパスワードを設定

### rootが使用するパスワードを入力し、確認の項目の方にもう一度入力し、Enter.

![Grapchical Debian Installer](https://i.imgur.com/XufernA.jpg)

## 8.使用するユーザ名を入力

### 8-1.自分ということがわかるユーザのフルネームを入力

![Grapchical Debian Installer](https://i.imgur.com/HhHbGa9.jpg)

### 8-2.先ほどのユーザアカウントのユーザ名を入力

![Grapchical Debian Installer](https://i.imgur.com/XZQfX0l.jpg)

### 8-3.先ほどのユーザ用のパスワードを設定

![Grapchical Debian Installer](https://i.imgur.com/xlvYlBP.jpg)

## 9.ディスクのパーティショニング

### 9-1.自分が利用するストレージのパーティションを構成します。

今回は【ガイド - ディスク全体を使う】を選択し、Enter.

![Grapchical Debian Installer](https://i.imgur.com/QwAUSzg.jpg)

### 9-2.使用するストレージの選択

今回は【SCSI1(0,0,0)(sda) - 21.5GB ATA QEMU HARDDISK】を選択し、Enter

![Grapchical Debian Installer](https://i.imgur.com/NCBUU96.jpg)

### 9-3.パーティション分割方式の選択

今回は【すべてのファイルを一つのパーティションに】を選択し、Enter.

![Grapchical Debian Installer](https://i.imgur.com/egp3qlw.jpg)


※詳細に/home,/var,/tmpを設定したい場合は【/home,/var,/tmpを分割】を選択し、Enter.

![Grapchical Debian Installer](https://i.imgur.com/JzKsHdU.jpg)

### 9-4.パーティショニングの確認

以下のように表示がされたら【パーティショニングの終了とディスクへの変更の書き込み】を選択し、Enter.

※　注意: ここは解説をきちんと読んでからEnterを押すこと！ここでは間違えてEnterを押しても次が最終確認のため、パーティショニングを引き返すことができすが次はできません。

![Grapchical Debian Installer](https://i.imgur.com/d3IAOgf.jpg)

### 9-4.パーティショニングの最終確認

先ほど設定したパーティショニングの最終確認です。値があっていれば【はい】にチェックをいれて、Enter.

※　注意: ここは解説をきちんと読んでからEnterを押すこと！ここで間違えるとパーティショニングは引き返すことができません。

![Grapchical Debian Installer](https://i.imgur.com/ivAScsn.jpg)

## 10.パッケージマネージャの設定

### 10-1.ネットワークミラーを使うので【はい】を選択し、Enter.

![Grapchical Debian Installer](https://i.imgur.com/eRmc9fg.jpg)

### 10-2.アーカイブミラーの【日本】を選択し、Enter

![Grapchical Debian Installer](https://i.imgur.com/kaj1NbF.jpg)

### 10-3.アーカイブミラーのリンク【ftp.jp.debian.org】を選択し、Enter.

![Grapchical Debian Installer](https://i.imgur.com/Tgpjanp.jpg)

### 10-4.httpプロキシの設定、今回は空白でEnter.

![Grapchical Debian Installer](https://i.imgur.com/1BqmTSg.jpg)

## 11.ブートローダーのインストール

### 11-1.マスターブートレコードにGRUBブートローダーをインストールする。【はい】を選択し、Enter.

![Grapchical Debian Installer](https://i.imgur.com/H0H3njk.jpg)

### 11-2.どのハードディスクへGRUBブートローダーを入れるかを選択、今回は【/dev/sda】

![Grapchical Debian Installer](https://i.imgur.com/jXEnmCD.jpg)


## 12.正常であればインストールが完了

### 【続ける】を選択し、インストールを終了する。

![Grapchical Debian Installer](https://i.imgur.com/UV7a4aS.jpg)


## 再起動後、ブートローダが起動

![Grapchical Debian Installer](https://i.imgur.com/abKqNJO.jpg)

### 【＊Debian GNU/Linux】を選択するとブートが始まり、正常にインストールが成功して入ればユーザ認証画面が表示されます。

先ほど設定したパスワードを入力すると以下のような画面になり、インストールが完了しております。これでDebianを使う準備ができました。

お疲れさまでした。

![Grapchical Debian Installer](https://i.imgur.com/uq8N6C6.jpg)









