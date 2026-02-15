# Ubuntu 24にMySQL 8.xをインストールし、DBeaverで接続する方法

## 環境
```
VirtualBox: 7.0
Ubuntu 24.04
MySQL: 8.3
```

## はじめに
このガイドでは、Ubuntu 24にMySQL 8.0をインストールし、DBeaverでtakeshiユーザーを使用してデータベース管理を行うための設定方法について説明します。

### 1. MySQL 8.0のリポジトリを追加
まず、MySQL 8.0の公式リポジトリを追加します。packageのダウンロードが完了している時点からスタート。

```bash
sudo dpkg -i /PATH/mysql-apt-config_0.8.30-1_all.deb
```

※ 公式リポジトリの追加については以下を参照
- [MySQL APTリポジトリの追加(Oracle公式)](https://dev.mysql.com/doc/mysql-apt-repo-quick-guide/en/#apt-repo-fresh-install)
- [packageのダウンロード](https://dev.mysql.com/downloads/repo/apt/)

### 2. パッケージリストの更新とMySQLのインストール
リポジトリを追加した後、パッケージリストを更新し、MySQL 8.0をインストールします。

```bash
sudo apt-get update
sudo apt-get install mysql-server
```

### 3. MySQLサービスの起動
インストールが完了したら、MySQLサービスを起動します。

```bash
sudo systemctl start mysql
```
サービスが正しく起動していることを確認します。

```bash
sudo systemctl status mysql
```

### 4. MySQLの自動起動設定
MySQLがシステム起動時に自動的に開始されるように設定します。

```bash
sudo systemctl enable mysql
```

### 5. MySQLのポート番号の確認
MySQLはデフォルトでポート番号3306を使用しますが、確認しておくことをお勧めします。

設定ファイルでの確認
```bash
sudo grep 'port' /etc/mysql/mysql.conf.d/mysqld.cnf
```

MySQLクライアントでの確認
```sql
SHOW VARIABLES LIKE 'port';
```

### 6. ユーザーの作成と権限設定
まず、MySQLにrootユーザーでログインし、takeshiユーザーを作成し、権限を設定します。
ここでは `takeshi` を設定したいと思います。

```bash
sudo mysql -u root -p
```

以下のSQLコマンドを実行します。

```sql
CREATE DATABASE testdb;
CREATE USER 'takeshi'@'localhost' IDENTIFIED BY '<強力なパスワード>';
GRANT ALL PRIVILEGES ON testdb.* TO 'takeshi'@'localhost';
FLUSH PRIVILEGES;
```

### 7. DBeaverでの接続設定
DBeaverでtakeshiユーザーとして接続を行います。

#### 手順
1. DBeaverを開く。
2. 新しい接続を作成するために「Database」メニューから「New Database Connection」を選択。
3. MySQLを選択し、次へ進む。
4. 接続設定を以下のように入力：
```
Server Host: MySQLサーバーのIPアドレスまたはホスト名（ローカルホストの場合は127.0.0.1）
Port: 3306
Database: testdb
Username: takeshi
Password: <強力なパスワード>
```
5. ドライバーのプロパティで、allowPublicKeyRetrieval: trueに設定。
6. Test Connectionボタンを押して接続を確認。

### 8. データベースの作成
DBeaverからtakeshiユーザーで接続した後、新しいデータベースを作成します。

#### 手順
1. DBeaverで接続したMySQLインスタンスを右クリック。
2. **"Create" -> "Database..."**を選択。
3. データベース名を入力し、OKをクリック。
例えば、testdbという名前のデータベースを作成します。

```sql
CREATE DATABASE testdb;
```

以上で、Ubuntu 24にMySQL 8.xをインストールし、DBeaverで`takeshi`ユーザーを使用してデータベースを作成する方法が完了しました。
