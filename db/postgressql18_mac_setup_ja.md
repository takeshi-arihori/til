# Mac Intel + Homebrewで PostgreSQL 18 を初期化し直して、ja_JP.UTF-8 & localhost運用に整える手順

## はじめに

Mac（Intel）でHomebrew経由のPostgreSQL 18を使っていると、初期化直後に次のようなエラーでつまずくことがあります。

- `psql` 実行時: `FATAL: database "<OSユーザー名>" does not exist`
- `psql -U postgres` 実行時: `FATAL: role "postgres" does not exist`

今回の目的は次の3つです。

1. クラスタをクリーン初期化する  
2. `US` ロケールから `ja_JP.UTF-8` へ変更する  
3. セキュリティ向上のため `listen_addresses='localhost'` を設定する

---

## 1. まず原因を整理する

`psql` は、接続先を省略すると「OSユーザー名」をDBユーザー名に使い、さらに同名DBをデフォルトDB名として使います。  
そのため、同名DBがないと `database "<username>" does not exist` になります。

また、`createdb` は**シェルコマンド**です。  
`postgres=#`（psqlのSQLプロンプト）内で `createdb "$USER"` を打つとSQLとして解釈され、構文エラーになります。psql内では `CREATE DATABASE ...;` を使います。

---

## 2. `.zshrc` のPATHを修正（閉じクォート漏れ注意）

誤り例（閉じ `"` がない）:

```zsh
export PATH="/usr/local/sbin:$PATH
```

修正例:

```zsh
export PATH="/usr/local/opt/postgresql@18/bin:$PATH"
export PATH="/usr/local/sbin:$PATH"
```

反映:

```bash
source ~/.zshrc
```

---

## 3. PostgreSQLクラスタを初期化し直す（全DB削除）

> ⚠️ この手順は既存DBをすべて消します。

```bash
brew services stop postgresql@18
rm -rf "$(brew --prefix)/var/postgresql@18"

initdb -D "$(brew --prefix)/var/postgresql@18" \
  --locale=ja_JP.UTF-8 \
  --encoding=UTF8 \
  --username=postgres \
  --auth-local=scram-sha-256 \
  --auth-host=scram-sha-256

brew services start postgresql@18
```

### 補足
`initdb` で認証方式を指定しない場合、ローカル認証に `trust` が使われることがあります。  
セキュリティを意識するなら `--auth-local/--auth-host` を明示するのがおすすめです。

---

## 4. 初期化後の接続確認

```bash
psql -U postgres -d postgres -c "\l"
locale -a | grep -i ja_JP
```

`Collate/Ctype` が `ja_JP.UTF-8` で、`postgres` ロール/DBが見えていればOKです。

---

## 5. `psql` の基本（`\c` / `\q` / `\r`）

運用時によく使うメタコマンド:

- `\c`（または `\connect`）: 接続先DB/ユーザーを切り替える  
- `\q`（または `\quit`）: psqlを終了する  
- `\r`（または `\reset`）: 入力中のクエリバッファをクリアする  

例:

```sql
\c postgres postgres
\q
```

---

## 6. セキュリティ: `listen_addresses='localhost'` は有効？

結論: **ローカル運用が前提なら有効**です。  
`listen_addresses` を `localhost` にすると、ループバック以外のNICからの接続受付を避けられます。  
このパラメータは**サーバー起動時のみ反映**なので、変更後は再起動が必要です。

### 変更するファイル

基本的に編集対象は:

1. `postgresql.conf`（`listen_addresses`）
2. `pg_hba.conf`（認証ルール）

パス確認:

```sql
SHOW config_file;
SHOW hba_file;
```

### 例: `postgresql.conf`

```conf
listen_addresses = 'localhost'
password_encryption = 'scram-sha-256'
```

### 例: `pg_hba.conf`（ローカル限定）

```conf
local   all             postgres                                peer
local   all             all                                     scram-sha-256
host    all             all             127.0.0.1/32            scram-sha-256
host    all             all             ::1/128                 scram-sha-256
```

### 反映方法

- `listen_addresses` 変更 → **再起動**
- `pg_hba.conf` 変更のみ → `SELECT pg_reload_conf();` で再読込可

---

## 7. 最終チェックコマンド

```bash
brew services list
psql -U postgres -d postgres -c "SHOW listen_addresses;"
psql -U postgres -d postgres -c "SHOW hba_file;"
psql -U postgres -d postgres -c "\l"
```

---

## まとめ

- `psql` の「DBがない」エラーは、**デフォルト接続先（OSユーザー名DB）**が主因になりやすい
- `createdb` は**psqlの外（シェル）**で使う
- `initdb` をやり直すなら、`--locale` / `--username` / `--auth-*` まで最初に決める
- セキュリティ重視なら `listen_addresses='localhost'` + `pg_hba.conf` を明示設定
