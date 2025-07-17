# セキュアエージェントクラウドアーキテクチャ

このドキュメントでは、セキュアエージェントのAWSおよびAzureアーキテクチャについて説明します。

## AWS システム構成図

```mermaid
graph LR
    User["User"]
    
    %% AWS Cloud
    subgraph AWS["AWS Cloud"]
        %% Client
        subgraph Client["クライアント"]
            Route53["Route53"]
            CloudFront["CloudFront"]
            S3["Amazon S3<br/>(Angular)"]
            Route53 --> CloudFront
            CloudFront <--> S3
        end

        %% API Server
        subgraph APIServer["APIサーバー"]
            APIGW["Amazon API<br/>Gateway"]
            ECS["ECS<br/>(SpringBoot)"]
            RDS["PostgreSQL"]
            APIGW <--> ECS
            ECS --> RDS
        end

        %% AI
        subgraph AI["生成AI"]
            Bedrock["Bedrock<br/>(Anthropic)"]
            ECS --> Bedrock
        end

        %% Monitoring
        CloudWatch["Amazon<br/>CloudWatch"]
        CloudWatchLogs["Amazon<br/>CloudWatch Logs"]
        ECS --> CloudWatch
        ECS --> CloudWatchLogs
    end

    %% External Services
    subgraph External["外部サービス"]
        GitHub["GitHub"]
        ECR["ECR"]
        GitHub --> ECR
        GitHub --> S3
        ECR --> ECS
    end

    %% Azure Services
    subgraph Azure["Azure AI Services"]
        AzureOpenAI["AzureOpenAI<br/>(OpenAI)"]
        AzureAISearch["AzureAISearch"]
        Gemini["Gemini<br/>(Gemini)"]
    end

    %% Main Flow
    User --> Route53
    User -- "APIリクエスト" --> APIGW
```

## Azure システム構成図

```mermaid
graph LR
    User["User"]
    
    %% Azure Cloud
    subgraph Azure["Azure Cloud"]
        %% Client
        subgraph Client["クライアント"]
            DNS["Azure DNS"]
            CDN["Azure CDN"]
            Storage["Azure Storage<br/>(Angular)"]
            DNS --> CDN
            CDN <--> Storage
        end

        %% API Server
        subgraph APIServer["APIサーバー"]
            APIM["API Management"]
            AKS["AKS<br/>(SpringBoot)"]
            AzureSQL["Azure SQL"]
            APIM <--> AKS
            AKS --> AzureSQL
        end

        %% AI
        subgraph AI["生成AI"]
            OpenAI["Azure OpenAI"]
            AISearch["Azure AI Search"]
            AKS --> OpenAI
            AKS --> AISearch
        end

        %% Monitoring
        Monitor["Azure Monitor"]
        AppInsights["Application<br/>Insights"]
        LogAnalytics["Log Analytics"]
        AKS --> Monitor
        AKS --> AppInsights
        AppInsights --> LogAnalytics
    end

    %% External Services
    subgraph External["外部サービス"]
        GitHub["GitHub"]
        ACR["Azure Container<br/>Registry"]
        GitHub --> ACR
        GitHub --> Storage
        ACR --> AKS
    end

    %% Main Flow
    User --> DNS
    User -- "APIリクエスト" --> APIM
```

## コンポーネントの説明

### AWS クライアントサイド
- **Route 53**: DNSサービス
- **CloudFront**: CDNによるコンテンツ配信
- **S3**: 静的ウェブサイトホスティング（Angularアプリケーション）

### Azure クライアントサイド
- **Azure DNS**: DNSサービス
- **Azure CDN**: CDNによるコンテンツ配信
- **Azure Storage**: 静的ウェブサイトホスティング（Angularアプリケーション）

### AWS APIサーバー
- **API Gateway**: RESTful APIのエンドポイント管理
- **ECS (SpringBoot)**: コンテナ化されたバックエンドアプリケーション
- **RDS (PostgreSQL)**: リレーショナルデータベース

### Azure APIサーバー
- **API Management**: RESTful APIのエンドポイント管理
- **AKS (SpringBoot)**: Kubernetesによるコンテナオーケストレーション
- **Azure SQL**: マネージドSQLデータベース

### AWS AI/ML サービス
- **AWS Bedrock (Anthropic)**: AWS上での生成AI機能

### Azure AI/ML サービス
- **Azure OpenAI**: Azureで提供されるOpenAI機能
- **Azure AI Search**: 高度な検索機能
- **Gemini**: Google提供の生成AIモデル（外部連携）

### AWS モニタリング
- **CloudWatch**: メトリクス監視とアラート
- **CloudWatch Logs**: ログ管理

### Azure モニタリング
- **Azure Monitor**: メトリクス監視とアラート
- **Application Insights**: アプリケーション性能管理
- **Log Analytics**: ログ管理と分析

### AWS CI/CD
- **GitHub**: ソースコード管理
- **ECR**: コンテナイメージレジストリ

### Azure CI/CD
- **GitHub**: ソースコード管理
- **Azure Container Registry**: コンテナイメージレジストリ

## デプロイフロー

### AWS
1. フロントエンド（S3）
   - GitHubからS3へ直接デプロイ
   - CloudFrontによるキャッシュと配信

2. バックエンド（ECS）
   - GitHubからECRへのイメージプッシュ
   - ECRからECSへのデプロイ

### Azure
1. フロントエンド（Azure Storage）
   - GitHubからAzure Storageへ直接デプロイ
   - Azure CDNによるキャッシュと配信

2. バックエンド（AKS）
   - GitHubからACRへのイメージプッシュ
   - ACRからAKSへのデプロイ

## セキュリティ考慮事項

### AWS
- CloudFrontでのS3アクセス制限
- API Gatewayでの認証・認可
- RDSのセキュリティグループ設定
- AIサービスへのアクセス制御

### Azure
- Azure CDNでのStorageアクセス制限
- API Managementでの認証・認可
- Azure SQLのファイアウォール設定
- マネージドIDによるサービス間認証

## スケーリング戦略

### AWS
- CloudFrontによる静的コンテンツの配信最適化
- ECSタスクの自動スケーリング
- RDSのリードレプリカ（必要に応じて）

### Azure
- Azure CDNによる静的コンテンツの配信最適化
- AKSの水平ポッドオートスケーリング（HPA）
- Azure SQLのエラスティックプール活用 