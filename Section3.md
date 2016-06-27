#Section 3 Ansibleによる自動化とテスト
##3-0 Ansibleのインストール
1. Ansibleをインストール  
   `$sudo apt-get install software-properties-common`  
   `$sudo apt-add-repository ppa:ansible/ansible`  
   `$sudo apt-get update`  
   `$sudo apt-get install ansible`  

##3-1 ansibleでWordpressを動かす
1. vagrantのプラグインのインストール  
   `$vagrant plugin install vagrant-proxyconf`  
2. 作業用のディレクトリを作成しその中でVagrantfileの作成  
   `$vagrant init Centos7`  
3. ホストオンリーアダプターの設定  
   `$vi Vagrantfile`を開く  
   Vagrant.configure(2) do |config| emd  
   の間に`config.vm.network :private_network,ip:"192.168.58.124"を追加  
4. proxyの設定 Vagrantfileに下記を追加  
   if Vagrant.has_plugin?("vagrant-proxyconf")  
      config.proxy.http     = ENV["HTTP_PROXY"]  
      config.proxy.https    = ENV["HTTP_PROXY"]  
      config.proxy.no_proxy = ENV["NO_PROXY"]  
   end  
5. hostsファイルを作る  
   `$vi hosts`  
     ↓書き込み  
   (自分のサーバーのIPアドレス)  
   または`$ echo 自分のサーバーのIPアドレス > hosts`  
6. playbook.ymlを作成。同じ場所にnginx,php-fpm,wordpressディレクトリを作成  
   `$mkdir nginx php-fpm wordpress`  
7. 頑張ってググッてplaybookを書いていこう!  
8. playbookを実行する  
   `$ ansible-playbook playbook.yml -i hosts --private-key ~/.vagrant.d/insecure_private_key -u vagrant -k -vv`  

##3-1-2 VagrantfileからAnsibleを呼び出す
ggr(WIP)  
