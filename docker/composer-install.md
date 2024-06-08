# Docker環境でのComposerインストール手順
ComposerはPHPのパッケージマネージャとして最も人気があります。以下の手順に従って、Docker環境でComposerをインストールします。

## Dockerfileの作成
docker/php/Dockerfile を作成し、以下の内容を記述します。
```Dockerfile
FROM php:8.1.18-fpm

# PHP 設定ファイルのコピー
COPY php.ini /usr/local/etc/php/

# 必要なパッケージのインストール
RUN apt-get update && apt-get install -y \
    git \
    curl \
    zip \
    unzip \
    && docker-php-ext-install pdo_mysql

# Composer のインストール
RUN curl -sS https://getcomposer.org/installer -o composer-setup.php \
    && HASH="$(curl -sS https://composer.github.io/installer.sig)" \
    && echo "$HASH composer-setup.php" | sha384sum -c - \
    && php composer-setup.php --install-dir=/usr/local/bin --filename=composer \
    && rm composer-setup.php

# 作業ディレクトリの設定
WORKDIR /var/www
```

## Docker Composeファイルの作成
docker-compose.yml ファイルに以下のサービスを定義します。

```yaml
services:
  nginx:
    image: nginx:1.25.0
    ports:
      - 8000:80
    volumes:
      - ./src:/var/www/html
      - ./docker/nginx:/etc/nginx/conf.d
    depends_on:
      - app
    networks:
      - project_network

  app:
    build:
      context: ./docker/php
      dockerfile: Dockerfile
    volumes:
      - ./src:/var/www/html
    env_file:
      - .env
    networks:
      - project_network

  mysql:
    image: mysql:8.0
    environment:
      MYSQL_ROOT_PASSWORD: ${MYSQL_ROOT_PASSWORD}
      MYSQL_DATABASE: ${MYSQL_DATABASE}
      MYSQL_USER: ${MYSQL_USER}
      MYSQL_PASSWORD: ${MYSQL_PASSWORD}
    volumes:
      - dynamic-web-server_mysqldata:/var/lib/mysql
      - ./docker/mysql/my.cnf:/etc/mysql/conf.d/my.cnf
    ports:
      - 3306:3306
    networks:
      - project_network

  phpmyadmin:
    image: phpmyadmin
    restart: always
    ports:
      - 8080:80
    environment:
      PMA_HOST: mysql
      PMA_PORT: 3306
      PMA_ARBITRARY: 1
    depends_on:
      - mysql
    networks:
      - project_network

volumes:
  dynamic-web-server_mysqldata:

networks:
  project_network:
    driver: bridge
```

## Docker環境の構築と実行
1. コンテナを停止し、削除する:

```bash
docker compose down
```

2. イメージをビルドして、コンテナをバックグラウンドで起動する:

```bash
docker compose up -d --build
```

3. Composerのインストール確認
PHPコンテナ内でComposerのバージョンを確認する:
```bash
docker compose exec app composer -V
```

このコマンドにより、Composerのバージョン情報が表示され、インストールが成功していることを確認できます。
