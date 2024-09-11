
## Blue/Greenデプロイ
### コンテナを利用したアプリケーション開発におけるデプロイ(アップデート時に起こりやすい問題)
App v1, App v1, App v1 -> App v2, App v2, App v2
古いコンテナをいったん閉じて新しいバージョンのコンテナを開く必要があるのだが、どういう手順でアップデートする??

#### 古いコンテナを一括で止める -> 新しいコンテナを立ち上げる
問題1: 古いコンテナを止めてから新しいコンテナを起動するまでアプリを止めてしまうことになる。
問題2: 深刻な bag が発生する可能性がある。

#### 上記の課題を解決するためのECSがサポートするデプロイ方式
- ローリングアップデート: コンテナひとつづつ新しいコンテナに置き換える方法(コンテナが0にならないのでユーザーはどこかのコンテナにアクセスすることができる)
- Blue/Greenデプロイ: 新旧のバージョンを並行稼働させることでダウンタイムを最小限に抑え、リスクを管理しやすくする手法。(**AWS CodeDeploy**を使って実装)

### Blue/Greenデプロイの流れ

![Greenデプロイ drawio](https://github.com/user-attachments/assets/5bbd85c0-5a23-465e-b18c-51af4a6aff69)
1. 古いほうのコンテナにルーティングしたまま、いきなりアプリを含んだコンテナを一気に立ち上げる。
2. Load Balancerを新しいほうのコンテナに向ける
3. 古いコンテナを削除(コスト節約のため、不要になった時点で削除)

### Load Balancerの中身
- Linsener: 特定のプロトコルを監視する(例: HTTP:80 -> Target Group Blue に流し込む設定)
- Target Group: Listenerのルールに基づいて通信を受け取る対象をグルーピング化したもの

#### Greenアプリが正常稼働しているか確認する方法
1. ターゲットグループのヘルスチェックを行う
2. 開発者が手動で事前にHTTP:9000を使った動作確認

![Greenデプロイ drawio](https://github.com/user-attachments/assets/4cd92776-20d9-426d-b585-ad4bde75fcc2)

### ハンズオン流れ

#### リージョン:
オレゴン

#### セキュリティグループの作成
- EC2 -> セキュリティグループ
- インバウンドルール: HTTP -> anywhere, カスタムTCP -> 自分のIP/32

#### ロードバランサの設定
- Type: Application Load Balancerを選択(HTTP, HTTPS のレイヤーでのルーティングのためにこれを選択)
- 名前: my-app-alb
- スキーム: インターネット向け
- ロードバランサーの IP アドレスタイプ: IPv4
- ネットワークマッピング: 作成したVPC、アベイラビリティゾーン(2つ以上必要なので足りない場合はこの時に別アベイラビリティゾーンを設定したサブネットを追加)(ルートテーブルは後で作成)
- セキュリティグループの設定: my-app-lb-sg
- リスナーとルーティング -> ターゲットグループの作成: IPアドレス, グループ名`my-app-frontend-tg-1`, ターゲットグループはそのまま(defaultのIPアドレスは削除)

一般ユーザーからHTTP:80 -> Target Groupに先導まで作成済み。

#### ECS Service作成
ECS -> クラスターを選択 -> Serviceを作成
- コンピューティングオプション: 起動タイプ, FARGATE, LATEST
- デプロイ設定: アプリケーションタイプ -> サービス, ファミリー -> my-app-frontend, リビジョン -> 最新, サービスタイプ -> レプリカ, 必要なタスク -> 1, デプロイオプション -> Blue/Greenデプロイ, デプロイ設定 -> CodeDeployDefault.ECSAllAtOnce,
CodeDeploy のサービスロール -> IAMの作成(AWSCodeDeployRoleForECS -> 名前: AWSCodeDeployRoleForECS), 

![スクリーンショット 2024-09-11 112656](https://github.com/user-attachments/assets/4bf97f2d-ac51-4da2-93cd-568e2b7c091b)

![スクリーンショット 2024-09-11 123334](https://github.com/user-attachments/assets/6505a6ad-8e31-43bc-82f4-cecf42c05646)


![スクリーンショット 2024-09-11 112656](https://github.com/user-attachments/assets/b4f0fc42-2101-447e-95bb-c68037afb470)

作成後、ロードバランサーのリスナーとルールに9000番で設定したルールが追加されている

![スクリーンショット 2024-09-11 124253](https://github.com/user-attachments/assets/2b10f031-e20e-4fc5-aa34-12ccf5f1183b)


### Code Deployの設定
Blue/Greenデプロイがまだ未完成なので、デプロイメントは作成されていない。
CodeDeploy -> アプリケーション -> 作成されたDeployを選択

![スクリーンショット 2024-09-11 135032](https://github.com/user-attachments/assets/0061800f-03cc-418d-840a-a85f94b5b8f0)

#### デプロイ設定
- リスナーが新しいターゲットグループに変更するのを手動か自動化設定
トラフィックの再ルーティング: トラフィックを再ルーティングするタイミングを指定します -> 2時間

- Blueコンテナを削除するまでの猶予時間
元のリビジョンの終了: 1時間

### ECR設定
1. 自分たちのECRリポジトリを作成
   ECR -> リポジトリ -> プライベートリポジトリ作成
2. "Hello World"と表示する簡単なアプリを作成する
**開発の流れ**
![learn-ecs-original-application drawio](https://github.com/user-attachments/assets/b68e0e4d-d6df-40b0-9add-c773e1700ea6)
- LocalでDockerfileを作成し、テスト
- IAM でアクセス権限を付与
- AWS CLIをインストールし、アクセスキー等の設定を行う

4. 作成したアプリを含むコンテナイメージをECRリポジトリにアップロード(Push)する
- ECRのプッシュコマンドを表示通りにコマンドを入力しPush -> ECRにPushしたイメージが反映されているか確認
- Blue/GreenのGreenで使用するためのタスクを追加 (URIはPushしたイメージのURI)
- クラスター -> 作成したクラスター -> サービスの更新 -> 最新のリビジョンを選択 -> 

ECSの削除はCloudFormationから行うと楽。


