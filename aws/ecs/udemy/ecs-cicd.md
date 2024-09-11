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
