# Laravel Sail で Tinker時の文字化け解消

### 課題

- Laravel Sail環境で`php artisan tinker`を使った際の文字化けを解消したい。


## なぜ文字化けするのか

Laravel Sailを使うと、Docker環境内でPHP・MySQLなどが動作します。  
その際、デフォルトのMySQLやターミナルのロケール設定がUTF-8ではない場合、特に日本語などのマルチバイト文字を扱う時に文字化けが発生します。  
Tinkerは、アプリケーション内部のテストやモデルファクトリで日本語を出力すると、MySQLのデフォルトエンコーディングやクライアントの文字コード設定が不十分な場合、ターミナル上で正常に表示されません。  

## 解決

### 1. **Sail の Dockerfileを編集する準備**

 Laravel Sailでカスタム設定を反映するには、デフォルトのSail構成ファイルをパブリッシュします。  

 ```zsh
 sail artisan sail:publish
 ```

これでdocker/ディレクトリ以下にDocker用の設定ファイル群が生成され、自由にカスタマイズできます。

### 2. MySQL 設定ファイル(my.cnf)の追加

docker/使用中のdockerのversion/ ディレクトリ（例：docker/8.4/）内に my.cnf ファイルを作成し、以下の内容を記述します。

```zsh
[mysqld]
character-set-server=utf8mb4
collation-server=utf8mb4_general_ci
explicit-defaults-for-timestamp=1
general-log=1
general-log-file=/var/log/mysql/mysqld.log

[client]
default-character-set=utf8mb4

```

この設定により、MySQLサーバーとクライアントとのやり取りに常にutf8mb4が使用され、文字化けが防止できます。

### 3. docker-compose.yml の volumes に追記

docker-compose.yml のMySQLサービス定義部分にmy.cnfをマウントする設定を追加します。
例として、MySQLサービスが mysql という名前の場合：

```zsh
services:
  mysql:
    # ...その他設定...
    volumes:
      - './docker/8.4/my.cnf:/etc/my.cnf'

```

これでコンテナ起動時にMySQLが上記設定を反映するようになります。

### 4. 再ビルド・再起動

設定を反映させるため、Dockerイメージの再ビルドとコンテナの再起動を行います。
```zsh
sail build --no-cache
sail up -d
```

### 5. mysqlの確認
`sail mysql` でmysqlにログインし確認

**結果**  
```zsh
mysql> SHOW VARIABLES LIKE 'character%';
+--------------------------+--------------------------------+
| Variable_name            | Value                          |
+--------------------------+--------------------------------+
| character_set_client     | utf8mb4                        |
| character_set_connection | utf8mb4                        |
| character_set_database   | utf8mb4                        |
| character_set_filesystem | binary                         |
| character_set_results    | utf8mb4                        |
| character_set_server     | utf8mb4                        |
| character_set_system     | utf8mb3                        |
| character_sets_dir       | /usr/share/mysql-8.0/charsets/ |
+--------------------------+--------------------------------+
8 rows in set (0.01 sec)
```


再度、`php artisan tinker`を実行し、日本語文字列が正しく表示されるか確認してください。

### まとめ
MySQLとクライアント間の文字コード設定がUTF-8でなかったため、Tinker上での日本語が文字化けしていました。  
my.cnfでutf8mb4を指定し、Docker Composeで適切なマウント設定を行うことで、文字化けを解消できます。  
これらの手順を踏むことで、Laravel Sail環境でのTinker実行時に日本語が正常に表示され、開発効率やデバッグの快適さが向上します。  

