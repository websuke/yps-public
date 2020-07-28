# EC2インスタンスにlaravelをインストール & VSCodeでのSSH接続まで

##  MySQL5.7のインストール・設定<br><br>

1. まずはEC2にログインしているか確認
```bash
whoami
```
***
2. centosでログインできていれば、下記コマンドでインストール実施
```bash
sudo yum localinstall http://dev.mysql.com/get/mysql57-community-release-el7-7.noarch.rpm
```
```bash
yum install -y mysql-community-server
```
***
3. インストールできたか確認
```bash
mysqld --version
```
- 下記が表示されればOK

        mysqld  Ver 5.7.31 for Linux on x86_64 (MySQL Community Server (GPL))

***

4. 自動起動設定
```bash
sudo systemctl enable mysqld
```
***
5. エラーが表示されないことを確認しながら以下3つを確認
```bash
sudo systemctl start mysqld
```
```bash
sudo systemctl status mysqld
```
```bash
sudo systemctl stop mysqld
```
***

6. 初期パスワードの取得を行う<br>
grepコマンドを使用してパスワード部分を rootで絞り込み検索をかけている。
```bash
sudo cat /var/log/mysqld.log | grep -i root
```
- xxxxxxxxx部分が初期パスワード

        root@localhost: xxxxxxxxx
***

7. MySQL起動
```bash
sudo systemctl start mysqld
```
***

8. 先程取得した初期パスワードを使用してログイン<br>
```bash
mysql -u root -p
```
パスワードを要求されるので、先程取得した初期パスワードを貼り付けてEnter
***

