# DBeaverでDocker上のMySQLに接続する方法

この記事では、Dockerで構築したMySQLデータベースにDBeaverを使って接続する手順を説明します。

## 前提条件
- Dockerがインストールされていること
- MySQLコンテナがDocker上で稼働していること
- DBeaverがインストールされていること

```
Docker version 26.1.1
Docker Compose version v2.27.0-desktop.2
```

## 1. Dockerコンテナの確認

まず、Docker上でMySQLコンテナが正常に稼働していることを確認します。

```sh
docker-compose ps

NAME                                  IMAGE                        COMMAND                   SERVICE      CREATED        STATUS                    PORTS
docker-laravel-handson-db-1           docker-laravel-handson-db    "/entrypoint.sh mysq…"   db           13 hours ago   Up 23 minutes (healthy)   33060-33061/tcp, 0.0.0.0:33060->3306/tcp
```

## 2. DBeaverでのMySQL接続設定
**接続設定**

1. DBeaverを開き、「新しい接続」をクリックします。
2. MySQLを選択し、「次へ」をクリックします。

<img width="1250" alt="スクリーンショット 2024-05-31 12 11 31" src="https://github.com/takeshi-arihori/til/assets/83809409/7d1460f0-2ed6-4fab-a94c-bb2ea72d4d2e">

3. 接続設定を以下のように行います：
MySQLで設定されている内容
- .env
```
DB_CONNECTION=mysql
DB_HOST=db
DB_PORT=3306
DB_DATABASE=laravel
DB_USERNAME=phper
DB_PASSWORD=secret
```

- compose.yml
```
  db:
    build: ./infra/mysql
    volumes:
      - db-store:/var/lib/mysql
    ports:
      - "33060:3306"
```

- DBeaverに入力する内容
```
ホスト: localhost
ポート: 33060 # ホストマシンからの接続するport
データベース: laravel
ユーザー: phper
パスワード: secret
```

  
<img width="1121" alt="スクリーンショット 2024-05-31 12 15 18" src="https://github.com/takeshi-arihori/til/assets/83809409/9e4e4d4e-d9c6-46e8-b1fa-8c0f7c8e8b38">  

4. Test Connectionをクリックし、接続が成功することを確認します。


## エラー解決
#### Public Key Retrieval is not allowedエラー


<img width="1034" alt="スクリーンショット 2024-05-31 12 30 16" src="https://github.com/takeshi-arihori/til/assets/83809409/52dea579-5a6f-4f30-8d69-0484ee043286">


    
このエラーが発生した場合、以下の手順で解決できます。

- Driver Propertiesタブで、allowPublicKeyRetrievalプロパティを追加し、値をtrueに設定します。


<img width="1106" alt="スクリーンショット 2024-05-31 12 31 14" src="https://github.com/takeshi-arihori/til/assets/83809409/117e0be5-fbe2-498c-9758-ff47c6d27fab">


再度接続を試みます。  

<img width="1034" alt="スクリーンショット 2024-05-31 12 32 59" src="https://github.com/takeshi-arihori/til/assets/83809409/28d9d5bd-83df-4d5e-b10e-06f600d29bfc">


<img width="760" alt="スクリーンショット 2024-05-31 12 34 14" src="https://github.com/takeshi-arihori/til/assets/83809409/96e5a1f1-a92a-480c-b62f-8728e232f052">
