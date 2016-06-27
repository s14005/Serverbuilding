#Section 2 その他のWebサーバー環境
##2-1 Vagrantを使用したCentOS 7環境の起動
###注意sudoつけていない所がるので自分で必要だと思ったらつけてください
1. vagrant用のCentOSを先生のUSBからコピー
2. vagrant用のBOXをforceする  
   `$vagrant box add CentOS7 コピーしたbox ファイル --force`
3. 作業用のディレクトリを作成してその中でVagrantの初期設定を行います  
   `$vagrant init CentOS7` 
3. ホストオンリーアダプターの設定  
   `$vi Vagrantfile`で開いて  
   Vagrant.configure(2) do |config|  
   これを追加→config.vm.network :private_network,      ip:"192.168.58.129"  
   end  
4. Vagrantを起動  
   `$vagrant up`
5. Vagrantに接続  
   `$vagrant ssh`

##2-2 Wordpressを動かす
##プロキシの設定

1. プロキシの設定  
   $vi /etc/profileファイルを開く  
       ↓を追加  
   PROXY='172.16.40.1:8888'  
   export http_proxy=$PROXY  
   export HTTP_PROXY=$PROXY  
   export HTTPS_PROXY=$PROXY  
   export https_proxy=$PROXY  
2. yumにプロキシを設定  
   `$sudo vi /etc/yum.conf`ファイルを開く  
       ↓を追加  
   proxy=http://172.16.40.1:8888/  
3. wgetにプロキシの設定  
   `$sudo vi /etc/wgetrc`ファイルを開く
    ↓を追加
   http_proxy = 172.16.40.1:8888/
   https_proxy = 172.16.40.1:8888/
   ftp_proxy = 172.16.40.1:8888/
4. yumのアップデート
   `$sudo yum update`

##nginxのインストル

1. nginxのリポジトリのインストール
   `$sudo yum install http://nginx.org/packages/centos/7/noarch/RPMS/nginx-release-centos-7-0.el7.ngx.noarch.rpm`
2. nginxのパッケージのインストール
   `$ sudo yum install --enablerepo=nginx nginx`
3. 自動起動に設定
   `$sudo systemctl enable nginx.service`
4. nginxのサービスを起動
   `$sudo systemctl start nginx`

##PHPのインストール設定

1. PHPと必要なモジュールをインストール
   `$sudo yum -y install php-mysql php php-gd php-mbstring php-fpm`
2. php-fpm設定
   `$sudo vi /etc/php-fpm.d/www.conf`開いて
   user = apache
   group = apach
   変更　↓
   user = nginx
   group = nginx
3. 自動設定に変更
   `$ sudo systemctl enable php-fpm.service`
4. 起動してないので起動
   `$ sudo systemctl start php-fpm.service`

##MariaDBのインストール,設定,作成

1. MariaDBのインストール
   `$yum -y install mariadb mariadb-server`
2. MariaDB自動設定
   `$systemctl enable mariadb`
3. MariaDBのサービスの起動
   `$systemctl start mariadb`
4. 設定 パスワード変更設定以外全部yesでおｋ
   `$mysql_secure_installation`
5. Mariadb再起動
   `$systemctl restart mariadb`
6. 1回終了してrootでログイン
   mysql>exit
   `$mysql -u root -p`
7. wordpressで使うデーターベースを作成する
   mysql> CREATE DATABASE データベース名;
8. ユーザーの作成
   mysql> GRANT ALL PRIVILEGES ON データベース名.* to 'ユーザ 名'@'localhost' IDENTIFIED BY '任意パスワード';

##Wordpressをインストール,設定,立ち上げまで
1. Wordpressをダウンロード
   `$wget http://wordpress.org/latest.tar.gz`
2. 解凍
   `$tar xzfv latest-ja.tar.gz`
3. 所有者とグループをnginxに変更
   `$chown -R nginx:nginx wordpress` 
4. wordpressをディレクトリ/var/www/にいどう
   `$mv wordpress /var/www`
5. default.confの中身を変更
   rootをwordpressを置いた場所に変更
   root   /usr/share/nginx/html;
　      変更 ↓　　　　　↓
   root  /var/www/wordpress/;

   fastcgi_param SCRIPT_FILENAME /scripts$fastcgi_script_name;
       変更 ↓　　　　　↓
　　　fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;

