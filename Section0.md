#Section 0 講義の前のセットアップ

##0-1 VirtualBoxのインストール

[公式サイト](https://www.virtualbox.org/wiki/Linux_Downloads)からDL


以下のコマンドを実行してインストール

sudo apt install libsdl1.2debian  dkms libsdl-ttf2.0-0  libqt4-opengl

sudo dpkg -i virtualbox-5.0_5.0.20-106931-Ubuntu-xenial_amd64.deb

##0-2 Vagrantのインストール
[Vagrant公式](https://www.vagrantup.com/downloads.html)からDL
LINUX(DEB)の64bitをダウンロード
dpkg -i vagrant_1.8.1_x86_64.deb
vagrant -v で確認
