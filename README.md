# linux-centos

  ConoHa VPS で CentOS Stream 9 サーバーを構築！

Reference:
1. ssh  https://knowledge.sakura.ad.jp/8218/#VPSSSH
2. ssh  https://happy-nap.hatenablog.com/entry/2022/10/08/220946
3. dnf  https://atmarkit.itmedia.co.jp/ait/articles/2002/06/news010.html 

## SSH でリモートログイン

### ssh コマンドを使う場合

  次のコマンドを実行して，rootのパスワードを入力する．
```
$ # <hostname> は VPS のIPアドレス or ドメイン名
$ ssh root@<hostname>
```

### PuTTY を使う場合

  PuTTY をインストール．起動して「Host Name (or IP address)」の欄にVPSのIPアドレスを入力して「Open」を押す．
  初回接続時は未知のホストに接続している旨の警告が表示されるので，承認する．
  「login as:」に「root」と入力し，パスワードを聞かれるのでrootに設定したパスワードを入力する．

## パッケージの更新

  次のコマンドを実行してパッケージを更新する．
```
$ dnf upgrade
```

  「dnf」はCentOSの標準のパッケージマネージャー．

## root でSSHログインできないようにする

  次のコマンドを実行してログイン用のユーザを作成する．
```
$ # 新しいユーザを作成する．<username>は任意のユーザ名で置き換える．
$ adduser <username>
$ # パスワードを設定する．
$ passwd <username>
$ # ユーザを管理者に追加する．wheelグループのメンバーはsudoが使える．
$ usermod -G wheel <username>
```

  ssh の設定を編集して root で直接SSHログインできないようにする．
```
$ # バックアップを取る．
$ cp /etc/ssh/sshd_config /etc/ssh/sshd_config.old
$ # 編集内容は下記参照．
$ vim /etc/ssh/sshd_config
$ # sshd を再起動する．
$ systemctl restart sshd.service
```

  `/etc/ssh/sshd_config` の内容は次のように編集する．
```diff
- #PermitRootLogin prohibit-password
+ PermitRootLogin no
```

  以後はrootでなく作成したユーザでログインして作業する．
```
$ su - <username>
```

## 公開鍵認証でSSHログイン

### ssh コマンドを使う場合

  クライアント側で秘密鍵と公開鍵のペアを生成し，公開鍵をサーバーに転送する．
```
$ # 既存の鍵の確認．
$ ls ~/.ssh
$ # `id_ed25519`, `id_ed25519.pub` が存在しないときは新規に鍵を作る．
$ ssh-keygen -t ed25519
$ # ホームディレクトリに移動
$ cd
$ # 公開鍵をサーバー側に転送する．
$ scp .ssh/id_ed25519.pub <username>@<hostname>:
```

  サーバー側で公開鍵を登録する．
```
$ # ~/.ssh がない場合は作成する
$ mkdir ~/.ssh
$ # 転送された公開鍵を追記
$ cat ~/id_ed25519.pub >> ~/.ssh/authorized_keys
$ # パーミッションを修正する
$ chmod 700 ~/.ssh
$ chmod 600 ~/.ssh/authorized_keys
$ # 転送された公開鍵を削除
$ rm ~/id_ed25519.pub
```

  これでクライアント側から ssh コマンドを利用して公開鍵認証でSSHログインできる．
  ユーザのパスワードではなく秘密鍵のパスフレーズを入力します．パスワードをネットワークに送信しないので安全！
```
$ ssh <username>@<hostname>
```

### PuTTY を使う場合

  PuTTYgen を起動する．「Generate」ボタンを押してマウスをグリグリ動かすと鍵が生成される．
  「Public key for pasting into OpenSSH authorized_keys file:」というところにある内容を
  サーバー側の `~/.ssh/authorized_keys` に追記する．
  「Key passphrase」を入力して秘密鍵(.ppk)を保存する．
  秘密鍵はPuTTYを起動して「Connection → SSH → Auth → Credentials」に移動し
  「Private key file for authentication」というところに作成した秘密鍵ファイルを登録する．

  これでPuTTYを利用して公開鍵認証でSSHログインできる．Pageant に秘密鍵を登録しておくとパスフレーズの入力を省ける．

- [firewall](firewall.md)
- [nginx/mariadb/php](nginx-mariadb-php.md)
  