6. nignxを再起動
   `$systemctl restart nginx`
7. wordpressフォルダのwp-config-sample.phpをコピーしてwp-config.phpを作成する
wordpressフォルダの中で`$cp wp-config-sample.php wp-config.php`
8. コピーしたwp-config.phpの中を編集
   `$vi wp-config.php`で開く
   WordPress のためのデータベース名  
   define('DB_NAME', 'database_name_here');  
        ↓
   define('DB_NAME', 'mysqlで作ったデータベース名');  
   MySQL データベースのユーザー名  
   define('DB_USER', 'username_here');  
        ↓
   define('DB_USER', 'mysqlで作ったデータベースの所有者名');  
   MySQL データベースのパスワード  
   define('DB_PASSWORD', 'password_here');  
       ↓
   define('DB_PASSWORD', 'mysqlで作ったデータベースの所有者名のパスワード');  

9. Wordpressにアクセス
  `http://192.168.58.129/wp-admin/`

##2-3Wordpressをソースから構成
必要なソフトをインストール
無いとApacheを構築できないので予めインストールしておいてください。

centosプロキシの設定

1 プロキシを設定する
$vi /etc/profile ファイルに下記を追記する

 PROXY='172.16.40.1:8888'  
 export http_proxy=$PROXY  
 export HTTP_PROXY=$PROXY  
 export HTTPS_PROXY=$PROXY  
 export https_proxy=$PROXY  
2 yumにプロキシの設定
vi /etc/yum.conf 
ファイルに下記を追記
proxy=http://172.16.40.1:8888/
3 wgetにプロキシを設定
$vi /etc/wgetrc ファイルに下記を追記

http_proxy = 172.16.40.1:8888/  
https_proxy = 172.16.40.1:8888/  
ftp_proxy = 172.16.40.1:8888/  
4 プロキシの設定が終わったのでupdateする
$yum update

いらないらしい
# Cコンパイラ
yum -y install gcc
yum -y install gcc-c++
# PCRE
yum -y install pcre-devel
# OpenSSL
yum -y install openssl-devel
まとめての場合yum -y install gcc gcc-c++ pcre-devel openssl-devel

別にいらないらしい
APRをインストール
ARP（Apache Portable Runtime）は足りない機能を補完してどのOSでもソフトを使えるようにするライブラリなようです。
cd /usr/local/src
wget http://www.apache.org/dist/apr/apr-1.5.2.tar.gz
tar zxf apr-1.5.2.tar.gz
cd apr-1.5.2
./configure
make
make install

別にいらないらしい
ARP-utilをインストール
cd /usr/local/src
wget http://www.apache.org/dist/apr/apr-util-1.5.4.tar.gz
tar zxf apr-util-1.5.4.tar.gz
cd apr-util-1.5.4
./configure --with-apr=/usr/local/apr
make
make install

Apacheをインストール

wget http://ftp.tsukuba.wide.ad.jp/software/apache/httpd/httpd-2.2.31.tar.gz
# tar zxvf httpd-2.2.31.tar.gz
# cd httpd-2.2.31
./configure
make
#make install

PHPインストール
wget http://jp2.php.net/get/php-7.0.6.tar.bz2/from/this/mirror
tar -xvf mirror
sudo yum install libxml2-devel
./configure --with-apxs2=/usr/local/apache2/bin/apxs --with-mysqli
make

make install

Apacheを起動
sudo /usr/local/apache2/bin/apachectl startで起動

/usr/local/apache2/bin/apachectl stop で停止

/usr/local/apache2/bin/apachectl restart　再起動


エラーがでますので直していきましょう
/usr/local/apache2/conf/httpd.confの中身をいじっていきます。
sudo vi /usr/local/apache2/conf
#ServerName www.example.com:80
ServerName dacelo:80　←を追加
変更してもエラーがでるなら
Listen 80
↓
Listen 3000

php.iniファイルを設定する

インストールしてきたphpディレクトリの中で、

$ sudo cp php.ini-development /usr/local/lib/php.ini  

20 Apache が特定の拡張子のファイルを PHP としてパースするよう設定する
$sudo vi /usr/local/apache2/conf/httpd.confを開いて下記を追記

<FilesMatch \.php$>     
SetHandler application/x-httpd-php   
</FilesMatch>   

