## Docker環境での faker インストール手順

### 事前準備
- Dockerfileにcomposerのインストールを記述しておく
```
# Composer のインストール
RUN curl -sS https://getcomposer.org/installer -o composer-setup.php \
    && HASH="$(curl -sS https://composer.github.io/installer.sig)" \
    && echo "$HASH composer-setup.php" | sha384sum -c - \
    && php composer-setup.php --install-dir=/usr/local/bin --filename=composer \
    && rm composer-setup.php
```

### 手順1. 
1: PHPコンテナ内でFakerをインストール
まず、PHPコンテナに入ってFakerをインストールします。

```bash
docker compose exec app bash
```

2. プロジェクトのルートディレクトリに移動する:

```bash
cd /var/www/html
```

3. Fakerをインストール:

```bash
composer require fakerphp/faker
```

### 手順2: PHPファイルでFakerを使用
`src/index.php` ファイルを修正して、以下の内容を含めます。

```php
<?php
require_once 'vendor/autoload.php';

$faker = Faker\Factory::create();

echo $faker->name . "\n";
echo $faker->address . "\n";
echo $faker->text . "\n";
```

### 手順3: 確認
以下の手順でコンテナを再起動し、動作を確認します。

コンテナを停止し、削除する:

```bash
docker compose down
```
イメージをビルドして、コンテナをバックグラウンドで起動する:

```bash
docker compose up -d --build
```

PHPコンテナ内に再度入り、Fakerをインストールしたディレクトリに移動する:

```bash
docker compose exec app bash
cd /var/www/html
```

`src/index.php` ファイルを実行して、Fakerが正しく動作するか確認する:

```bash
php src/index.php
```
