# SQLのサンプルデータダウンロード

このサイトの下部に配布されているデータを直接リモートサーバーにアップする

https://tech.pjin.jp/blog/2016/12/05/sql%e7%b7%b4%e7%bf%92%e5%95%8f%e9%a1%8c-%e4%b8%80%e8%a6%a7%e3%81%be%e3%81%a8%e3%82%81/

1. まずはこれからDB,TABLEのデータをリモートサーバーにアップする。
***
2. centosでEC2にログイン
```bash
 ssh -p <カスタムTCP> centos@<IPv4 パブリック IP>
 -i ~/.ssh/xxx.pem
```
***
3. tmpへ移動
```bash
cd /tmp
```
***
4. yumでwgetをインストール

ターミナルでデータをダウンロードする際には、wgetコマンドを利用すると便利！
わざわざWEBサイト等からデータをダウンロードして、FTPツール等を利用してサーバへデータをアップロードするような作業では手間が省けて便利なコマンドなので覚えておくと良いらしい。
```bash
sudo yum install wget
```
***
5. 先程インストールしたwgetコマンドでFTPを介さずに直接ダウンロード
```bash
wget http://tech.pjin.jp/wp-content/uploads/2016/04/worldcup2014.zip
```
***
6. 以前Task2-1で取得したunzipコマンドで解凍
```bash
unzip worldcup2014.zip
```
***
7. ちゃんと解凍できたかファイルを確認
```bash
ls -la worldcup2014.sql
```
***
8. MySQLにrootでログイン
```bash
mysql -u root -p
```
パスワードを入力
***
9. データベース作成(SQL文実行)
```sql
create database worldcup2014db;
```
***
10. 使用するデータベースを選択
```sql
use worldcup2014db;
```
***
11. ソースファイルを使用してテーブル作成
```sql
source ./worldcup2014.sql;
```
***
12. テーブルの確認
```sql
show tables;
```
***
13. 一旦離脱
```bash
exit
```
***
14. VSCodeでの編集を可能にするための準備を行う。(今回はypsディレクトリへ一旦移動)
```bash
cd /var/www/html/yps/
```
***
15. .envファイルを編集
```bash
vi .env
```
- DB_DATABASE=xxxを下記に変更

        DB_DATABASE=worldcup2014db
***
16. TestCommand.phpを作成(command)
```bash
php artisan make:command TestCommand
```
***
17. 確認
```bash
ls -la app/Console/Commands/TestCommand.php
```
***
18. Player.phpを作成(model)
```bash
php artisan make:model Models/Player
```
***
19. 確認
```bash
ls -la app/Models/Player.php
```
***
20. VSCodeで作成したTestCommand.phpを編集

<編集前>
```php
<?php

namespace App\Console\Commands;

use Illuminate\Console\Command;

class TestCommand extends Command
{
    /**
     * The name and signature of the console command.
     *
     * @var string
     */
    protected $signature = 'command:name';

    /**
     * The console command description.
     *
     * @var string
     */
    protected $description = 'Command description';

    /**
     * Create a new command instance.
     *
     * @return void
     */
    public function __construct()
    {
        parent::__construct();
    }

    /**
     * Execute the console command.
     *
     * @return int
     */
    public function handle()
    {
        return 0;
    }
}

```
<編集後>　全選択してコピペでOK
```php
<?php

namespace App\Console\Commands;

use Illuminate\Console\Command;
use App\Models\Player;

class TestCommand extends Command
{
    /**
     * The name and signature of the console command.
     *
     * @var string
     */
    protected $signature = 'test_command';

    /**
     * The console command description.
     *
     * @var string
     */
    protected $description = 'Test Command';

    /**
     * Create a new command instance.
     *
     * @return void
     */
    public function __construct()
    {
        parent::__construct();
    }

    /**
     * Execute the console command.
     *
     * @return int
     */
    public function handle()
    {
        $players = Player::get();
        foreach($players as $player) {
            echo $player->name."\n";
        }
        return 0;
    }
}
```
***
21. よくわからんがターミナルに戻ってypsディレクトリでclear
```bash
php artisan config:clear
```
***
22. test_command実行
```bash
php artisan test_command
```
***
選手名の一覧が出てくればOK