<IfModule dir_module>
DirectoryIndex index.html index.php ←index.phpを追加
</IfModule>

sudo /usr/local/apache2/bin/apachectl restartで再起動

Mariadbをインストール

sudo yum -y install mariadb mariadb-devel mariadb-server

sudo systemctl start mariadb

mariadbを起動する
$mysql -u root
rootユーザーにパスワードを設定する
MariaDB [(none)]> SET PASSWORD FOR 'root'@'localhost' = PASSWORD('xxxxxxxxxx');
1回終了してrootでログイン

MariaDB [(none)]> exit $mysql -u root -p

データベースの作成
MariaDB [(none)]> CREATE DATABASE データベース名;
MariaDB [(none)]> GRANT ALL PRIVILEGES ON データベース名.* to 'ユーザー名'@'localhost' IDENTIFIED BY '任意パスワード';
$sudo systemctl restart mariadbで再起動する

Wordpressのインストール

$wget http://wordpress.org/latest.tar.gz
$tar -xvf latest.tar.gzで解凍する
$sudo mv wordpress/ /usr/local/apache2/htdocs/で移動させる
$cd /usr/local/apache2/htdocs/wordpressで移動
$cp wp-config-sample.php wp-config.phpでwp-config.phpを作成
vi wp-config.phpでwp-config.phpを編集

WordPress のためのデータベース名
define('DB_NAME', 'database_name_here');
↓
define('DB_NAME', 'mysqlで作ったデータベース名');
MySQL データベースのユーザー名
define('DB_USER', 'username_here');
↓
define('DB_USER', 'mysqlで作ったデータベースの所有者名');
MySQL データベースのパスワード
define('DB_PASSWORD', 'password_here');
↓
define('DB_PASSWORD', 'mysqlで作ったデータベースの所有者名のパスワード');
↓
/** MySQL hostname */
define('DB_HOST', '127.0.0.1');


自分のIPアドレスhttp://192.168.33.130/wordpress/wp-admin/install.php

2-4ベンチマーク

1 Ubuntu側(ホスト側)にapache2-utilsインストール
$sudo apt-get install apache2-utils
2 abコマンドを使って性能を確かめる
1000ユーザが同時に1000リクエストを発行した場合を想定。
$ab -k -c 100 -n 100 192.168.33.130/wordpress
を実行したらこのような結果が表示される
This is ApacheBench, Version 2.3 <$Revision: 1706008 $>
Copyright 1996 Adam Twiss, Zeus Technology Ltd, http://www.zeustech.net/
Licensed to The Apache Software Foundation, http://www.apache.org/

Benchmarking 192.168.33.130 (be patient).....done


Server Software:        Apache/2.2.31
Server Hostname:        192.168.33.130
Server Port:            80

Document Path:          /wordpress
Document Length:        240 bytes

Concurrency Level:      100
Time taken for tests:   6.544 seconds
Complete requests:      100
Failed requests:        13
   (Connect: 0, Receive: 0, Length: 13, Exceptions: 0)
Non-2xx responses:      87
Keep-Alive requests:    87
Total transferred:      44979 bytes
HTML transferred:       20880 bytes
Requests per second:    15.28 [#/sec] (mean)
Time per request:       6543.576 [ms] (mean)
Time per request:       65.436 [ms] (mean, across all concurrent requests)
Transfer rate:          6.71 [Kbytes/sec] received

Connection Times (ms)
              min  mean[+/-sd] median   max
Connect:        0    8   4.6      7      15
Processing:    23 4322 1766.5   5011    6527
Waiting:       23 3708 2202.8   4539    6527
Total:         26 4330 1769.0   5011    6541

Percentage of the requests served within a certain time (ms)
  50%   5011
  66%   5533
  75%   5548
  80%   5556
  90%   5571
  95%   5578
  98%   6515
  99%   6541
 100%   6541 (longest request)

3 プラグイン導入
cd /usr/local/apache2/htdocs/wordpress/wp-content/plugins/
でプラグインの中に移動

wget -q http://downloads.wordpress.org/plugin/wp-super-cache.1.4.zipでwp-super-cacheをとってくる

unzip wp-super-cache.1.4.zipで展開して

Wordpressのプラグインのページから有効にして

ab -k -c 100 -n 100 192.168.33.130/wordpressコマンドを再度実行して
前のデータと比較して早くなっていればおｋ
