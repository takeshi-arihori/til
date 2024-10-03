# 実践Terraform AWSにおけるシステム設計とベストプラクティス

## Terraformの設定
### クレデンシャル
```
export AWS_ACCESS_KEY_ID={アクセスキー}
export AWS_SECRET_ACCESS_KEY={シークレットアクセスキー}
export AWS_DEFAULT_REGION={リージョン} ap-northeast-1
```

### AWSアカウントの確認
```
aws sts get-caller-identity --query Account --output text
aws sts get-caller-identity
```


### tfenvを使用してTerraformをインストール(Mac)
Terraformのバージョンマネージャで異なるバージョンのTerraformを簡単に扱える
```
brew install tfenv
tfenv --version
```
### tfenvを使用してTerraformをインストール(Windows WSL2)

- 任意のパスにチェックアウト
```
git clone --depth=1 https://github.com/tfutils/tfenv.git ~/.tfenv
```

- bashのpathに追加
```
echo 'export PATH=$PATH:$HOME/.tfenv/bin' >> ~/.bashrc
```

- シンボリックリンクの設定
```
ln -s ~/.tfenv/bin/* /usr/local/bin
```


### 使い方
`list-remote`コマンドで、インストールできるバージョンの一覧を取得  
```
tfenv list-remote
```

- `install`コマンドで指定したバージョンをインストール(複数のバージョンをインストール可能)
```
tfenv install 0.12.5
tfenv list
```

- `use`コマンドでバージョンを切り替える
```
tfenv use 0.12.5
```

- チーム開発時は`.terraform-version`ファイルをリポジトリに含める
```
echo 0.12.5 > .terraform-version
```
このファイルにバージョンを記述すると、チームメンバーが `tfenv install` コマンドを実行するだけでバージョンを統一できる。

- git secretsの設定
```
git secrets --register-aws --global
git secrets --install ~/.git-templates/git-secrets
git config --global init.templateDir ~/.git-templates/git-secrets
```  
[git secrets](https://github.com/awslabs/git-secrets)


## 基本操作

### terraformのlockがかかった時の対処法
- [WSL2 からterraform apply やterraform destory 実行時に固まる時の対処法](https://www.kdkwakaba.com/articles/how-to-fix-terraform-not-working-when-exe-it)

- 実行中のプロセスの確認
```
ps aux | grep terraform
kill -9 8285 8301 8369 // プロセスの削除
```

- ロックの強制解除
```
terraform force-unlock c6aada90-1f0a-c8d5-2c95-6614e360adbd
```










