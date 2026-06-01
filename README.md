# azure-3tier-portfolio
Secure 3-Tier Web Application Infrastructure on Azure (Hub-Spoke architecture with Azure Firewall, Bastion, and Private Endpoints) built using Terraform and GitHub Actions.



---

## 📋 プロジェクト企画・作業計画書

### 1. プロジェクトの目的
Azure 上で一般的な Web サービス構造（Web / API / DB の三層構造）を実際に構築し、クラウド設計・ネットワーク・セキュリティ・自動化といった **実務で求められるスキルを証明すること** を目的とした。

特に、企業システムで必須となる  
- セキュアなネットワーク構成  
- インターネット公開の安全性  
- IaC による再現性  
- 運用監視の基礎  

これらを自分の手で構築し、**実行力のあるエンジニアであることを示す** ことを狙いとした。

### 2. やったことの全体像
一般的な EC サイトを想定し、インターネットから安全にアクセスできる Azure 環境を構築した。

実施内容は以下の通り：
- Web / API / DB の三層アプリケーション構築  
- Hub-Spoke ネットワーク設計  
- Azure Firewall による通信制御  
- Bastion による安全な管理アクセス  
- Private Endpoint による DB の非公開化  
- Log Analytics による監視  
- Terraform による IaC 化  
- GitHub Actions による自動デプロイ  

これらを通じて、**セキュアなクラウドアーキテクチャの一連の流れ** を実践した。

### 3. スプリント計画（スケジュール）
進行管理には Jira を使用し、アジャイルスプリント形式でタスクとサブタスクを整理しながら進めた。これにより、作業の抜け漏れ防止と進捗の可視化を実現した。

- **Day 1〜3**：アプリ（Web / API / DB）の構築・接続確認
- **Day 4〜6**：ネットワーク（Hub-Spoke / Firewall / UDR / 疎通テスト）
- **Day 7〜8**：DNS・SSL（カスタムドメイン / パブリックアクセス検証）
- **Day 9〜10**：監視・運用（Log Analytics / アラート設定）
- **Day 11〜12**：IaC化（Terraformによるコード化・テンプレート化）
- **Day 13〜14**：最終仕上げ（README・構成図の作成、スプリントレビュー）

### 4. 事前準備（実体験ベース）
本構築を進めるにあたり、以下の準備を行った：
- Azure 上にリソースを展開するための受け皿（Resource Group / Storage Account）を作成  
- Terraform を GitHub に push することで自動デプロイできる環境を構築  
- apply が暴走しないよう、plan-only の安全なテストを実施  
- Azure Budget を設定し、高額課金を防ぐため閾値ごとにメール通知を設定  
- 手動構築 → IaC 化の順番を決め、作業の再現性と安全性を確保  

これらの準備により、**安全に・確実に・再現性のある構築プロセス** を整えた。

### 5. 成果物
- 企画書（本文）  
- 三層アプリ  
- Hub-Spoke ネットワーク  
- Firewall / Bastion  
- Private Endpoint  
- Terraform モジュール  
- GitHub Actions workflow  
- README / 図解  

実際に動作する環境と、構成を説明できるドキュメントの両方を揃えた。

### 6. 学び・改善点
AI に相談しながら構築を進めたことで、Azure のネットワーク・セキュリティ・IaC の理解が大きく深まった。今後は今回の経験をベースに、より自走できるエンジニアとしてスキルを磨いていきたい。

特に、  
- Firewall ルールの調整  
- Private DNS Zone のリンク設定  
- Terraform の変数化  
- GitHub Actions の安全な運用  

これらは実際に手を動かすことで理解が進んだため、今後の案件でも活かせる学びとなった。
