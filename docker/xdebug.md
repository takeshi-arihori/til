# Docker 環境での Xdebug 3 設定とデバッグの問題解決

## はじめに
この記事では、Docker 環境で PHP の Xdebug 3 を設定し、デバッグが正常に動作しない問題を解決する方法を解説します。この記事を参考にすれば、Xdebug を使ったデバッグ環境を再現できます。

## 環境
- **OS**: macOS
- **PHP バージョン**: 8.3
- **Xdebug バージョン**: 3.x
- **Docker**: 使用
- **エディタ**: Visual Studio Code

## 問題
Xdebug 3 を設定しても、以下のような問題が発生することがあります。

1. ブラウザでリクエストを送信しても、設定したブレークポイントで停止しない。
2. 以下のようなエラーメッセージが表示される。

```
Xdebug: [Config] The setting 'xdebug.remote_autostart' has been renamed, see the upgrading guide at https://xdebug.org/docs/upgrade_guide#changed-xdebug.remote_autostart
Xdebug: [Config] The setting 'xdebug.remote_enable' has been renamed, see the upgrading guide at https://xdebug.org/docs/upgrade_guide#changed-xdebug.remote_enable
```

## 解決策

### 1. 必要なファイル構成
以下のファイル構成を用意します。

```
project/
├── compose.yaml
├── docker/
│   ├── Dockerfile
│   └── php.ini
├── src/
│   └── index.php
```

### 2. 各ファイルの内容

#### **compose.yaml**
Docker Compose の設定ファイルです。

```yaml
services:
  php:
    build:
      context: ./docker
    ports:
      - "8081:80"
    volumes:
      - ./src:/var/www/html
      - ./docker/php.ini:/usr/local/etc/php/php.ini
```

#### **docker/Dockerfile**
PHP と Xdebug をインストールするための Dockerfile です。

```dockerfile
FROM php:8.3-apache-bullseye

RUN pecl install xdebug \
  && docker-php-ext-enable xdebug \
  && echo "zend_extension=$(find /usr/local/lib/php/extensions/ -name xdebug.so)" > /usr/local/etc/php/conf.d/xdebug.ini

WORKDIR /var/www
```

#### **docker/php.ini**
Xdebug の設定ファイルです。

```ini
[xdebug]
xdebug.mode=debug
xdebug.start_with_request=yes
xdebug.client_host=host.docker.internal
xdebug.client_port=9001
xdebug.log=/tmp/xdebug.log
```

#### **src/index.php**
デバッグ対象の PHP ファイルです。

```php
<?php 

$x = 2;

$x += 300;

echo $x . PHP_EOL;
```

### 3. Xdebug の設定確認

#### **1. コンテナ内の設定ファイルを確認**
コンテナ内で Xdebug の設定が正しく反映されているか確認します。

```bash
docker compose exec php php --ini
```

出力例:
```
Configuration File (php.ini) Path: /usr/local/etc/php
Loaded Configuration File:         /usr/local/etc/php/php.ini
Scan for additional .ini files in: /usr/local/etc/php/conf.d
Additional .ini files parsed:      /usr/local/etc/php/conf.d/docker-php-ext-xdebug.ini
```

#### **2. 古い設定の削除**
`/usr/local/etc/php/conf.d/docker-php-ext-xdebug.ini` に古い設定が含まれている場合、以下のコマンドで削除します。

```bash
docker compose exec php rm /usr/local/etc/php/conf.d/docker-php-ext-xdebug.ini
```

### 4. コンテナの再起動
設定を反映させるため、コンテナを再起動します。

```bash
docker compose down
docker compose up --build
```

---

### 5. 動作確認
以下のコマンドで Xdebug が有効になっているか確認します。

```bash
docker compose exec php php -m | grep xdebug
```

出力例:
```
xdebug
```

## 結果
Xdebug が正常に動作し、ブレークポイントで停止するようになります。これでデバッグ環境が構築できました。

## 補足
- **VSCode の設定**: `.vscode/launch.json` を以下のように設定してください。

```jsonc
{
    "version": "0.2.0",
    "configurations": [
        {
            "name": "Listen for Xdebug",
            "type": "php",
            "request": "launch",
            "port": 9001,
            "pathMappings": {
                "/var/www/html": "${workspaceFolder}/src"
            }
        }
    ]
}
```

- **ブラウザキャッシュ**: ブラウザのキャッシュが原因でブレークポイントが効かない場合があります。キャッシュをクリアして再試行してください。
