UbuntuでLaravel 11とMySQL 8.0の環境構築手順
開発環境
VirtualBox 7.0
Ubuntu 24.04
Laravel 11
PHP 8.3
MySQL 8.0
必要な設定
Laravelをインストールするために以下のソフトウェアと設定が必要です。

PHP（最低バージョン8.2以上）
PHP拡張モジュール
Composer
Node.js
npm
MySQL
手順
1. PHPとPHP拡張モジュールのインストール
コマンド実行
bash
コードをコピーする
sudo apt update && sudo apt upgrade -y
sudo add-apt-repository ppa:ondrej/php
sudo apt update
sudo apt-get install -y php php-cli php-common php-fpm php-mysql php-zip php-gd php-mbstring php-curl php-xml php-bcmath openssl php-json php-tokenizer
インストールされるパッケージ
php：最新のPHPバージョン
php-cli：PHPのコマンドラインインターフェース
php-common：PHPの共通ファイル
php-fpm：PHP FastCGIプロセスマネージャー
php-mysql：PHPのMySQLデータベースサポート
php-zip：ZIPアーカイブサポート
php-gd：画像処理拡張（GDライブラリ）
php-mbstring：マルチバイト文字列サポート
php-curl：cURL拡張
php-xml：XMLサポート
php-bcmath：任意精度数学関数サポート
openssl：OpenSSLライブラリ
php-json：JSONサポート
php-tokenizer：トークナイザーサポート
インストールの確認
bash
コードをコピーする
php -v   # PHPのバージョン確認
php -m   # PHPの拡張モジュール一覧を表示
2. Composerのインストール
コマンド実行
bash
コードをコピーする
sudo apt-get install composer
インストールの確認
bash
コードをコピーする
composer --version   # Composerのバージョン確認
3. MySQLのインストールと設定
手順
MySQL 8のインストールとサービスの開始

bash
コードをコピーする
sudo apt install mysql-server
sudo service mysql start
rootユーザーのパスワード設定と認証方式の変更

bash
コードをコピーする
sudo mysql
MySQLコンソールで以下を実行:

sql
コードをコピーする
ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY 'password';
exit;
rootユーザーで再度ログインし、auth_socket認証を設定

bash
コードをコピーする
sudo mysql -u root -p
パスワードを入力後、以下を実行:

sql
コードをコピーする
ALTER USER 'root'@'localhost' IDENTIFIED WITH auth_socket;
exit;
MySQLのセキュリティ設定

bash
コードをコピーする
sudo mysql_secure_installation
Laravel用ユーザーの作成
rootユーザーでMySQLにログイン

bash
コードをコピーする
mysql -u root -p
ユーザーの作成と権限の付与

sql
コードをコピーする
CREATE USER 'laravel'@'localhost' IDENTIFIED WITH mysql_native_password BY 'password@123';
GRANT CREATE, ALTER, DROP, INSERT, UPDATE, INDEX, DELETE, SELECT, REFERENCES, RELOAD ON *.* TO 'laravel'@'localhost' WITH GRANT OPTION;
FLUSH PRIVILEGES;
exit;
4. Node.jsとnpmのインストール
cURLのインストール
bash
コードをコピーする
sudo apt install curl
NVM（Node Version Manager）のインストール
bash
コードをコピーする
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.7/install.sh | bash
NVMを有効化
bash
コードをコピーする
export NVM_DIR="$([ -z "${XDG_CONFIG_HOME-}" ] && printf %s "${HOME}/.nvm" || printf %s "${XDG_CONFIG_HOME}/nvm")"
[ -s "$NVM_DIR/nvm.sh" ] && \. "$NVM_DIR/nvm.sh"
既存のNode.jsとnpmの削除（必要に応じて）
bash
コードをコピーする
sudo apt purge nodejs npm
インストールの確認
bash
コードをコピーする
nvm -v   # NVMのバージョン確認
node -v  # Node.jsのバージョン確認
npm -v   # npmのバージョン確認
5. Laravelプロジェクトの作成
Laravelインストーラーのグローバルインストール
bash
コードをコピーする
composer global require laravel/installer
ComposerのbinディレクトリをPATHに追加
bash
コードをコピーする
composer global config bin-dir --absolute
echo 'export PATH="$PATH:$HOME/.config/composer/vendor/bin"' >> ~/.bashrc
source ~/.bashrc
Laravelのバージョン確認
bash
コードをコピーする
laravel --version
プロジェクトの作成
bash
コードをコピーする
laravel new <プロジェクト名>
プロジェクト作成時に以下を選択します。

Starter Kit: Laravel Breeze
Breeze Stack: Blade with Alpine
Dark Mode Support: No
Testing Framework: PHPUnit
Initialize a git repo: No
Database application will use: MySQL
Default database updated, run default database migration?: No
6. .envファイルの設定
プロジェクトディレクトリの.envファイルを編集し、データベース接続情報を設定します。

env
コードをコピーする
DB_CONNECTION=mysql
DB_HOST=127.0.0.1
DB_PORT=3306
DB_DATABASE=<データベース名>
DB_USERNAME=laravel
DB_PASSWORD=password@123
7. データベースの作成
bash
コードをコピーする
sudo mysql -u root -p
MySQLコンソールで以下を実行:

sql
コードをコピーする
CREATE DATABASE <データベース名> CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
exit;
8. 開発サーバーの起動とアセットのコンパイル
Laravelの開発サーバーを起動
bash
コードをコピーする
php artisan serve
フロントエンドのアセットをコンパイル
bash
コードをコピーする
npm install
npm run dev
