# Task2の補足

インスタンスを停止させるとIPv4 パブリック IPだけでなく**パブリック DNS (IPv4)部分**も変わってしまうので
変更するのが面倒。
インスタンスが1つしか起動していないなら追加料金もないので起動させっぱなしのほうが吉！

もし停止させてしまった場合の手順は以下

- VSCodeのssh/configを新しいパブリック DNS (IPv4)に書き換えるだけで一応繋がるしVSCodeで編集したものが反映もされる。

- でも他にもDNS (IPv4)記載している箇所があるので
なんとなく気持ち悪いので以下の2つの記述も現在起動中のパブリック DNS (IPv4)に合わせる。
# まずはcentosで仮想サーバーにログイン
```bash
ssh -p カスタムTCP centos@<IPアドレス> -i <フルパス>/xxxxx.pem
```
***
1. centosでログインしたらdefault.conf編集
```bash
sudo vi /etc/nginx/conf.d/default.conf
```
- 以下の箇所を修正

        server_name  パブリック DNS (IPv4);
***
2. プロジェクト配下までcdで移動

例)
```bash
cd ../../var/www/html/yps
```
2-1. envファイル編集
```bash
vi .env
```
- 以下のように編集して保存終了(もしかしたらこれは自動で書き換わる。けど一応確認)

        APP_URL=ブラウザのURL

