# MyCheatSheet

## centos でssh接続
```bash
ssh -p <カスタムTCP> centos@<IPv4 パブリック IP> -i <フルパス>/xxx.pem
```
***
## ログイン後のypsプロジェクトへのパス
```bash
cd /var/www/html/yps
```
***
## サーバー上でPHPを実行する仕組みがphp-fpmでその設定ファイル
- www.conf編集
```bash
sudo vi /etc/php-fpm.d/www.conf
```
***

## nginxのlocation等の設定ファイル
- default.conf編集
```bash
sudo vi /etc/nginx/conf.d/default.conf
```
***

## DB接続情報を確認するためのコマンド

確認したいディレクトリ上で下記コマンド入力
```bash
php artisan tinker
```
<br>

- HOST
```bash
config('database.connections.mysql.host');
```
- 接続先DB
```bash
config('database.connections.mysql.database');
```
- userName
```bash
config('database.connections.mysql.username');
```
- port番号
```bash
config('database.connections.mysql.port');
```
- password
```bash
config('database.connections.mysql.password');
```
***
## 設定の変更方法
```bash
sudo vi /etc/nginx/conf.d/default.conf
```
- 切り替えたい方の root 記載行の # を削除して保存終了

        # Laravel
        # root /var/www/html/yps/public;

        # WordPress
        # root /var/www/html/wwpp;


- 再起動
```bash
sudo systemctl restart php-fpm
sudo systemctl restart nginx
```
