#Section6
##6-0 AWSコマンドラインインターフェイスのインストール
1. awcliをインストールする  
   `$pip install awscli`  
2. awscliの設定を変更する  
   `$aws configure`
   AWS Access Key ID [None]: 先生からもらったID  
   AWS Secret Access Key [None]: 先生からもらったKey  
   Default region name [None]: ap-northeast-1  
   Default output format [None]: json  


   ssh s14005@172.16.40.2でよなしろせんせいのgrusに接続  
   scp s14005.pem s14005@172.16.40.2:で鍵をコピーする  

##6-1 AWS EC2 + Ansible
##EC2

1. EC2のアイコンをクリックしてEC2のダッシューボードにアクセス  
2. インスタンスの作成  
3. インスタンスの作成をクリック  
4. Amazon linuxを選択  
5. t2.microを選択  
6. 新しいキーペアを作成する※ダウンロードしたキーファイルはsshで使うので消さないように！  
7. インスタンスの作成をクリック  
8. 動いているマシンの一覧が出てくるから自分のを選択して一番右にあるセキュリティグループをクリックする  
9. グループを選択して右クリックしてインバウンドルールの編集を選択してルールを追加をクリックしてHTTPを追加する  

##ansible

1. grusを使うのでgrusに作成したキーファイルとansibleファイルをコピーする  
   `$ scp -r コピーしたい物 学籍番号@172.16.40.2:`  
2. playbookファイルをAWSで動くように書き換える  
3. hostsファイルをなければ作成あればEC2のアドレスに書き換える  
4. playbook実行 
   `$ansible-playbook playbook.yml -i hosts --private-key pemキー -u ec2-user -k`  

##AMIを作る
1. EC2にあるAMI作成ボタンをクリック  
2. インスタンの作成でマイAIMを選択  
3. 自分で作成したAMIを選択  
4. 6-1と同じ方法でインスタンスを作成しする  
5. パブリックDNSのhttp://でアクセス6-1で作ったWordpreessと同じ画面がでていればおｋです  

##6-2 AMIMOTO
1. インスタンの作成をクリック  
2. 左のメニューバーから`AWS Marketplace`を選択して`AMIMOTO`を検索する  
3. `WordPress powered by AMIMOTO (HHVM)`を選択  
4. 表の一番上の`t2.micro`を選択して右下にある確認と作成をクリック  
5. セキュリティグループのところでエラーが出るので編集を選択  
6. セキュリティグループを6-1で作ったものに変える  
7. 6−1で作ったキー名を選択してインスタンスの作成をクリック  

##6-4 CloudFront
1. AMIを使ってインスタンスを作る  
2. CloudFrontに行ってCreate Distributionをクリック  
3. WebとRTMPがあるが、Webを選択  
4. Origin Domain Nameに1で作ったインスタンスのパブリックDNSを指定。  
5. Create Distributionを押す  
6. Domain nameをhttp://のあとに追加して実行  

##6-5 ELB
0. Amazon web servicesにログインする
1. EC2のアイコンをクリックしてアクセス  
2. EC2で3台くらいマシンを動かしとく  
3. 左側のメニューからロードバランサーをクリック  
4. ロードバランサーの作成をクリック  
5. セキュリティグループも動いてるマシンと同じにする  
6. ステップ 5まで移動  
   自分のマシンをすべて選択する  
7. ステップ7まで移動  
   作成をクリック   
8. すべてのマシンにSSHで接続  
9. ログを表示する  
   $cd /var/log/nginx/  
   $tail -f accecs.log  
10. IPアドレスが分散されているか確認  
11. ロードバランサーに行って、説明の一番上にあるDNSのアドレスに繋いで、最初に作ったワードプレスが表示されたら終了  
