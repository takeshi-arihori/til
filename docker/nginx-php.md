
# Docker × NginxでPHPディレクトリを効率的に運用する設定方法

## 背景
Nginxを使用して、特定のディレクトリ（例: `selfphp`）内のPHPスクリプトを適切に表示できるようにするには、`nginx`の設定が重要です。本記事では、`http://localhost:8832/selfphp` でディレクトリ内の `index.php` をデフォルトで表示する方法を紹介します。

---

## 必要な環境
- Docker
- Docker Compose
- Nginx
- PHP (FastCGI)
- MySQL（必要に応じて）

---

## 設定内容

### 1. `default.conf`の修正
以下の設定を `default.conf` に適用します。この設定により、`selfphp` ディレクトリにアクセスした際に、自動的に `index.php` を表示するようになります。

#### 修正版 `default.conf`
```nginx
server {
    listen 80;
    root /var/www/html;
    index main.php index.php;

    # Default PHP processing (for root directory)
    location ~ \.php$ {
        fastcgi_split_path_info ^(.+?\.php)(/.*)$;
        if (!-f $document_root$fastcgi_script_name) {
            return 404;
        }
        fastcgi_param HTTP_PROXY "";
        fastcgi_pass app:9000;
        fastcgi_index main.php;
        include fastcgi_params;
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
    }

    # Configuration for /selfphp/ directory
    location /selfphp/ {
        alias /var/www/html/selfphp/;
        index index.php;
        try_files $uri $uri/ /index.php?$query_string;

        location ~ \.php$ {
            fastcgi_split_path_info ^(.+?\.php)(/.*)$;
            fastcgi_param HTTP_PROXY "";
            fastcgi_pass app:9000;
            fastcgi_index index.php;
            include fastcgi_params;
            fastcgi_param SCRIPT_FILENAME /var/www/html/selfphp$fastcgi_script_name;
        }
    }
}
```

### 2. Docker Compose の確認
以下は `compose.yml` の設定例です。この設定を基に環境を構築してください。

```yaml
services:
  nginx:
    image: nginx:1.25.0
    ports:
      - 8832:80
    volumes:
      - ./src:/var/www/html
      - ./docker/nginx/default.conf:/etc/nginx/conf.d/default.conf
    depends_on:
      - app
    networks:
      - dynamic-web-server
  app:
    build:
      context: ./docker/php
      dockerfile: Dockerfile
    volumes:
      - ./src:/var/www/html
    env_file:
      - .env
    networks:
      - dynamic-web-server

  mysql:
    image: mysql:8.0
    environment:
      MYSQL_ROOT_PASSWORD: ${MYSQL_ROOT_PASSWORD}
      MYSQL_DATABASE: ${MYSQL_DATABASE}
      MYSQL_USER: ${MYSQL_USER}
      MYSQL_PASSWORD: ${MYSQL_PASSWORD}
    volumes:
      - dynamic-webserver-db:/var/lib/mysql
      - ./docker/mysql/my.cnf:/etc/mysql/conf.d/my.cnf
    ports:
      - 33061:3306
    networks:
      - dynamic-web-server

volumes:
  dynamic-webserver-db:

networks:
  dynamic-web-server:
    driver: bridge
```

---

## 手順

### 1. Nginxの設定ファイルを更新
プロジェクトの`docker/nginx/default.conf`を修正し、上記の設定を適用します。

### 2. Docker Composeで再構築
修正を反映するために、以下のコマンドを実行してコンテナを再構築します。

```bash
docker-compose up --build
```

### 3. 動作確認
ブラウザで以下のURLにアクセスして動作を確認します。

- `http://localhost:8832/selfphp`
  - → ディレクトリ内の `index.php` が自動的に表示される。

---

## まとめ
本設定では、特定のディレクトリ（`selfphp`）にアクセスした際、自動的に `index.php` が補完されるようになっています。Nginxの`alias`と`try_files`を組み合わせることで、柔軟かつ簡潔にPHPファイルのルーティングが可能です。

Docker環境を活用することで、再現性のある環境構築が実現できます。ぜひプロジェクトで活用してみてください！
