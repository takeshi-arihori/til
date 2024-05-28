# ghのインストールと設定
本番環境（ここでは EC2 インスタンス）では、コマンドラインインターフェース（CLI）を使用してすべての作業を行うため、GitHub のウェブサイトを介して SSH キーを GitHub に追加することはできません。  
その代わりに、GitHub の公式 CLI ツールである「gh」を使用して、SSH キーを生成し、それを GitHub に追加します。gh は、ユーザーがターミナルやコマンドラインから直接 GitHub の機能を操作することを可能にします。

## gh のインストールと設定方法
1. gh をインストールします。
```
https://github.com/cli/cli/blob/trunk/docs/install_linux.md
```

2. 新しい SSH キーを生成し、それを ssh-agent に追加します.
```
https://docs.github.com/en/authentication/connecting-to-github-with-ssh/generating-a-new-ssh-key-and-adding-it-to-the-ssh-agent
```

3. コマンド gh auth login を実行して、gh で認証を行う。

認証はトークンを利用して行う。
```
https://github.com/settings/tokens
```
![gh-1](https://github.com/takeshi-arihori/til/assets/83809409/2f76966d-2684-4ca2-b9db-07f4d7a4ef6a)

例: チェック
- repo
- admin:org/read:org
- admin:public_key
