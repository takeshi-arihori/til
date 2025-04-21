# Docker環境でのDBサーバー接続エラー解決ガイド

## 発生したエラー
DBeaverからDocker上のMySQLに接続しようとした際に、以下のエラーが発生：
```
Communications link failure
The last packet sent successfully to the server was 0 milliseconds ago. The driver has not received any packets from the server.
```

## 原因
1. ポートマッピングの設定ミス
2. MySQLの初期化が完了していない
3. ネットワーク接続の問題
4. DBクライアントの設定問題

## 解決手順

### 1. コンテナの状態確認
```bash
# コンテナの状態を確認
docker compose ps

# DBコンテナのログを確認
docker compose logs db
```

### 2. ポートマッピングの確認
compose.yamlで正しく設定されているか確認：
```yaml
services:
  db:
    ports:
      - target: 3306
        published: 3307  # ホスト側のポート
```

### 3. ネットワーク接続テスト
```bash
# ポートが開いているか確認
nc -zv localhost 3307

# コンテナ内からの接続テスト
docker compose exec db mysql -u phper -psecret recursion
```

### 4. DBeaverの設定調整
- ホスト: `127.0.0.1`（`localhost`ではなく）
- ポート: `3307`（ホスト側のポート）
- データベース: `recursion`
- ユーザー名: `phper`
- パスワード: `secret`

ドライバープロパティで追加設定：
- `allowPublicKeyRetrieval`: `true`
- `useSSL`: `false`

### 5. コンテナ間の依存関係設定
compose.yamlに`depends_on`を追加：
```yaml
services:
  app:
    depends_on:
      - db
  web:
    depends_on:
      - app
```

### 6. パーミッションの確認（Laravelの場合）
```bash
docker compose exec app chmod -R 775 storage bootstrap/cache
docker compose exec app chown -R www-data:www-data storage bootstrap/cache
```

## 重要なポイント
1. コンテナ内部とホストマシンでのポート番号の違いを理解する
2. MySQLの初期化が完了するまで待つ
3. ファイアウォールの設定を確認する
4. DBクライアントの接続設定を正確に行う

## まとめ
Docker環境でのDB接続問題の多くは、ポートマッピングと接続設定の不一致が原因です。適切な設定とトラブルシューティング手順に従うことで、ほとんどの問題を解決できます。
