# Dockerfileを使用したPHP環境の構築

## directory
```
.
├── app
│   ├── src
│   │    └── index.php
│   └── Dockerfile
├── mysql
│   ├── data
│   │    └── 
│   ├── Dockerfile
│   └── init.sql
└── README.Docker.md
```
## Dockerfileのみでの立ち上げ
※ Dockerfileがある場所にdirectoryを移動

### 公式イメージ参照
- [PHP Docker 公式イメージ](https://hub.docker.com/_/php)

Image Variants で 好きなPHPイメージを作成できる
例: Apache with a Dockerfile
Dockerfile
```
FROM php:7.2-apache
COPY src/ /var/www/html/
```
`src`は全てのPHPコードを含むディレクトリ

### Dockerイメージをビルドして実行するためのコマンド
```
docker build -t my-php-app .
docker run -d --name my-running-app my-php-app
```
### 自動でマウントさせるために
php-app イメージを基に新しいコンテナを作成し、ホストの8080ポートをコンテナの80ポートにマッピングして、バックグラウンドで実行。
`-v "$(pwd)"/src:/var/www/html`がなければマッピングしない。開発環境でホスト側のコードをコンテナ内で実行する場合などは便利。
`docker run -p 8080:80 -d -v "$(pwd)"/src:/var/www/html --name php-container php-app`

## docker-composeを使用 (PHP・MySQLを設定)

Dockerfile
```
FROM php:8.2-apache
## コンテナ内でパッケージをインストールするコマンド
RUN apt update \
		&& docker-php-ext-install pdo_mysql
COPY src/ /var/www/html/
```

index.php
```
<?php

// 接続
// hostはコンテナ名を記載する
$dsn = 'mysql:dbname=test_db;host=run-php-db;';
$user = 'test';
$password = 'test';

try {
	$pdo = new PDO($dsn, $user, $password);
	$sth = $pdo->query("SELECT * FROM users WHERE id = 1");
	$user = $sth->fetch(PDO::FETCH_ASSOC);
	var_dump($user);
} catch (PDOException $e) {
	print('Error:' . $e->getMessage());
	exit;
}

```

mysql/Dockerfile
```
FROM mysql:8.0

# データベースを作成するsqlをdockerで実行
# docker-entrypoint-initdb.dにおくと実行してくれる
COPY init.sql /docker-entrypoint-initdb.d/init.sql

# コンテナ内の環境変数を設定
## ルートユーザのパスワード
ENV MYSQL_ROOT_PASSWORD=root
## ユーザ名
ENV MYSQL_USER=test
## ユーザのパスワード
ENV MYSQL_PASSWORD=test
## データベース名
ENV MYSQL_DATABASE=test_db

# ボリューム
VOLUME ./data:/var/lib/mysql
```

init.sql
```
CREATE TABLE test_db.users (
    id      INT             NOT NULL,
    first_name  VARCHAR(20)     NOT NULL,
    age         INT,
    PRIMARY KEY (id)
);

INSERT INTO `users` VALUES (1, 'Takeshi Arihori', 37)
```
### コンテナの立ち上げ
※ 各Dockerfileがある場所にdirectoryを移動

#### ネットワークを作成
```
docker network create php-mysql-network
```

#### php
- build(イメージ作成)
```
docker build -t php-app .
```
- run(コンテナ作成)
appディレクトリに移動して実行する！
```
docker container run \
-d \
--rm \
-p 8080:80 \
-v "$(pwd)"/src:/var/www/html \
--network php-mysql-network \
--name run-php-app php-app
```

#### mysql
- build(イメージ作成)
```
docker build -t php-db .
```

- run(コンテナ作成)
```
docker container run \
-d \
--rm \
--network php-mysql-network \
--name run-php-db php-db
```

