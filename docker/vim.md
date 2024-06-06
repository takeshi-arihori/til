# Dockerコンテナ内でVimをインストールして設定する方法

Dockerコンテナ内で開発を行っている際、Vimなどのエディタがインストールされていない場合があります。本記事では、Dockerコンテナ内でVimをインストールし、インデントスタイルをVSCodeと一致させる方法を説明します。

## 手順1: コンテナIDもしくはコンテナ名を確認

まず、対象のDockerコンテナの確認を行います。
`docker ps` または `docker ps -a` で確認するとコンテナ名もしくはコンテナIDを確認することができます。

<img width="1255" alt="スクリーンショット 2024-06-06 10 03 54" src="https://github.com/takeshi-arihori/til/assets/83809409/b13dc524-6d59-49bf-a9f6-48c2b2e76a80">


## 手順2: コンテナにrootユーザーでアクセスする
コンテナにrootユーザーでアクセスするには、以下のコマンドを使用します。

```sh
docker exec -it -u root <コンテナ名> /bin/bash
```

## 手順3: Vimをインストールする
rootユーザーでコンテナ内に入ったら、以下のコマンドを実行してVimをインストールします。

Debian/Ubuntuベースの場合:
```sh
apt-get update
apt-get install vim -y
```

## 手順4: Vimの設定を行う
Vimをインストールした後、以下の手順で設定を行います。

Vimの設定ファイルを確認・編集
Vimの設定は通常、/root/.vimrcファイルで行います。このファイルが存在しない場合は作成します。

```sh
vim /root/.vimrc
```


