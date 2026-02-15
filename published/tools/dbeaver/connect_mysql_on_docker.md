# Docker上のMySQLにDBeaverから接続する手順

## 前提
- Docker / Docker Compose が利用できる
- MySQL コンテナが起動している
- DBeaver がインストール済み

## 1. コンテナとポート公開の確認
```bash
docker compose ps
docker compose logs db
```

`db` サービスで `ホスト側ポート:3306` の公開があることを確認します。

例:
```yaml
services:
  db:
    ports:
      - "3307:3306"
```

## 2. MySQL 側の確認
```bash
docker compose exec db mysql -u root -p
```

MySQL内で:
```sql
SHOW DATABASES;
```

## 3. DBeaver 接続設定
- Host: `127.0.0.1`
- Port: `3307`（compose の published 側）
- Database: `<利用DB名>`
- Username: `<利用ユーザー>`
- Password: `<利用パスワード>`

## 4. 接続エラー時のチェック
### Communications link failure
- コンテナが `Up` になっているか
- 公開ポートが競合していないか
- MySQL初期化完了前に接続していないか

### Public Key Retrieval is not allowed
DBeaver の Driver Properties で以下を設定:
- `allowPublicKeyRetrieval=true`
- `useSSL=false`

## 5. 実運用での注意
- `root` ではなく専用ユーザーで接続する
- パスワードを固定値で記事に残さない
- 開発環境でも不要なポート公開は避ける

## 関連
- Docker側の詳細例: `published/infrastructure/docker/laravel_install_mysql.md`
- 旧版メモ: `drafts/tools/dbeaver/legacy_db_dbeaver.md`, `drafts/tools/dbeaver/legacy_docker_dbeaver_setting.md`
