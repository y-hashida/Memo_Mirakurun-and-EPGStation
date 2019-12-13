# Mirakurun + EPGStation のインストール・設定

## Node.js のインストール
```bash
$ curl -sL https://deb.nodesource.com/setup_10.x | sudo -E bash -
$ sudo apt install nodejs
```

## Mirakurun のインストール
```bash
$ sudo npm install pm2 -g
$ sudo npm install mirakurun -g --unsafe --production
```

## Mirakurun の設定
```bash
$ sudo EDITOR=emacs mirakurun config server    # サーバの設定
$ sudo EDITOR=emacs mirakurun config tuners    # チューナの設定
$ sudo EDITOR=emacs mirakurun config channels　# チャンネルの設定
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