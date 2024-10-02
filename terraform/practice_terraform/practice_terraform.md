# 実践Terraform AWSにおけるシステム設計とベストプラクティス

## Terraformの設定
### クレデンシャル
```
export AWS_ACCESS_KEY_ID={アクセスキー}
export AWS_SECRET_ACCESS_KEY={シークレットアクセスキー}
export AWS_DEFAULT_REGION={リージョン}
```

### AWSアカウントの確認
```
aws sts get-caller-identity --query Account --output text
```


### tfenvを使用してTerraformをインストール
Terraformのバージョンマネージャで異なるバージョンのTerraformを簡単に扱える
```
brew install tfenv
tfenv --version
```

- 使い方: `list-remote`コマンドで、インストールできるバージョンの一覧を取得  
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

- git secretsnの設定
```
git secrets --register-aws --global
git secrets --install ~/.git-templates/git-secrets
git config --global init.templateDir ~/.git-templates/git-secrets
```  
[git secrets](https://github.com/awslabs/git-secrets)












