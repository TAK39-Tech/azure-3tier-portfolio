azure-3tier-portfolio
Secure 3-Tier Web Application Infrastructure on Azure (Hub-Spoke architecture with Azure Firewall and Private Endpoints) built using Terraform and GitHub Actions.

Azure 3-Tier Architecture Project
Status: 企画・設計フェーズ（進行中）

Azure 上に一般的な Web サービス構造（Web / API / DB）を構築し、ネットワーク・セキュリティ・IaC・自動化まで含めた実務レベルのクラウドアーキテクチャを作るプロジェクトです。
まだ構築途中ですが、進めている内容を公開しながら進行していきます。

1. プロジェクトの目的
Azure 上で Web / API / DB の三層構造を実際に構築しながら、
クラウド設計・ネットワーク・セキュリティ・IaC・監視といった 実務で必要なスキルを自分の手で身につけること を目的としています。

特に意識しているポイントは以下です。

セキュアなネットワーク構成（Hub-Spoke）

インターネット公開時の安全性（Azure Firewall）

IaC による再現性（Terraform）

運用監視の基礎（Log Analytics）

自動デプロイ（GitHub Actions）

2. システムの全体像（構築予定）
一般的な EC サイトを想定し、インターネットから安全にアクセスできる Azure 環境を構築します。

Web / API / DB の三層アプリケーション

Hub-Spoke ネットワーク

Azure Firewall による通信制御
※個人検証環境のため Bastion はコスト面から除外

Private Endpoint による DB の非公開化

Log Analytics によるログ監視

3. 2週間（14日間）のスプリント計画
Jira を使ってタスクを整理しながら、アジャイル形式で進めます。

Day 0（現在）  
企画・設計フェーズ

Day 1〜3  
アプリ構築（Web / API / DB の展開と接続確認）

Day 4〜6  
ネットワーク構築（Hub-Spoke / Firewall / UDR の設定と疎通テスト）

Day 7〜8  
DNS・SSL（カスタムドメイン設定、パブリックアクセス検証）

Day 9〜10  
監視設定（Log Analytics、アラート設定）

Day 11〜12  
IaC 化（Terraform によるコード化・テンプレート化）

Day 13〜14  
最終まとめ（ドキュメント整理、構成図作成、スプリントレビュー）

■ Jira（プロジェクト管理詳細）
Epics（全体構成）
Module 1 – 3-Tier Application（Web / API / DB）

Module 2 – Network Foundation（Hub-Spoke / Firewall / UDR）

Module 3 – Operations Automation（Monitor / Logic Apps / Alerts）

Module 4 – DNS & Custom Domain Configuration

Module 5 – Infrastructure as Code（Terraform / Bicep）

Sprint Overview（要約）
Sprint 1：アプリ構築  
Web / API / DB の作成、接続確認、App Service 設定、DB 初期化

Sprint 2：ネットワーク基盤  
Hub / Spoke、NSG、UDR、Peering、Firewall、疎通テスト

Sprint 3：監視・運用  
Log Analytics、Diagnostic Settings、アラート、Logic Apps、Dashboard

Sprint 4：DNS / SSL  
DNS Zone、A レコード、カスタムドメイン、SSL、HTTPS 検証

Sprint 5：IaC / GitHub  
Terraform / Bicep、パラメータ化、IaC デプロイ、GitHub 反映、構成図追加

4. 事前準備（完了済み）
プロジェクトを安全に進めるため、以下の準備を完了しています。

Azure 上にリソース展開用の Resource Group / Storage Account を作成

Terraform → GitHub → 自動デプロイの動作テスト

apply 暴走防止のため plan-only テストを実施

Azure Budget によるコストアラート設定

手動構築 → IaC 化の順番を整理

5. 最終成果物（予定）
企画書（本ドキュメント）

三層構造アプリケーション（Web / API）

Hub-Spoke ネットワーク構成

Firewall / Private Endpoint 設定

Terraform モジュール

GitHub Actions workflow

Draw.io による構成図
