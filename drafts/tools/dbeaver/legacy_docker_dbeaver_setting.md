# Dockerで立ち上げたMySQLにDBeaverから接続する手順

## 前提条件
- Dockerがインストールされていること
- DBeaverがインストールされていること

## Dockerの設定内容
### 環境変数 `.env`
以下の環境変数を設定します。

```sh
MYSQL_ROOT_PASSWORD=secret
MYSQL_DATABASE=server-with-databases
MYSQL_USER=user
MYSQL_PASSWORD=secret
PROJECT_NAME=server-with-databases
```

### docker-compose.yml
以下の docker-compose.yml を使用します。

```yaml
services:
  nginx:
    image: nginx:1.25.0
    ports:
      - 8005:80
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
      - 3308:3306
    networks:
      - project_network

  phpmyadmin:
    image: phpmyadmin
    restart: always
    ports:
      - 8085:80
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

## 手順
#### 1. Dockerコンテナとボリュームの再作成
コンテナとボリュームを削除して再作成します。

```sh
docker-compose down -v
docker-compose up -d
```

#### 2. MySQLコンテナ内でデータベースの存在確認
MySQLコンテナに入り、データベースが存在するか確認します。

```sh
docker exec -it <mysql_container_name> mysql -u root -p
```

パスワード `secret` を入力し、以下のコマンドを実行します。

```sql
SHOW DATABASES;
```

`server-with-databases` がリストに表示されることを確認します。

#### 3. DBeaverの接続設定
DBeaverで以下の設定を行います。

```
Host: localhost
Port: 3308
Database: server-with-databases
Username: root
Password: secret
```

#### 接続手順
1. DBeaverを開きます。
2. Database メニューから New Database Connection を選択します。
3. MySQL を選択して Next をクリックします。
4. 接続設定画面で以下の情報を入力します。
```
Server Host: localhost
Port: 3308
Database: server-with-databases
Username: root
Password: secret
```

Test Connection ボタンをクリックして接続をテストします。
接続が成功したら、 Finish をクリックします。
