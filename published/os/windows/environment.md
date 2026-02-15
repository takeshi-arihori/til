# Laravel 開発環境構築ガイド (Windows11, SQLServer)

## 📋 環境構成概要

-   **OS**: Windows 11
-   **Web Server**: IIS (Internet Information Services) 10.0
-   **Database**: SQL Server 2022 Express (互換レベル 120 / SQL Server
    2014 相当で動作)
-   **Language**: PHP 8.3 (Non Thread Safe)
-   **Framework**: Laravel 12

------------------------------------------------------------------------

## 1. データベース構築 (SQL Server 2022)

### 1-1. インストールと初期設定

1.  **SQL Server 2022 Express** を「カスタム
    (Custom)」モードでインストール。
    -   インスタンス名: `SQLEXPRESS`
    -   認証モード: **混合モード**
    -   `sa` パスワード: `<強力なパスワード>`
2.  **TCP/IP の有効化とポート固定**
    -   TCP ポート: `1433`

### 1-2. データベース作成と互換性設定 (SSMS)

1.  `localhost\SQLEXPRESS` に接続。
2.  DB {project-name} を作成。
3.  互換性レベル: `SQL Server 2014 (120)` に設定。

### 1-3. アプリケーション用ユーザー作成

-   `test-user` を作成（Pass: `<強力なパスワード>`）
-   `db_owner` 付与。

------------------------------------------------------------------------

## 2. PHP 8.3 環境構築

### 2-1. PHP の配置

-   PHP 8.3 (NTS x64) を `C:\php\php8.3` に展開。
-   Path に追加。

### 2-2. 必須ドライバ

-   **ODBC Driver 17 for SQL Server**
-   `php_sqlsrv_83_nts_x64.dll`
-   `php_pdo_sqlsrv_83_nts_x64.dll` を `ext` へ配置。

### 2-3. php.ini

``` ini
extension_dir = "C:\php\php8.3\ext"
date.timezone = Asia/Tokyo
extension=curl
extension=fileinfo
extension=mbstring
extension=openssl
extension=pdo_mysql
extension=php_sqlsrv_83_nts_x64.dll
extension=php_pdo_sqlsrv_83_nts_x64.dll
```

### 2-4. Composer

-   `C:\php\php8.3\php.exe` を指定してインストール。

------------------------------------------------------------------------

## 3. アプリケーションセットアップ

``` powershell
mkdir C:\Projects
cd C:\Projects
git clone {project-name} 

cd {project-name}\src
composer install

cp .env.example .env
php artisan key:generate
php artisan migrate
```

------------------------------------------------------------------------

## 4. IIS 構築

### 4-1. 必須機能

-   Windows 機能 → **CGI** を有効化。

### 4-2. URL Rewrite

-   URL Rewrite 2.1 をインストール。

### 4-3. PHP ハンドラマッピング

-   FastCGI → `C:\php\php8.3\php-cgi.exe`
-   ハンドラ → `*.php` へ適用。

### 4-4. サイト作成

-   パス: `C:\Projects\{project-name}\src\public`
-   ポート: `8080`

### 4-5. パーミッション

-   `storage` / `bootstrap/cache` に書き込み権限付与。

------------------------------------------------------------------------

## 5. 動作確認

-   `http://localhost:8080` にアクセス。
