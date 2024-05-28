# EC2にnginxを設定する手順

NGINX が一番よく利用される用途は、HTTP を経由してウェブリソースを提供するウェブサーバとしての役割。

## 事前準備
新たに `web` という名前のフォルダを作成し、その中に GitHub からリポジトリをクローンします。 

## 本番環境の Ubuntu に NGINX をインストール
```
sudo apt update
sudo apt install nginx
```
システムがカーネルの依存関係を更新するように求めた場合は、Enter をクリックしてデフォルトのまま続行してください。
その後、`sudo reboot` コマンドを実行してシステムを再起動し、再起動後に SSH で再接続します。

NGINX が正常にインストールされたかを確認するには、`nginx -v` コマンドを実行します。
正常にインストールされていればバージョン番号が表示されます。`nginx version: nginx/1.24.0 (Ubuntu)`

最後に、NGINX の設定ファイルの場所を確認します。一般的に、設定ファイルは `/etc/nginx` 下に、ブータブル（実行可能）ファイルは `/usr/sbin/nginx` 下にインストールされます。 
`ls /etc/nginx/` コマンドを実行すれば、設定ファイルの一覧を表示できます。

```
ubuntu@ip-172-31-36-126:~$ ls -F /etc/nginx/
conf.d/         koi-win             nginx.conf        sites-enabled/
fastcgi.conf    mime.types          proxy_params      snippets/
fastcgi_params  modules-available/  scgi_params       uwsgi_params
koi-utf         modules-enabled/    sites-available/  win-utf
```

nginx.conf はウェブサーバ全体の設定を保存し、mime.types はウェブサーバが提供可能なファイルタイプを指定します。
`include /etc/nginx/mime.types`　行がある場合 → mime.types ファイルが正しく読み込まれている。
`include /etc/nginx/sites-enabled/*` 行がある場合 → HTTP のサイト設定が sites-enabled フォルダに委譲。

sites-enabled フォルダには、個々のサイトごとの HTTP ウェブサーバ設定が保存されます。この設定により、一つの物理サーバ上で複数のウェブサイトをホストすることが可能になります。
example.com → `/etc/nginx/sites-enabled/example.com` 
another-example.com → `/etc/nginx/sites-enabled/another-example.com`

## NGINXの起動
- 全てのウェブサーバ設定ファイルを保存する場所 → `/etc/nginx/sites-available/`
- 実際に有効化したい設定ファイルへのシンボリックリンクを保持する場所 → `/etc/nginx/sites-enabled/`
- default設定フォルダー → NGINX が他のドメイン設定を見つけられない場合に利用される `/var/www/html/index.nginx-debian.html`

- NGINX を起動: `sudo systemctl start nginx`
- NGINX を停止: `sudo systemctl stop nginx`
- NGINX を再起動: `sudo systemctl restart nginx`

※ EC2 以外のサービスを利用している場合
セキュリティ強化のためにファイアウォールルールの設定を行うことが推奨されます。
- `ufw app list`: ufw が認識しているアプリケーションの一覧が表示されます。
- `sudo ufw allow 'Nginx Full'`: HTTP と HTTPS のアクセスを許可。
- `sudo ufw allow 'OpenSSH'` というコマンドを実行して、SSH 接続を許可。

## ウェブサイトのホスト
- Step1: ディレクトリの設定
  ウェブサイトは `/var/www/{website}` の下に置く。
  public フォルダを設定し、そのフォルダの下にあるすべてのファイルを他の人が読めるように公開する。
  HOMEディレクトリ外では sudo 権限が必要
  ```
  sudo mkdir -p /var/www/{website}/public
  ```
  フォルダの所有者を現在のユーザーに変更
  ```
  sudo chown -R $USER:$USER /var/www/{website}/public
  ```
  表示確認
  ```
  echo $USER
  ```
  
- Step2: シンボリックリンクの作成
  ウェブサイトのディレクトリと public サブディレクトリの間に接続を作成します。
  ウェブサイトのディレクトリに置くものは何でも public ディレクトリからアクセスすることができるようにできます。
  ```
  sudo ln -s ~/web/{project-website} /var/www/{website}/public
  ```
  
- Step3: NGINX のアクセス権限の設定
  ウェブサイトを実行するソフトウェア（この場合は NGINX）が、訪問者にファイルをアクセスおよび表示するために必要なアクセス許可を持っていることを確認します。
  ```
  chmod 755 /home/{user}
  ```
  （EC2 の場合は user が ubuntu）

- Step4: シンボリックリンクの確認
  ウェブサイトにコンテンツを追加し始める前に、すべてが正しく設定されていることを確認するためのダブルチェック。

  - 以下のコマンドで作成したシンボリックリンクが正しく機能していることを確認
  ```
  ls -l /var/www/{website}/public
  ls -l /var/www/{website}/public/{project-website}/
  ```
  
- Step5: NGINX のデフォルトサーバ設定し、ホームページ を表示
  NGINX のデフォルトサーバ設定
  1. `sudo vi /etc/nginx/sites-available/default` を実行して、NGINX のデフォルトサーバ設定ファイルを開きます。
  2. 設定ファイル内の `root /var/www/html;` という行をコメントアウトし、新しいルートパスである `root /var/www/{website}/public/{project-website};` を追加します。
  3. `index.html index.htm index.nginx-debian.html;` という行もコメントアウトし、新しいインデックスとして index {index.html}; を追加します。index は、誰かがウェブサイトの URL にナビゲートしたときにどのファイルを提供するかを NGINX に指示します。
     ここで指定したhtmlファイルがウェブサイトのホームページになります。
  5. シンボリックリンクが設定され、`sites-enabled` に設定が反映されていることを確認してください。これはnginxが使用するものです。 `sudo ln -s /etc/nginx/sites-available/default /etc/nginx/sites-enabled` を実行します。
  6. 変更を反映させるために、`sudo systemctl restart nginx` を実行し、NGINX を再起動します。
 
