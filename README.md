# azure-3tier-portfolio
Secure 3-Tier Web Application Infrastructure on Azure (Hub-Spoke architecture with Azure Firewall and Private Endpoints) built using Terraform and GitHub Actions.

---

# Azure 3-Tier Architecture Project  
**Status: 企画・設計フェーズ（進行中）**

Azure 上に一般的な Web サービス構造（Web / API / DB）を構築し、ネットワーク・セキュリティ・IaC・自動化まで含めた実務レベルのクラウドアーキテクチャを作るプロジェクトです。  
まだ構築途中ですが、進めている内容を公開しながら進行していきます。

---

## 1. プロジェクトの目的
Azure 上で Web / API / DB の三層構造を実際に構築しながら、  
クラウド設計・ネットワーク・セキュリティ・IaC・監視といった **実務で必要なスキルを自分の手で身につけること** を目的としています。

特に意識しているポイントは以下です。

- セキュアなネットワーク構成（Hub-Spoke）  
- インターネット公開時の安全性（Azure Firewall）  
- IaC による再現性（Terraform）  
- 運用監視の基礎（Log Analytics）  
- 自動デプロイ（GitHub Actions）

---

## 2. システムの全体像（構築予定）
一般的な EC サイトを想定し、インターネットから安全にアクセスできる Azure 環境を構築します。

- Web / API / DB の三層アプリケーション  
- Hub-Spoke ネットワーク  
- Azure Firewall による通信制御  
  ※個人検証環境のため、検証後はコスト最適化の観点から速やかに destroy（削除）します 
- Private Endpoint による DB の非公開化  
- Log Analytics によるログ監視  

---

## 3. 2週間（14日間）のスプリント計画
Jira を使ってタスクを整理しながら、アジャイル形式で進めます。

**Day 0（現在）**  
企画・設計フェーズ

**Day 1〜3**  
アプリ構築（Web / API / DB の展開と接続確認）

**Day 4〜6**  
ネットワーク構築（Hub-Spoke / Firewall / UDR の設定と疎通テスト）

**Day 7〜8**  
DNS・SSL（カスタムドメイン設定、パブリックアクセス検証）

**Day 9〜10**  
監視設定（Log Analytics、アラート設定）

**Day 11〜12**  
IaC 化（Terraform によるコード化・テンプレート化）

**Day 13〜14**  
最終まとめ（ドキュメント整理、構成図作成、スプリントレビュー）

---

## ■ Jira（プロジェクト管理詳細）

### Epics（全体構成）
- Module 1 – 3-Tier Application（Web / API / DB）  
- Module 2 – Network Foundation（Hub-Spoke / Firewall / UDR）  
- Module 3 – Operations Automation（Monitor / Logic Apps / Alerts）  
- Module 4 – DNS & Custom Domain Configuration  
- Module 5 – Infrastructure as Code（Terraform）

```text
┌──────────────────────────────────────────────────────────────┐
│ Project A: 3-Tier E-Commerce Prototype on Azure              │
└──────────────────────────────────────────────────────────────┘

【Module 1: Application (Web/App/DB)】
  ├─ Web App (App Service)
  ├─ API App (App Service)
  └─ Database (MySQL Flexible Server or Azure SQL)

【Module 2: Network (Hub-Spoke + Firewall + UDR)】
  ├─ Internet
  ├─ Azure Firewall
  ├─ Hub VNet
  │    ├─ Firewall Policy
  │    └─ Private DNS Zone
  └─ Spoke VNet
       ├─ Subnet-Web (NSG-Web)
       │    ├─ Web LB
       │    └─ Web App
       ├─ Subnet-App (NSG-App)
       │    ├─ App LB
       │    └─ API App
       └─ Subnet-DB (NSG-DB)
            ├─ Private Endpoint
            └─ Database

【Module 3: Monitoring Automation (Logic Apps + Teams)】
  ├─ Azure Monitor
  ├─ Log Analytics Workspace
  ├─ Logic Apps (Alert → Teams Notification)
  └─ Teams Channel

【Module 4: DNS + Custom Domain】
  ├─ Azure DNS Zone
  ├─ A / CNAME Records
  └─ App Service Custom Domain

【Module 5: Infrastructure as Code (Terraform)】
  ├─ Terraform Modules
  │    ├─ network
  │    ├─ compute/app
  │    ├─ database
  │    └─ monitoring
  ├─ GitHub Actions (CI/CD)
  └─ Remote State (Azure Storage + Key Vault)
```



### Sprint Overview（要約）

**Sprint 1：アプリ構築**  
Web / API / DB の作成、接続確認、App Service 設定、DB 初期化

**Sprint 2：ネットワーク基盤**  
Hub / Spoke、NSG、UDR、Peering、Firewall、疎通テスト

**Sprint 3：監視・運用**  
Log Analytics、Diagnostic Settings、アラート、Logic Apps、Dashboard

**Sprint 4：DNS / SSL**  
DNS Zone、A レコード、カスタムドメイン、SSL、HTTPS 検証

**Sprint 5：IaC / GitHub**  
Terraform /パラメータ化、IaC デプロイ、GitHub 反映、構成図追加

---

## 4. 事前準備（完了済み・一部未着手）
本プロジェクトを安全かつ再現性高く進めるため、以下の基盤準備を実施しています。
GUI で構築したリソースは、後続フェーズで Terraform に置き換える予定です。

### ■ Azure 基盤準備（完了）
- Resource Group（アプリ / ネットワーク / 監視 / DNS / tfstate 用）を作成  
- Terraform backend 用 Storage Account / Container を作成  
- Log Analytics Workspace を作成  
- Azure Budget によるコストアラート設定  
- GitHub Actions（OIDC）による Azure Login の動作確認  
- Terraform → GitHub Actions → Azure の plan-only 動作テスト  

### ■ アプリ・ネットワークの初期構築（GUI / 一部未着手）
- Web / API / DB の初期構築（GUI）  
- Hub VNet の作成（GUI）  
- Spoke / Private Endpoint / Firewall は後続フェーズで構築予定  
- GUI で構築した内容は後で Terraform に置き換える（IaC 化）  

### ■ プロジェクト進行方針（整理済み）
- 「手動構築 → 動作確認 → Terraform 化」の順で進める  
- コスト最適化のため、Firewall など高額リソースは検証後に destroy  
- スプリント計画に沿って Module ごとに段階的に構築  
---

## 5. 最終成果物（予定）
- 企画書（本ドキュメント）  
- 三層構造アプリケーション（Web / API）  
- Hub-Spoke ネットワーク構成  
- Firewall / Private Endpoint 設定  
- Terraform モジュール  
- GitHub Actions workflow  
- Draw.io による構成図  
