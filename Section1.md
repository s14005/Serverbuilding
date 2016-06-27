#Section1 基本のサーバー構築

##1-1 CentOS7のインストール
1. [Centos7の公式サイト](https://www.centos.org/)の(Get CentOS Now)からダウンロードページに移動  
2. 次にダウンロードページの(Minimal ISO)をクリック  
3. CentOS 7 Minimal ISO(x86_64)のISOファイルを選択しダウンロード  

##VirtualBoxの設定
1. virtualBoxを起動  
2. 新規ボタンを押して名前:centos7 タイプ:Linux バージョン:Red Hat(64bit)を設定して次へをクリック  
3. メモリーサイズに1024MB割り当てて次へをクリック  
4. 仮想ハードドライブを作成するを選んで作成をクリック  
5. VDIを選んで次へ可変サイズを選んで次へ  
6. 容量を8Gに設定して作成ををクリックでBox作成完了  
7. Boxの設定をクリック  
8. ホストオンリーアダプターの設定・・・VirtualBoxのファイル(ctr+g)→ネットワーク→ホストオンリーアダプターを選んでホストオンリーアダプターを追加する  
9. 作ったcentos7の設定をおしてネットワークの設定に移動してアダプター2をクリック  
    ネットワークアダプターを有効化にチャックを入れる。  
    割り当て:ホストオンリーアダプターにする  
10. Boxにcentos7のisoイメージをセットする・・・Boxの設定→ストレージを選んぶ→コントローラ：IDEの下のCDのマークをクリック→CD/DVDドライブの右側のCDマークをクリックしてインストールしたCenots7を選ぶ  
11. Boxの設定完了  

##VirtualBoxにCentos7をインストール
1. VirtualBoxを起動  
2. install CentOS7を選択  
3. 言語選択で日本語を選択  
4. キーボードのマークをクリックして日本語を削除し英語(us)を追加  
5. インストール先を選んでHDDを選ぶ  
6. インストールの開始をクリック  
7. インストール中にrootのパスワード設定とroot以外のユーザを作成する  

##Centos7の設定
1. `$vi /etc/sysconfig/network-scripts/ifcfg-enp0s3`のファイルを開いて ONBOOT=onをONBOOT=yesに変更するifcfg-enp0s8のファイルも同じようにする  
2. 設定を反映するために  
   `$/etc/sysconfig/network-scripts/ifup enp0s3`  
   `$/etc/sysconfig/network-scripts/ifup enp0s8`  
   を実行する  
3. 設定が反映されてるか確認するため  
   `$ip addr`  
   を実行すると`enp0s8`に`192.168.xxx.xxx`ってアドレスが振られてる  
4. enp0s8のipアドレスを覚えてubuntuの端末から`$ssh 192.168.xxx.xxx`でアクセスする  
5. プロキシを設定する  
   `$vi /etc/profile` ファイルに下記を追記する  

    PROXY='172.16.40.1:8888'  
    export http_proxy=$PROXY  
    export HTTP_PROXY=$PROXY  
    export HTTPS_PROXY=$PROXY  
    export https_proxy=$PROXY  

6. yumにプロキシを設定  
   `$vi /etc/yum.conf`に下記を追記  
   `proxy=http://172.16.40.1:8888/`  
7. wgetにプロキシを設定  

    $vi /etc/wgetrcに下記を追記  
    http_proxy = 172.16.40.1:8888/  
    https_proxy = 172.16.40.1:8888/  
    ftp_proxy = 172.16.40.1:8888/  

8. プロキシの設定が終わったのでupdateする  
   `$yum update`  
## Wordpressに必要なものをインストール
1. Wordpressに必要なものをインストール・・・PHP Apache  
   `$ yum install -y unzip wget httpd php php-mysql php-gd php-mbstring`  
2. MySQLのインストール  
   `$wget http://dev.mysql.com/get/mysql-community-release-el7-5.noarch.rpm`  
   MySQLのリポジトリのインストール  
   `$rpm -Uvh mysql-community-release-el7-5.noarch.rpm`  
   設定ファイルを下記のように書き換える  
   `$/etc/yum.repos.d/mysql-community.repo`  

    [mysql-connectors-community]  
    enabled=1→enabled=0  
    [mysql-tools-community]  
    enabled=1→enabled=0  
    [mysql56-community]  
    enabled=1→enabled=0  

3. MySQLのインストール  
   `$yum --enablerepo=mysql56-community install mysql-community-server`
4. MySQLとApacheを起動する  
   `$systemctl start mysqld.service`・・・MySQL  
   `$systemctl start httpd.service`・・・Apache  
5. MySQLとApacheをずっと起動したままにする  
   `$systemctl enable mysqld.service`  
   `$systemctl enable mysqld.service`  
6. SELinux無効設定  
   `$vi /etc/sysconfig/selinux`を開いて下記を変更  
   `SELINUX=enforcing→SELINUX=disabled`  
7. firewalld設定・・・http,httpsを許可する  
   `$firewall-cmd --add-service=http --zone=public --permanent`  
   `$firewall-cmd --add-service=https --zone=public --permanent`  

## Wordpressで使うデータベースの作成
1. MySQLを起動する  
   `$mysql -u root`
2. rootユーザーにパスワードを設定する  
   `mysql> SET PASSWORD FOR 'root'@'localhost' = PASSWORD('xxxxxxxxxx');`
3. 1回終了してrootでログイン  
   `mysql>exit`
   `$mysql -u root -p`
4. wordpressで使うデータベースを作成する  
   `mysql> CREATE DATABASE データベース名;`  
5. ユーザの作成    
   `mysql> GRANT ALL PRIVILEGES ON データベース名.* to 'ユーザー名'@'localhost' IDENTIFIED BY '任意パスワード';`

## Wordpressのインストール
1. Wordpressのzipファイルをインストールする  
   `$wget https://ja.wordpress.org/wordpress-4.2.2-ja.zip`  
2. インストールしたzipファイルを解凍する  
   `$unzip wordpress-4.2.2-ja.zip`  
3. 解凍したファイルを/var/www/htmlに移動する  
   `$mv wordpress-4.2.2-ja.zip /var/www/html`  
4. wordpressのフォルダに移動  
   `$cd /var/www/html/wordpress`  
5. wordpressフォルダの`wp-config-sample.php`をコピーして`wp-config.php`を作成する  
   `$cp wp-config-sample.php wp-confing.php`  
6. コピーしたwp-config.phpの中を編集  
   `$vi wp-config.php`  

    `WordPress のためのデータベース名`   
    `define('DB_NAME', 'database_name_here');`  
    ↓  
    `define('DB_NAME', 'mysqlで作ったデータベース名');`  
    `MySQL データベースのユーザー名`  
    `define('DB_USER', 'username_here');`  
    ↓  
    `define('DB_USER', 'mysqlで作ったデータベースの所有者名');`  
    `MySQL データベースのパスワード`  
    `define('DB_PASSWORD', 'password_here');`  
    ↓  
    `define('DB_PASSWORD', 'mysqlで作ったデータベースの所有者名のパスワード');`  

7. ブラウザからWordpressにアクセス  
   `192.168.xxx.xxx/wordpress/wp-admin/install.php`
