# 環境構築手順書

## バージョン一覧
|OSS|Ver.|
|-------|-------|
|macOS Catalina|10.15.5|
|PHP|7.3.25|
|nginx|1.19.5|
|MySQL|5.7.32|
|laravel|6.20.5|


## Vagrantの作業ディレクトリを用意する
以下のコマンドを実行してください。
`$ mkdir virtual_env_manual`

## vagrantの導入・設定
### VirtualBoxのインストール
下記のサイトからそれぞれのdmgファイルをダウンロード後、インストールを進めてください。
[Virtual Box公式](https://www.virtualbox.org/wiki/Download_Old_Builds_6_0)
**※ Vagrantの最新バージョンがVirtualBoxの最新バージョンに対応していないため、Virtual Boxはver6.0.14をインストールするようにしてください。**
以下のコマンドを実行してVirtualBoxのウィンドウが表示されれば正常にインストールされています。
`$ virtualbox`
コマンド実行後は入力を受け付けない状態となるため、 `Control + c` を押してください。


### Vagrantのインストール
下記コマンドでvagrantをインストールしてください。
`$ brew cask install vagrant`
下記コマンドでバージョンの確認ができたら問題ありません。
`$ vagrant -v`


### vagrant boxのダウンロード
LinuxのCentOSのバージョン7のbox名 `centos/7` を指定して実行。
`vagrant box add centos/7`
コマンドを実行すると、下記のような選択肢が表示されます。
```
1) hyperv
2) libvirt
3) virtualbox
4) vmware_desktop

Enter your choice: 3
```
今回使用するソフトはVirtualBoxのため、3を選択してenterを押し,下記のように表示されたら完了です。
`Successfully added box 'centos/7' (v1902.01) for 'virtualbox'!`

`cd vagrant_lesson`で最初に作成したディレクトリ`vagrant_lesson`に移動し,以下のコマンドを実行します。
```
vagrant init centos/7

# 実行後問題なければ以下のような文言が表示されます
A `Vagrantfile` has been placed in this directory. You are now
ready to `vagrant up` your first virtual environment! Please read
the comments in the Vagrantfile as well as documentation on
`vagrantup.com` for more information on using Vagrant.
```

### Vagrantfileの編集
`vi Vagrantfile`
でVagrantfileを編集していきます。
ファイルの中身が表示されたら `i` を押すと編集が可能になります。
今回行う編集は、三箇所です。
```
# 変更点①
config.vm.network "forwarded_port", guest: 80, host: 8080

# 変更点②
config.vm.network "private_network", ip: "192.168.33.19"

# 変更点③
config.vm.synced_folder "../data", "/vagrant_data"
# ↓ 以下に編集
config.vm.synced_folder "./", "/vagrant", type:"virtualbox"
```
3点全て # を外し、3つ目に関しては上記の通り変更してください。
編集後はescで編集モードを終了し `:wq` を入力すると保存して終了できます。


### Vagrant プラグインのインストール
今回は vagrant-vbguest というプラグインをインストールします。
`vagrant plugin install vagrant-vbguest`
vagrant-vbguestのインストールが完了しているか下記のコマンドを実行して確認してください。
`vagrant plugin list`


## Vagrantを使用してゲストOSの起動,ログイン
Vagrantfileがあるディレクトリにて以下のコマンドを実行してください。
`vagrant up`
起動が正常に終了したら次は下記コマンドでログインします。
`$ vagrant ssh`
実行後このような表示になればゲストOSへのログインができています。
```
Welcome to your Vagrant-built virtual machine.
[vagrant@localhost ~]$
```


### パッケージをインストール
下記のコマンドを実行してください。
`sudo yum -y groupinstall "development tools"`
このコマンドを実行することによりgitなどの開発に必要なパッケージを一括でインストールできます。

### PHPをインストール
```
sudo yum -y install epel-release wget
sudo wget http://rpms.famillecollet.com/enterprise/remi-release-7.rpm
sudo rpm -Uvh remi-release-7.rpm
sudo yum -y install --enablerepo=remi-php73 php php-pdo php-mysqlnd php-mbstring php-xml php-fpm php-common php-devel php-mysql unzip
php -v
```
phpのバージョンが確認できればインストール完了です。


### composerのインストール
次にPHPのパッケージ管理ツールであるcomposerをインストールしていきます。
```
php -r "copy('https://getcomposer.org/installer', 'composer-setup.php');"
php composer-setup.php
php -r "unlink('composer-setup.php');"

# どのディレクトリにいてもcomposerコマンドを使用を使用できるようfileの移動を行います
sudo mv composer.phar /usr/local/bin/composer
composer -v
```


### Laravelのインストール
`composer create-project --prefer-dist laravel/laravel laravel6 "6.0"`


### データベースのインストール
今回インストールするデータベースはMySQLとなります。versionは5.7を使用します。
```
sudo wget http://dev.mysql.com/get/mysql57-community-release-el7-7.noarch.rpm
sudo rpm -Uvh mysql57-community-release-el7-7.noarch.rpm
sudo yum install -y mysql-community-server
mysql --version
```
versionの確認ができましたらインストール完了です。
次にMySQLを起動し接続を行います。
MySQLにパスワードが設定されているため、パスワードを調べて再設定します。
```
sudo cat /var/log/mysqld.log | grep 'temporary password'
# このコマンドを実行したら下記のように表示されたらOKです
2017-01-01T00:00:00.000000Z 1 [Note] A temporary password is generated for root@localhost: hogehoge
```

### データベースの作成
実際にLaravelのTodoアプリケーションを動かす上で使用するデータベースの作成を行います。
`mysql > create database laravel_app;`
Query OKと表示されたら作成は完了となります。

### .envファイルの編集
laravel_appディレクトリ下の `.env` ファイルの内容を以下に変更してください。
```
cd laravel_app
vi .env
```
```
DB_PASSWORD=
# ↓ 以下に編集
DB_PASSWORD=登録したパスワード
```
完了したらlaravel_appディレクトリに移動して `php artisan migrate` を実行します。


### Nginxのインストール
Nginxの最新版をインストールしていきます。
`$ sudo vi /etc/yum.repos.d/nginx.repo`
書き込む内容は以下になります。
```
[nginx]
name=nginx repo
baseurl=http://nginx.org/packages/mainline/centos/\$releasever/\$basearch/
gpgcheck=0
enabled=1
```
書き終えたら保存して、以下のコマンドを実行しNginxのインストールを実行します。
```
$ sudo yum install -y nginx
$ nginx -v
```
バージョンが確認できたらNginxを起動します。
`$ sudo systemctl start nginx`



### 設定ファイルの編集
`$ sudo vi /etc/nginx/conf.d/default.conf`
編集箇所は以下の通りです。
```
  server_name  192.168.33.19; # Vagranfileでコメントを外した箇所のipアドレスを記述してください。
  # ApacheのDocumentRootにあたります
  root /vagrant/laravel6/public; # 追記
  index  index.html index.htm index.php; # 追記

  #charset koi8-r;
  #access_log  /var/log/nginx/host.access.log  main;

  location / {
      #root   /usr/share/nginx/html; # コメントアウト
      #index  index.html index.htm;  # コメントアウト
      try_files $uri $uri/ /index.php$is_args$args;  # 追記
  }

  # 省略

  # 該当箇所のコメントを解除し、必要な箇所には変更を加える
  # 下記は root を除いたlocation { } までのコメントが解除されていることを確認してください。

  location ~ \.php$ {
  #    root           html;
      fastcgi_pass   127.0.0.1:9000;
      fastcgi_index  index.php;
      fastcgi_param  SCRIPT_FILENAME  /$document_root/$fastcgi_script_name;  # $fastcgi_script_name以前を /$document_root/に変更
      include        fastcgi_params;
  }

  # 省略
```

php-fpm の設定ファイルも編集していきます。

`$ sudo vi /etc/php-fpm.d/www.conf`
変更箇所は以下になります。
```
;24行目近辺
user = apache
# ↓ 以下に編集
user = nginx

group = apache
# ↓ 以下に編集
group = nginx
```
下記コマンドにてnginx というユーザーでもログファイルへの書き込みができる権限を付与します。
```
$ cd /vagrant/laravel_app
$ sudo chmod -R 777 storage
```
完了したら起動してください。
```
$ sudo systemctl restart nginx
$ sudo systemctl start php-fpm
```

### laravelへログイン機能の実装
```
composer require laravel/ui 1.*
php artisan ui vue --auth
git clone https://github.com/creationix/nvm.git ~/.nvm
nvm install stable
npm install
npm run dev
```

**ブラウザでhttp://192.168.33.19が表示できれば完了。**


## 環境構築の所感
今までGIZTECHの流れに沿って学習を進めてきたが今回は自分で調べて実装する箇所があり苦労した。
エラーが色々な箇所で出ておりつまずいたがよく出てくるエラーについては対処法が身に付いたと感じる。

## 参考サイト
[Laravel学習帳](https://laraweb.net/tutorial/6792/)
[ReaDouble(laravel6.*)](https://readouble.com/)
[Qiita_Markdown記法 サンプル集](https://qiita.com/tbpgr/items/989c6badefff69377da7)
[Darablog(Vagrantで【nvm】を使用して《node.js》と《npm》を入れる方法)](https://dara-blog.com/node-install-in-vagrant)
