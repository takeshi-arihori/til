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
terraform force-unlock {hash}
```

### 大まかな流れ
1. `terraform init`: リソース作成に必要なバイナリファイルをダウンロード
2. `terraform plan`: 実行計画が出力される
3. `terraform apply`: plan結果が表示され、`yes`を選択すると実行される
4. `terraform destroy`: リソースの削除



## 変数

- variable: 実行時に上書き可能(`-var`, `TF_VAR_example_instance_type=t3.nano terraform plan`など)
- locals: コマンド実行時に変更不可
- output: 出力値が定義できる(applyすると実行結果の最後に、作成されたインスタンスIDが出力される。)
- data: 外部データを参照できる。`filter`などを使って検索条件を指定し取得も可能
- provider: AWS, GCP, AzureなどのAPIの違いを吸収する役割(Terraform本体とは分離されているため`terraform init`でバイナリファイルをダウンロードする必要がある)


### 接続テスト
```main.tf
resource "aws_instance" "example" {
  ami                    = "ami-0c3fd0f5d33134a76"
  instance_type          = "t3.micro"
  vpc_security_group_ids = [aws_security_group.example_ec2.id]

  user_data = <<EOF
        #!/bin/bash
        yum install -y httpd
        systemctl start httpd.service
EOF
}

output "example_public_dns" {
    value = aws_instance.example.public_dns
}

resource "aws_security_group" "example_ec2" {
  name = "example_ec2"

  ingress {
    from_port   = 80
    to_port     = 80
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }

  egress {
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }
}

```

- 出力された `example_public_dns` にアクセスしてHTMLが返ってくれば成功
```
terraform apply

# 出力結果
Outputs:
example_public_dns = ec2.....
curl ec2.....
```

### モジュール
```tree
├── http_server
│   └── main.tf # モジュールを定義するファイル
├── main.tf # モジュールを利用するファイル
```
モジュール作成後、`terraform get` か `terraform init` コマンドでモジュールを事前に取得する必要がある。  
`terraform apply` で作成後、curl で接続確認。  






