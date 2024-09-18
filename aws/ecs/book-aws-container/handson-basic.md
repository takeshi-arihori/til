
# ハンズオン
![aws-container-basic-sec03 sample-application-aws-diagramのコピー drawio](https://github.com/user-attachments/assets/9fc797c9-73fc-4a28-82bb-ef340e1c76ba)


## 流れ
**Step1: ネットワークの構築**  
- VPCとサブネットの作成
- インターネットゲートウェイの作成
- ルーティングテーブルの作成
- セキュリティグループの作成

**Step2: アプリケーションの構築**  
- サンプルアプリケーションの概要確認
- サンプルアプリケーションの取得

**Step3: コンテナレジストリの構築**  
- ECRの作成
- VPCエンドポイントの作成
- サンプルアプリケーションの登録

**Step4: オーケストレーションの構築**  
- ネットワーク関連設定
- タスク定義の作成
- ECSクラスター/ECSサービスの構築
- サンプルアプリケーションのデプロイ確認

**Step5: データベースの構築**  
- DBネットワークの作成
- DBインスタンスの作成
- テーブルの作成
- バックエンドアプリケーションとの接続

**Step6: 全体の動作確認**  
- フロントエンドアプリケーションの更新
- 疎通確認


## Step1: ネットワークの構築の概要
各サブネットへ割り当てるIPv4 CIDRブロックを検討しておく。  
ap-northeast-1c側の管理用サブネットを作成する理由: AZ障害時への備え(障害が起こった時にサブネット設計がなければサブネット割り当てから行う必要があるため)

