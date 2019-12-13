# Mirakurun + EPGStation のインストール・設定

## 前提条件
前提として MySQL がインストールされているものとする(EPGStation で利用).


## Node.js のインストール
```bash
$ curl -sL https://deb.nodesource.com/setup_10.x | sudo -E bash -
$ sudo apt install nodejs
```

## Mirakurun のインストール
```bash
## pm2 とMirakurun のインストール
$ sudo npm install pm2 -g
$ sudo npm install mirakurun -g --unsafe --production

## pm2 にMirakurun の自動起動設定を登録
$ sudo mirakurun init
```

## Mirakurun の設定
```bash
## mirakurun コマンドでファイルを開く場合
$ sudo EDITOR=emacs mirakurun config server    # サーバの設定
$ sudo EDITOR=emacs mirakurun config tuners    # チューナの設定
$ sudo EDITOR=emacs mirakurun config channels　# チャンネルの設定

## 直接ファイルパスを開く場合
$ sudo emacs -nw /usr/local/etc/mirakurun/server.yml    # サーバの設定
$ sudo emacs -nw /usr/local/etc/mirakurun/tuners.yml    # チューナの設定
$ sudo emacs -nw /usr/local/etc/mirakurun/channels.yml　# チャンネルの設定
```

## チューナーの設定
```bash
## チューナーの設定 (emacs)
$ sudo EDITOR=emacs mirakurun config tuners

## チューナーの設定を見るだけ
$ cat /usr/local/etc/mirakurun/tuners.yml
```
```yaml
- name: PT3-S1
  types:
    - BS
    - CS
  command: recpt1 --b25 --device /dev/pt3video0 --lnb 15 <channel> - -
  decoder: ~
  isDisabled: false

- name: PT3-S2
  types:
    - BS
    - CS
  command: recpt1 --b25 --device /dev/pt3video1 --lnb 15 <channel> - -
  decoder: ~
  isDisabled: false

- name: PT3-T1
  types:
    - GR
  command: recpt1 --b25 --device /dev/pt3video2 <channel> - -
  decoder: ~
  isDisabled: false

- name: PT3-T2
  types:
    - GR
  command: recpt1 --b25 --device /dev/pt3video3 <channel> - -
  decoder: ~
  isDisabled: false
```

## チャンネル設定 (外部サイト)
Mirakuru のチャンネル設定は上述したように `/usr/local/etc/mirakurun/channels.yml` を編集することになる.
このファイルを自動で設定させる方法が以下の記事に丁寧にまとめられているのでここでは割愛する.

https://www.jifu-labo.net/2019/02/mirakurn_channels_config/


## EPGStation のインストール

### DB の設定 (mysql)
root ユーザでログインして, データベースを作成する.

```bash
## mysql の root ユーザでログイン
$ mysql -u root -p

## DB を作成し, epgstation ユーザを作成する.
mysql> CREATE DATABASE epgstation CHARACTER SET utf8;
mysql> GRANT ALL ON epgstation.* TO epgstation@localhost IDENTIFIED BY 'epgstation';
mysql> GRANT ALL ON epgstation.* TO epgstation@127.0.0.1 IDENTIFIED BY 'epgstation';
```

### EPGStation のインストール
基本的には公式のマニュアルに従う.

https://github.com/l3tnun/EPGStation/blob/master/doc/linux-setup.md

ただ, ハマりどころがいくつかあったので, メモを残しておく.

#### 1. EPGStation のインストール
```bash
$ cd ~/opt/
$ git clone https://github.com/l3tnun/EPGStation.git
$ cd EPGStation
$ npm install
$ npm run build
```

#### 2. 設定ファイルの作成
```bash
$ cp config/config.sample.json config/config.json
$ cp config/operatorLogConfig.sample.json config/operatorLogConfig.json
$ cp config/serviceLogConfig.sample.json config/serviceLogConfig.json
```

#### 3. 設定ファイルの編集
`config/config.sample.json` と `config/config.json` の差分を以下に示す.
```diff
--- config/config.sample.json   2019-12-13 08:57:02.207801030 +0900
+++ config/config.json  2019-12-13 16:18:09.761417467 +0900
@@ -1,8 +1,21 @@
 {
     "readOnlyOnce": false,
     "serverPort": 8888,
-    "mirakurunPath": "http+unix://%2Fvar%2Frun%2Fmirakurun.sock/",
-    "dbType": "sqlite3",
+       "mirakurunPath": "http://localhost:40772",
+
+    "dbType": "mysql",
+    "streamFilePath": "/tmp",
+    "recorded": "/home/video",
+    "mysql": {
+        "host": "localhost",
+        "port": 3306,
+        "user": "epgstation",
+        "password": "epgstation",
+        "database": "epgstation",
+        "connectTimeout": 20000,
+        "connectionLimit": 10
+    },
+
     "ffmpeg": "/usr/local/bin/ffmpeg",
     "ffprobe": "/usr/local/bin/ffprobe",
     "maxEncode": 2,
```

## EPGStation の起動 / 終了
- 自動で起動する場合
	- [pm2](http://pm2.keymetrics.io/) を利用して自動起動設定が可能.
	- 初回のみ以下の起動設定が必要.

	```
	$ sudo npm install pm2 -g
	$ sudo pm2 start dist/server/index.js --name "epgstation"
	$ sudo pm2 save
	```
- 自動起動した EPGStation を終了する場合
	```
	$ sudo pm2 stop epgstation
	```
- 状態を確認する方法
   ```
   sudo pm2 status
   ```
## MySQL 使用時の注意

EPGStation 使用中は MySQL のバイナリログが大量に生成されてディスクを圧迫するので, MySQL の設定を変えることを推奨.

```
expire_logs_days = 1
```

