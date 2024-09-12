# ECSとCI/CDパイプライン
**GitHub Actionを使用**  

![learn-ecs-開発の流れ drawio](https://github.com/user-attachments/assets/b9feee9e-dde6-4646-be46-04bc87d0546d)

## 作成するリソース
| 必要なAWSリソース |  名称 |
| ---------------- | ----- |
| ECS Cluster | my-app-cluster |
| ECS Service | my-app-api-service |
| Task定義 | my-app-api |
| ECR Repository | my-app-api |
| VPC  | my-workspace-vpc |
|  Subnet  | my-workspace-subnet-app-public1-a |
|          |  my-workspace-subnet-app-public1-b |
| Security Group | my-app-api-sg |
| I AM Role | ecsTaskExecutionRole |

### WebAPIを実装
Java -> build -> 1つのJARファイルというパッケージにまとめられる -> アプリ実行

## Open ID Connect
ユーザー認証を簡略化するためのプロトコル。ユーザーがログインする際に、別の信頼できるサービスのアカウント(Google, FaceBookなど)を使用できるようにする。

**参考** [アクセス許可設定の追加](https://docs.github.com/ja/actions/security-for-github-actions/security-hardening-your-deployments/configuring-openid-connect-in-amazon-web-services)

![learn-ecs-OpenID Connect drawio](https://github.com/user-attachments/assets/4e4f4eaf-58d3-4705-976c-087b273a6f69)
![learn-ecs-IDトークンのデータ構造 drawio](https://github.com/user-attachments/assets/9b6c2bfc-d117-4437-93fc-ebb8a235f5dd)


## ECSへのデプロイ
ECSのアプリをアップデートする -> Task定義を更新し、デプロイすること
リビジョン番号1 -> 2

Task定義はJSONで記述してあるため、JSONファイルを更新することでリビジョン2としてサーバにアップロードする。
具体的にやることは、ほとんど変更なしのJSONテンプレートを用意し、imageフィールドだけを変更する記述をGitHubActionで設定。

