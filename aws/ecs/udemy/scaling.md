# オートスケーリング

システム利用者の増大に応じて、システム規模を拡大縮小すること。
- トラフィック多い -> コンテナ増
- トラフィック少ない -> コンテナ減

## スケーリングの種類
### 水平スケーリング (ECSサポート)
サーバの数を増減すること -> 処理をなるべく分散させるアプローチ
- スケールアウト: サーバの数を増やすこと
- スケールイン: サーバの数を減らすこと

#### 得意
一つ一つの処理は軽めだが大量の処理を行う必要がある。
例: ECでひとつひとつのユーザーからのリクエストなど。認知が上がり大量のアクセスが起こる場合など

#### 苦手
一つ一つの処理が高度な内容、メモリをたくさん消費する処理など。
状態を持つようなユースケースなど。ユーザー -> サーバへリクエスト -> 事前の状態を把握する必要がある場合など。(複雑性が増し設計が困難)

### 垂直スケーリング
サーバのCPUなどを変更すること。
- スケールアップ: 増やす
- スケールダウン: 減らす

#### 得意
大量のCPUやメモリをさばくこと。
動画、画像処理などの処理。
状態を持つようなユースケース。

#### 苦手
物理的にスケールアップの上限がある。
サーバの置き換えの際に落ちないか不安。

### ECSのサポートするスケーリング
#### ステップスケーリングポリシー(カスタマイズ性は高い)
CPUやメモリ使用率に応じて定義されたコンテナ数に合わせる。
例: CPU使用率 30 % -> 3個、60 % -> 5 個にする。
割合で設定も可能。(1.5倍など)

#### ターゲット追跡スケーリングポリシー (こちらを選択することが多い)
CPUやメモリのターゲット使用率になるようコンテナ数を増減させる。

ターゲット追跡アラームがユースケースで機能しない場合にステップスケーリングを使用する。


## システム要件と設計書
![learn-ecs-Auto Scaling drawio](https://github.com/user-attachments/assets/13554676-05b7-4773-8b52-69e5ebb461b0)

![learn-ecs-sec08-handson drawio](https://github.com/user-attachments/assets/cf772de0-b0b3-4010-a58c-d80674bedf3d)

**全体の流れ**  
- クライアントがHTTPリクエスト -> Load Balancer(Security Group 8080番のみ通す を設定)
- ロードバランサーがECSのタスクにリクエスト
- コンテナはオートスケールした場合、2,3 と増えることにになる。
- 暗号化を行うことで負荷を意図的に欠けるためのタスク
- Docker Image: ECR
- CloudWatch: タスクやCPU使用状況などを監視し、既定の値を超えた場合オートスケールするようにECSに命令を出す。

### 作成するリソース
| 必要なAWSリソース |  名称 |
| ---------------- | ----- |
| ECS Cluster | my-app-cluster |
| ECS Service | my-app-autoscaling-service |
| Task定義 | my-app-encryptor |
| ECR Repository | my-app-encryptor |
| CloudWatch Policy | my-app-autoscaling-policy |
| VPC  | my-workspace-vpc |
|  Subnet  | my-workspace-subnet-app-public1-a |
|          | my-workspace-subnet-app-public1-b |
| Security Group | my-app-autoscaling-lb-sg |
|                | my-app-autoscaling-service-sg |
| I AM Role | ecsTaskExecutionRole |
| Application Load Balancer | my-app-autoscaling-alb |
| Target Group | my-app-autoscaling-tg |  


![learn-ecs-sec08-handson2 drawio](https://github.com/user-attachments/assets/b2607836-c434-4ae8-a16e-73452cac328b)

## 作成手順

k6を使用: 負荷テスト用

ECRレジストリを作成後、localのimageをpushするため(Pushコマンド通りに手動でPush)  
ECS -> タスク定義を作成  
ECS -> Serviceの作成(LB, TGも一緒に作成)  
Serviceのセキュリティ: ロードバランサのセキュリティグループからのみのアクセスにインバウンドルールを指定
