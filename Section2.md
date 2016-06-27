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
   変更　↓↓  
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

   location / {  
       root   /usr/share/nginx/html;  
       index  index.html index.htm;  
   }  
     変更 ↓  ↓  
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
      　変更↓　　　　↓  
location ~ \.php$ {  
       root   /var/www/;  
       fastcgi_pass   127.0.0.1:9000;  
       fastcgi_index  index.php;  
       fastcgi_param  SCRIPT_FILENAME  $document_root$fastcgi_script_name;  
       include        fastcgi_params;  
     }  


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
##centosプロキシの設定
1. プロキシを設定する  
   `$vi /etc/profile` でファイルを開く  
    　↓を追加  
   PROXY='172.16.40.1:8888'  
   export http_proxy=$PROXY  
   export HTTP_PROXY=$PROXY  
   export HTTPS_PROXY=$PROXY  
   export https_proxy=$PROXY  

2. yumにプロキシの設定  
   `$vi /etc/yum.conf`でファイルを開く   
       ↓を追加  
   proxy=http://172.16.40.1:8888/  

3. wgetにプロキシを設定  
   `$vi /etc/wgetrc`でファイルを開く
       ↓を追加
   http_proxy = 172.16.40.1:8888/  
   https_proxy = 172.16.40.1:8888/  
   ftp_proxy = 172.16.40.1:8888/  

4. プロキシの設定が終わったのでyumをupdateする  
   `$yum update`  

##Apacheのインストール
1. 作業するディレクトリに移動  
   `$cd /usr/local/src`  
2. Apacheをダウンロード  
   `$wget http://ftp.tsukuba.wide.ad.jp/software/apache/httpd/httpd-2.2.31.tar.gz`  
3. ダウンロードしたファイルを解凍  
   `$tar zxvf httpd-2.2.31.tar.gz`  
3. 解凍したディレクトリに移動  
   `$cd httpd-2.2.31`  
4. ソースツリーの設定  
   `$./configure`  
5. ビルド  
   `$make`  
6. インストール  
   `$make install`  

##PHPインストール
1. 作業するディレクトリに移動  
   `$cd /usr/local/src`  
2. PHPをダウンロード  
   `$wget http://jp2.php.net/get/php-7.0.6.tar.bz2/from/this/mirror`  
3. ダウンロードしたファイルを解凍  
   `$tar -xvf mirror`  
4. エラーがでるのでlibxml2-develをインストールしておく  
   `$yum install libxml2-devel`  
5. ソースツリーの設定  
   `$./configure --with-apxs2=/usr/local/apache2/bin/apxs --with-mysqli`  
6. ビルド  
   `$make`  
7. インストール  
   `$make install`  

##Apache起動,設定
1. Apacheを起動  
   `$/usr/local/apache2/bin/apachectl start`で起動  

2. エラーがでるので治していく  
   `$sudo vi /usr/local/apache2/conf`で開く  
   `ServerName www.example.com:80`これの下に追加  
   `ServerName dacelo:80`　←を追加  

3. php.iniファイルを設定する  
   インストールしてきたphpディレクトリの中で(コピーしてる)  
   `$ sudo cp php.ini-development /usr/local/lib/php.ini`  
4. Apacheが特定の拡張子のファイルを PHP としてパースするよう設定する  
   `$sudo vi /usr/local/apache2/conf/httpd.conf`を開いて下記を追記  

   <FilesMatch \.php$>  
   SetHandler application/x-httpd-php  
   </FilesMatch>  

   <IfModule dir_module>  
   DirectoryIndex index.html index.php ←index.phpを追加  
   </IfModule>  

5. Apacheの再起動  
   `$sudo /usr/local/apache2/bin/apachectl restart`  

##Mariadbをインストール

1. Mariadbのインストール  
   `sudo yum -y install mariadb mariadb-devel mariadb-server`  
2. Mariadbのサービス起動  
   `$sudo systemctl start mariadb  
3. mariadbを起動する  
   `$mysql -u root`  
   rootユーザーにパスワードを設定する  
   MariaDB [(none)]> SET PASSWORD FOR 'root'@'localhost' = PASSWORD('xxxxxxxxxx');  
   1回終了してrootでログイン  
   MariaDB [(none)]> exit;   
  `$mysql -u root -p`  
   データベースの作成  
   MariaDB [(none)]> CREATE DATABASE データベース名;  
   MariaDB [(none)]> GRANT ALL PRIVILEGES ON データベース名.* to 'ユーザー名'@'localhost' IDENTIFIED BY '任意パスワード';  
4. Mariadbを再起動  
   `$sudo systemctl restart mariadb`で再起動する  

##Wordpress,設定のインストール

1. wordpressをダウンロード  
   `$wget http://wordpress.org/latest.tar.gz`  
2. ダウンロードしてきたwordpressを解凍  
   `$tar -xvf latest.tar.gz`  
3. wordpressを移動させる  
   `$sudo mv wordpress/ /usr/local/apache2/htdocs/`  
4. wordpressに移動  
   `$cd /usr/local/apache2/htdocs/wordpress`  
5. wp-config.phpを作成  
   `$cp wp-config-sample.php wp-config.php`  
6. wp-config.phpの編集  
   `$vi wp-config.php`  

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
6. Wordpressの確認  
   自分のIPアドレス`http://192.168.58.130/wordpress/wp-admin/install.php`  

##2-4ベンチマーク

1. Ubuntu側(ホスト側)にapache2-utilsインストール  
   `$sudo apt-get install apache2-utils`  
2. abコマンドを使って性能を確かめる(Ubuntu側)  
   100ユーザが同時に100リクエストを発行した場合を想定。  
   `$ab -k -c 100 -n 100 192.168.33.130/wordpress`を実行したらこのような結果が表示される  
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
                 `min  mean[+/-sd] median   max`  
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

3. プラグイン導入,プラグインファイルの中に移動(centOS7側)  
   `$cd /usr/local/apache2/htdocs/wordpress/wp-content/plugins/`  
4. wp-super-cacheをダウンロード(CentOS7側)  
   `$wget -q http://downloads.wordpress.org/plugin/wp-super-cache.1.4.zipでwp-super-cache`
5. ダウンロードしたものを展開
   `$unzip wp-super-cache.1.4.zip`
6. Wordpressのプラグインのページから有効にして
7. コマンドを実行して前のデータと比較して早くなっていればおｋ
   `$ab -k -c 100 -n 100 192.168.33.130/wordpress`
