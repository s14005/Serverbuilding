#Section 2 その他のWebサーバー環境
##2-1 Vagrantを使用したCentOS 7環境の起動

1. vagrant用のCentOSを先生のUSBからコピー
2. vagrant用のBOXをforceする  
   `$vagrant box add CentOS7 コピーしたbox ファイル --force`  
3. 作業用のディレクトリを作成してその中でVagrantの初期設定を行います
   `$vagrant init CentOS7`
3. ホストオンリーアダプターの設定
   `$vi Vagrantfile`で開いて  
   Vagrant.configure(2) do |config|  
   これを追加→config.vm.network :private_network, ip:"192.168.56.129"  
   end  

4. Vagrantを起動
   `$vagrant up`  
5. Vagrantに接続
   `$vagrant ssh`  

##2-2 Wordpressを動かす
##プロキシの設定

1. プロキシの設定
   `$vi /etc/profile`ファイルを開く  
 
   PROXY='172.16.40.1:8888'  
   export http_proxy=$PROXY  
   export HTTP_PROXY=$PROXY  
   export HTTPS_PROXY=$PROXY  
   export https_proxy=$PROXY  

2yumにプロキシを設定
$vi /etc/yum.confファイルを開く
↓を追加
proxy=http://172.16.40.1:8888/

3wgetにプロキシの設定
$vi /etc/wgetrcファイルを開く
↓を追加
http_proxy = 172.16.40.1:8888/  
https_proxy = 172.16.40.1:8888/  
ftp_proxy = 172.16.40.1:8888/

4yumのアップデート
$yum update

##Wordpressに必要なものをインストール
1nginxのリポジトリのインストール
$sudo yum install http://nginx.org/packages/centos/7/noarch/RPMS/nginx-release-centos-7-0.el7.ngx.noarch.rpm

2Nginxのパッケージのインストール
$ sudo yum install --enablerepo=nginx nginx

3自動起動に設定
$sudo systemctl enable nginx.service

4サービスを起動
$sudo systemctl start nginx

5PHPと必要なモジュールをインストール
$sudo yum -y install php-mysql php php-gd php-mbstring php-fpm

6php-fpm設定
sudo vi /etc/php-fpm.d/www.conf開いて

user = apache
group = apach
変更　↓
user = nginx
group = nginx

7自動設定に変更
$ sudo systemctl enable php-fpm.service

8起動してないので起動
$ sudo systemctl start php-fpm.service

9MariaDBのインストール
yum -y install mariadb mariadb-server

10自動設定
systemctl enable mariadb

11サービスの起動
systemctl start mariadb

12設定
mysql_secure_installation
全部yでおｋ

13再起動
systemctl restart mariadb

14 1回終了してrootでログイン
mysql>exit
$mysql -u root -p

15wordpressで使うデーターベースを作成する
mysql> CREATE DATABASE データベース名;

16ユーザーの作成
mysql> GRANT ALL PRIVILEGES ON データベース名.* to 'ユーザ名'@'localhost' IDENTIFIED BY '任意パスワード';

17Wordpressをダウンロード
wget http://wordpress.org/latest.tar.gz

18解凍
tar xzfv latest-ja.tar.gz

19所有者とグループをnginxに変更
chown -R nginx:nginx wordpress

20wordpressをディレクトリ/var/www/にいどう
mv wordpress /var/www

21default.confの中身を変更
rootをwordpressを置いた場所に変更
location / {  
  root   /usr/share/nginx/html;  
  index  index.html index.htm;  
  }  
↓  
location / {  
  root    /var/www/;
  index index.php index.html index.htm;
  }


location ~ \.php$ {
         html;
         fastcgi_pass   127.0.0.1:9000;
         fastcgi_index  index.php;
         fastcgi_param  SCRIPT_FILENAME  /scripts$fastcgi_script_name;
         include        fastcgi_params;
     }
↓
location ~ \.php$ {
         root   /var/www/;
         fastcgi_pass   127.0.0.1:9000;
         fastcgi_index  index.php;
         fastcgi_param  SCRIPT_FILENAME  $document_root$fastcgi_script_name;
         include        fastcgi_params;
     }

fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
  
22nignxを再起動
systemctl restart nginx

23wordpressフォルダのwp-config-sample.phpをコピーしてwp-config.phpを作成する
wordpressフォルダの中でcp wp-config-sample.php wp-config.php

24コピーしたwp-config.phpの中を編集
$vi wp-config.phpで開く

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

25Wordpressにアクセス
http://192.168.33.129/wp-admin/