9. パスワードの変更を行う<br>
※ 任意のパスワード部分を8文字以上で英大文字・小文字・数字・記号を混ぜて入力 <br><br>
(※の要件に1つでも当てはまらなければエラーになる。#については場合によってはコメントアウト扱いされて思わぬところでエラーになったりするのでNG)
```bash
SET PASSWORD = PASSWORD('任意のパスワード');
```

***
10. Mysqlからログアウト
```bash
exit
```
***

11. 設定ファイル編集
```bash
sudo vi /etc/my.cnf
```

- viエディタでファイルの**最後尾**に下記を追記

character-set-server=utf8mb4

今の所↓<br>
~~[client]~~<br>
~~default-character-set=utf8mb4~~

***

12. Mysql再起動
```bash
sudo systemctl restart mysqld
```
***
13. Mysql再ログイン(先程設定したパスワードを打ち込む)
```bash
mysql -u root -p
```
***
14. 以下を打ち込んでutf8mb4が表示されるのを確認<br>
latin1とかの記載があってはだめ！！！
utf8は今の所OK！！！

(本来手順11で消してある記述を記載して全てutf8mb4としたい所だが現在ターミナルから日本語が入力できなくなるためyotaroさんが対策を考えてくださってる途中なので皆さん回答待ちましょう)→[yotaroさんのgithubはこちら](https://github.com/yotaro-ok/yps)
```bash
show variables like "chara%";
```
***
1. Mysqlからログアウト
```bash
exit
```
***

## Nginx1.8のインストール・設定<br><br>
16. Nginxの設定ファイル作成
```bash
sudo vi /etc/yum.repos.d/nginx.repo
```
- 以下を記述して保存終了

        [nginx]
        name=nginx repo
        baseurl=https://nginx.org/packages/centos/$releasever/$basearch/
        gpgcheck=0
        enabled=1

***
17.nginxのインストール
```bash
sudo yum install nginx -y
nginx -v
```
- 下記が表示されればOK

        nginx version: nginx/1.18.0
***
18. 自動起動設定
```bash
sudo systemctl enable nginx
```
***
19. AWS (ブラウザ側) に戻ってルールの追加を選択してhttpとhttpsを追加
***
20.ターミナルに戻ってnginxの再起動
```bash
sudo systemctl start nginx
```
***
21. そーするとブラウザからアクセスできるようになる。
URLはAWSの IPv4 パブリック IPではなく**パブリック DNS (IPv4)部分**

そしてブラウザにアクセスした際 **Welcome to nginx!** となってればOK
***
22. またもやターミナルに戻って下記を入力
```bash
sudo mkdir /var/www
sudo mkdir /var/www/html
sudo chown -R centos:nginx /var/www
```
***
23. nginxの設定ファイル編集のバックアップファイル作成
```bash
sudo cp /etc/nginx/nginx.conf /etc/nginx/nginx.conf.org
```
***
24. 元ファイルである nginx.conf を vi で編集
```bash
sudo vi /etc/nginx/nginx.conf
```
一旦何もせずに終了
***
25. 多分23. 24.は飛ばしてOKでこれがちゃんとしたバックアップファイル作成をしている所
```bash
sudo cp /etc/nginx/conf.d/default.conf /etc/nginx/conf.d/default.conf.org
```
***
26. 元ファイルである default.conf を vi で編集
```bash
sudo vi /etc/nginx/conf.d/default.conf
```
- 下記のように変更して保存終了

        server_name  パブリック DNS (IPv4);

        #root   /usr/share/nginx/html;
        root   /var/www/html;
***

27. nginxを再起動
```bash
sudo systemctl restart nginx
```
ブラウザでリロードして403が表示されればOK
***

CentOS 7 の標準の yum リポジトリでは PHP 5.4 が提供されているが、
新しくサーバーを構築する際にはもっと新しいバージョンの PHP をインストールしたい場合があり今回はPHP 7.3.20をインストールしたいので以下続く<br><br><br>
28. エンタープライズLinux用の、高品質パッケージ EPL を追加
```bash
sudo yum install epel-release
sudo yum update
```
***
29. Remi リポジトリの追加<br>
※Remi で提供されているソフトウェアをインストールするためには EPEL のリポジトリも必須要件
```bash
sudo yum install https://rpms.remirepo.net/enterprise/remi-release-7.rpm
sudo yum update
```
***

## PHP7.3のインストール、設定<br><br>
30. PHP のインストール
```bash
sudo yum -y install --enablerepo=epel,remi,remi-php73 php php-devel php-mbstring php-pdo php-gd php-xml php-mcrypt php-fpm php-mysql php-mysqlnd
```
```bash
php -v
```
- 以下が表示されればOK

        PHP 7.3.20
***
31. www.confのバックアップとしてwww.conf.orgを作成してwww.confを編集
```bash
sudo cp /etc/php-fpm.d/www.conf /etc/php-fpm.d/www.conf.org
sudo vi /etc/php-fpm.d/www.conf
```
※ www.confファイルのコメントアウトは # ではなく ; なので注意
- 以下のように編集

        user = nginx       に変更
        group = nginx      に変更

        listen = /var/run/php-fpm/php-fpm.sock      を追記

        listen.owner = nginx                        を追記
        http://listen.group = nginx                 を追記
        listen.mode = 0660                          を追記

***
32. default.confを編集
```bash
sudo vi /etc/nginx/conf.d/default.conf
```
- 以下のようにindex.phpを追加した形に変更

        index  index.html index.htm index.php;

- #error_page  404　　　　/404.html;　　　　の上に下記内容を追記

        location ~ \.php$ {
                fastcgi_pass   unix:/var/run/php-fpm/php-fpm.sock;
                fastcgi_index  index.php;
                fastcgi_param  SCRIPT_FILENAME  $document_root$fastcgi_script_name;
                include        fastcgi_params;
        }

- ここまでで一旦の形は出来てるけど若干の微調整が必要 (手順については以下)

       先程変更した箇所である

        index  index.html index.htm index.php;

        を↓に変更

        index  index.php index.html index.htm;
        try_files $uri $uri/ /index.php?$query_string;

- もういっちょ！

        location ~ \.php$ {
                fastcgi_pass   unix:/var/run/php-fpm/php-fpm.sock;
                fastcgi_index  index.php;
                fastcgi_param  SCRIPT_FILENAME  $document_root$fastcgi_script_name;
                include        fastcgi_params;
        }

        のfastcgi_pass   unix:/var/run/php-fpm/php-fpm.sock;の上に
        root   /var/www/html;を追加


- 手順32.の default.confを編集での編集したあたりの完成形は以下

            server {
            listen       80;
            # server_name  localhost;
            server_name  パブリック DNS (IPv4);


            #charset koi8-r;
            #access_log  /var/log/nginx/host.access.log  main;

            location / {
                # root   /usr/share/nginx/html;
                root   /var/www/html/yps/public;
                index  index.php index.html index.htm;
                try_files $uri $uri/ /index.php?$query_string;
            }

           location ~ \.php$ {
                        root   /var/www/html/yps/public;
                        fastcgi_pass   unix:/var/run/php-fpm/php-fpm.sock;
                        fastcgi_index  index.php;
                        fastcgi_param  SCRIPT_FILENAME  $document_root$fastcgi_script_name;
                        include        fastcgi_params;
            }

            #error_page  404              /404.html;

            # redirect server error pages to the static page /50x.html
            #
            error_page   500 502 503 504  /50x.html;
            location = /50x.html {
                # root   /usr/share/nginx/html;
                root   /var/www/html/yps/public;
            }
***
33. index.phpを編集
```bash
vi /var/www/html/index.php
```
- 以下を追記して保存終了

        <?php
        echo phpinfo();
        ?>
***
## ここから Laravel7のインストール、設定

 ここから41. までは結構曖昧でyotaroさんの指示通りだと下記の①~
①②の通りにすればイントールできる。
でも恐らく更に下の34. から41.の順番でも可能。

        じゃー、次 本丸の laravel インストールすっかー！！

        ①
        cd /tmp
        php -r "copy('https://getcomposer.org/installer', 'composer-setup.php');"
        php composer-setup.php
        php -r "unlink('composer-setup.php');"
        sudo mv composer.phar /usr/local/bin/composer
        sudo chmod +x /usr/local/bin/composer


        ②
        cd /var/www/html
        composer create-project --prefer-dist laravel/laravel yps

        ③
        sudo yum install zip unzip -y


        memory うんたら〜　みたいな英語のエラーが出たら

        ④
        sudo systemctl stop mysqld
        sudo systemctl stop nginx

        ⑤
        sudo -s /bin/dd if=/dev/zero of=/var/swap.1 bs=1M count=1024
        sudo -s /sbin/mkswap /var/swap.1
        sudo -s /sbin/swapon /var/swap.1

        ⑥
        composer create-project --prefer-dist laravel/laravel yps

        ⑦
        cd yps
        cp -p .env.example .env

        ⑧
        php artisan key:generate

        ⑨
        vi .env

        APP_URL=ブラウザのURL
        DB_PASSWORD="手順9.で設定した任意のパスワード"

        で保存、終了

        で　※これで最後ｗ

        ⑩
        vi /etc/nginx/conf.d/default.conf

        の
        /var/www/html;
        を
        /var/www/html/yps/public;
        に変更する

        ①①
        sudo systemctl restart nginx
        sudo systemctl restart php-fpm

        ブラウザをリロードすれば、完了です


34. とりあえずなにをしているのかは不明
```bash
cd /tmp
php -r "copy('https://getcomposer.org/installer', 'composer-setup.php');"
php composer-setup.php
php -r "unlink('composer-setup.php');"
sudo mv composer.phar /usr/local/bin/composer
sudo chmod +x /usr/local/bin/composer
```
***
35. ディレクトリを移動
```bash
cd /var/www/html
```
***
36. Task2-3で使用するためZIPファイル解凍ツールをインストール
```bash
sudo yum install zip unzip -y
```
***
37. 一旦Mysqlとnginxを停止してスワップ領域を確保
```bash
sudo systemctl stop mysqld
sudo systemctl stop nginx

sudo -s /bin/dd if=/dev/zero of=/var/swap.1 bs=1M count=1024
sudo -s /sbin/mkswap /var/swap.1
sudo -s /sbin/swapon /var/swap.1
```
***
38. yps というプロジェクトを作成
```bash
composer create-project --prefer-dist laravel/laravel yps
```
***
39. 作成したypsプロジェクトに移動して.env.exampleをコピーして.envファイルを作成
```bash
cd yps
cp -p .env.example .env
```
***
40. .envファイルのAPP_KEY=　　　ってなってるところを<br>
APP_KEY=生成されたKEYに変える！
```bash
php artisan key:generate
```
```bash
vi .env
```
- 以下のように編集して保存終了

        APP_URL=ブラウザのURL ( httpから始まるパブリック DNS (IPv4))
        DB_PASSWORD="手順9. で設定した任意のパスワード"
***
41. npm を使用するためにnodeをインストールして composer と npm をインストールしている？？

補足
- chown: (change owner) ファイルやディレクトリの所有者を変更するコマンド
- chmod: (change mode) ファイルやディレクトリのアクセス権(パーミッション)を変更するコマンド
```bash
sudo yum install npm node -y

composer install
npm install

sudo chown -R centos:nginx /var/www/
sudo chmod -R 777 storage/ bootstrap/cache/
```
***
42. 最後にdefault.confをもう一度編集
```bash
vi /etc/nginx/conf.d/default.conf
```

- 以下のように変更して保存終了

        /var/www/html;

        を↓の記述に変更

        /var/www/html/yps/public;
***
43. 再起動
```bash
sudo systemctl restart nginx
sudo systemctl restart php-fpm
```
ブラウザをリロードしてlaravelのウェルカムページが表示されればOK
***
44. 試しにちょっと編集をしてみる
```bash
vi resources/views/welcome.blade.php
```
- このファイルがlaravelのウェルカムページのファイルになるので好きなように編集してみて保存終了してブラウザをリロードして変更が反映されるか確認
***
## VSCodeでSSHログイン設定


45. VSCodeの拡張機能「Remote - SSH」をインストール
左側のメニューに、[Remote Explorer] のアイコンが表示されたならOK
***
46. 左側のメニューに、[Remote Explorer] のアイコンを選択してSSH TARGETS の右側にある歯車アイコンを押下
***
47. SSH 接続用の config ファイルを作成
- /Users/ユーザー名/.ssh/config を選択<br><br>
※ etc/ssh/ssh_configでもリモート接続できるみたいだけど違うので注意！！
***
48. configファイル編集

        Host 任意(別に普通に変更できる)
        HostName パブリック DNS (IPv4)
        User centos
        Port カスタムTCP
        IdentityFile /フルパス/XxXXX.pem
***
49. configファイル保存

するとSSH TARGETSのところにconfigファイルのHostに記述した部分の名前でサーバーが追加される。
***
50. 追加されたサーバーにhoverすると右側に出てくるウィンドウマークを押下
(ここで fingerprint の確認が求められることがあるらしいが、
問題なければ continue を選んで接続していいらしい)
- /home/centos/ がデフォルトで入力されているが44. で編集したファイルは
/var/www/html/yps/ 配下のため一旦消して未入力の状態から入力し直す。
***
51. 終了

これでresources/views/welcome.blade.phpを見てみると先程44. で編集していたファイルをVSCodeでも編集ができるようになる。





