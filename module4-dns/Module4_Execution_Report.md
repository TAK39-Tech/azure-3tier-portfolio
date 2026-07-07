# Module 4 – DNS & Custom Domain Configuration 成果報告書

## 1. 概要（Overview）
お名前.comで取得したカスタムドメイン（`tiktak-dev.com`）のネームサーバーにAzure DNSを登録し、パブリックネットワークにおける名前解決とルーティング基盤を確立。Webフロントエンド（`www`）およびバックエンドAPI（`api`）の各App Serviceに対して独自のドメイン名を紐付け、Azureのマネージド証明書機能を用いた常時HTTPS化（SSL/TLS暗号化）による3層構造アプリケーションの外部公開基盤を構築しました。

## 2. 技術スタック（Tech Stack）

### 【構成図】Module 4: Public Routing & Domain Architecture

#### ■ パブリックルーティングイメージ（視覚的フロー）

> **[ここに構成図の画像を挿入]**
> - **対象画像**：MiroやDraw.io等で作成した、Module 4のネットワーク公開構成図

#### ■ パブリックデータフロー詳細（構造化テキスト）

```text
[ 外部インターネット ]
   │
   ▼ (ブラウザによるURLリクエスト)
[ お名前.com ] ── (ネームサーバー参照：://azure-dns.com 等)
   │
   ▼ 
[ Azure DNS Zone ] (tiktak-dev.com)
   │
   ├── ① ://tiktak-dev.com (CNAME転送) ──> [ Web App ] (web-3tier-demo-tak2026)
   │                                            │ (App Serviceマッピング)
   │                                            └── ssl_state: "SniEnabled" (常時HTTPS)
   │
   └── ② ://tiktak-dev.com (CNAME転送) ──> [ API App ] (api-3tier-demo-tak2026)
                                                │ (App Serviceマッピング)
                                                └── ssl_state: "SniEnabled" (常時HTTPS)
```

### 技術スタック詳細

- **DNSルーティング基盤**：Azure DNS Zone
    - ネームサーバー登録によるドメイン管理（`tiktak-dev.com`）
    - Webフロントエンド用CNAMEレコード（`www`）
    - バックエンドAPI用CNAMEレコード（`api`）
- **ドメイン所有権検証**：TXTレコード（`asuid` 認証方式）
- **暗号化通信（SSL/TLS）**：App Service マネージド証明書（無料） / SNI SSL設定

## 3. Jira に基づくタスク実行実績（Execution Based on Tasks）

本構成では、1つの独自ドメイン（`tiktak-dev.com`）を切り分け、Web層とAPI層のそれぞれに以下の2種類のサブドメインをマッピングしています。

*   **://tiktak-dev.com（Webフロントエンド用ドメイン）**
    一般ユーザーが外部のインターネット環境からブラウザ経由でポートフォリオサイトにアクセスするための公開窓口。Azure DNS Zone側で `www` のCNAMEレコードを作成し、Web用のApp Serviceへルーティング。無料のマネージド証明書（SNI SSL）をバインドすることで、常時HTTPS化による通信の安全性を確保。
*   **://tiktak-dev.com（バックエンドAPI用ドメイン）**
    フロントエンド画面からバックエンドのシステムへデータを通信するためのシステム専用窓口。Azure DNS Zone側で `api` のCNAMEレコードを作成し、API用のApp Serviceへルーティング。同様に無料のマネージド証明書による常時HTTPS化を実施。

### ■ Create DNS Zone
*   **Provision DNS Zone**：Azure上に専用のリソースグループ `rg-dns` をプロビジョニングし、グローバルパブリックDNSの受け皿となる `azurerm_dns_zone` リソース（ゾーン名: `tiktak-dev.com`）を作成。
*   **Verify NS records**：作成したDNSゾーンの管理画面から、Azure側で動的に自動生成された4本のネームサーバー（NSレコードのホスト名）の存在と、その正常なステータスを確認。
*   **Prepare domain registrar settings**：外部ドメインレジストラ（お名前.com）の管理コンソールにログインし、先ほど確認したAzure DNSの4本のネームサーバー情報を正確に登録し、ドメインの権限委譲に向けた事前設定を完了。

### ■ Add A Record
*   **Create A Record**：本構成においては、静的IPアドレスへの転送ではなくPaaS標準の初期ドメイン（`azurewebsites.net`）を活用する設計としたため、後述のCNAMEレコード作成にタスクを統合・最適化。
*   **Create CNAME (if needed)**：一般ユーザーおよびフロントエンドシステムからのアクセスを適切に振り分けるため、`www`（Webフロントエンド用）および `api`（バックエンドAPI用）の2本のCNAMEレコードをAzure DNSゾーン内に登録。
*   **Verify DNS propagation**：外部ターミナルから `nslookup` および `dig` コマンドを使用し、登録した `://tiktak-dev.com` および `://tiktak-dev.com` が、それぞれのApp Serviceの初期ホスト名へと正常に名前解決される（DNSが世界中に伝播している）事実を確認。

### ■ Configure Custom Domain
*   **Add custom domain to Web App**：Webフロントエンド用のApp Service（`web-3tier-demo-tak2026`）の管理コンソールにアクセスし、ホスト名バインディング設定に独自ドメイン `://tiktak-dev.com` をマッピング。
*   **Add custom domain to API App**：バックエンドAPI用のApp Service（`api-3tier-demo-tak2026`）の管理コンソールにアクセスし、システム専用の独立したホスト名 `://tiktak-dev.com` をマッピング。
*   **Validate domain ownership**：App Service側でのカスタムドメイン有効化の条件となる「所有権の証明」をクリアするため、Azure側から提示されたドメイン検証ID（ASUID）の値を抽出し、Azure DNSゾーン側へ `asuid.www` および `asuid.api` のTXTレコードをマッピング。

### ■ Bind SSL
*   **Create/Import certificate**：App Serviceへのドメイン登録完了をトリガーとして、各カスタムドメインに対応する「App Service マネージド証明書（無料のパブリックSSL/TLS証明書）」の自動発行プロセスを実行。
*   **Bind certificate to Web App**：発行された無料SSL証明書を `://tiktak-dev.com` に紐付け、暗号化のステータスとして `SniEnabled`（SNI SSL）を設定。
*   **Bind certificate to API App**：発行された無料SSL証明書を `://tiktak-dev.com` にも同様に紐付け、API通信における `SniEnabled`（SNI SSL）を設定。
*   **Verify HTTPS**：App Serviceの構成設定にて「HTTPS Only（HTTPSのみ強制転送）」のトグルを有効化し、暗号化されていないHTTP（ポート80）での不審なアクセスを遮断するプロトコルセキュリティを確認。

### ■ Verify Public Access
*   **Access via custom domain**：外部インターネットのクライアント環境から、ブラウザで直接 `://tiktak-dev.com` を入力してアクセスし、名前解決からWebアプリケーションの初期表示までが正常に行われることを検証。
*   **Test HTTPS**：アクセスしたブラウザのURL欄left側に「鍵マーク（🔒）」が正常に表示されていることを確認し、SSL/TLSハンドシェイクが成立して通信内容が完全に暗号化されていることを実証。
*   **End-to-end test**：独自ドメイン（`www`）で開いた画面からバックエンドへの各種リクエスト動作を実行。ブラウザ ➔ Webドメイン（`www`） ➔ APIドメイン（`api`） ➔ 仮想ネットワーク（VNet）内のMySQLデータベースという、異なるネットワーク境界を跨ぐ一連の通信がエラーなく安全に同期連動することを確認。

## 4. 課題解決（Troubleshooting 実績）

### 事象①：インポートパスの階層不足によるパースエラー
- **原因**：Azure App Serviceのカスタムドメイン（`hostNameBindings`）をインポートする際、プロバイダーが規定する「スラッシュで区切られた10個の階層（10 segments）」というルールに対し、末尾の指定が抜け落ちた「8個の階層」しか指定していなかったため。
- **対策**：エラーログの指示を確認し、不足していた末尾の階層（例：`/CNAME/www`）をそのまま付け足し、仕様に合致する「10個の階層」に直して実行することで解決しました。

### 事象②：インポートパスへの不要な記号の混入による解析エラー
- **原因**：パスを修正する過程で、ドメイン名の直前に不要な記号（`://`）が混入してしまい、文字列の解析が失敗したため。
- **対策**：エラーの文字列を確認し、不要な記号（`://`）だけを省いて実行することで解決しました。

### 事象③：Actionsの強制停止による管理ファイル（State）のロック残存
- **原因**：前のGitHub Actionsの処理が手入力待ちでフリーズし、途中で手動キャンセル（Cancel）したため。正常な終了処理が行われなかったことで、データの上書きを防ぐための安全鍵（ステートロック）が外されず、AzureのBlobコンテナ内に残ったまま放置され、次回のアクセスをブロックしていたため。
- **対策**：GitHub Actionsの画面を開き、実行中のジョブがすべて「停止している」になっていることを確認。その後、ローカルターミナルから `terraform force-unlock 【ロックID】` コマンドを実行し、残存していた古い鍵データを解除してパイプラインを復旧させました。

## 5. Module 4 の成果（Result）
- **独自ドメインによるパブリック名前解決ルートの確立**：お名前.comとAzure DNSゾーンの連携、およびCNAMEレコードの登録により、外部インターネットから独自ドメイン（`tiktak-dev.com`）を用いて各App Serviceへ正常にルーティングされるパブリックな名前解決環境を実現。
- **Web層とAPI層のネットワーク分離公開**：`www` と `api` のサブドメインを独立して各アプリへマッピングしたことで、一般ユーザー用のWebアクセス窓口と、システム間のAPI通信窓口をURLレベルで明確に分離した3層構造アプリケーションの公開状態を形成。
- **全エンドポイントの通信暗号化（常時HTTPS化）**：カスタムドメインへの無料マネージド証明書（SNI SSL）のバインドにより、フロントエンド画面のブラウザ接続時、およびフロント-API間のデータ通信時のすべてのレイヤーにおいて、SSL/TLSによる安全な暗号化通信を確立。

## 6. Module 4 完了エビデンス（Evidence）

### ① 独自ドメイン経由 of Webブラウザ閲覧（常時HTTPS化の検証）

> **[ここに写真を挿入]**
> - **対象画面**：ブラウザで開いた本番公開済みのポートフォリオ画面
> - **必要な要素**：URL欄の `https://://tiktak-dev.com` と、その左側に表示されている「鍵マーク（🔒）」
> - **役割**：お名前.comとAzure DNS間で登録したレコードが正常に名前解決され、無料マネージド証明書（SSL/TLS暗号化）によるHTTPS通信が機能していることの証明

- **検証の事実**：外部インターネット環境のブラウザから、登録した独自ドメイン（`https://://tiktak-dev.com`）へアクセスし、Webフロントエンドの画面が正常に表示されることを確認。また、URL欄の鍵マーク（🔒）の表示により、通信がHTTPSで保護されていることを実証。

### ② Azure DNS ゾーンのレコード登録状態

> **[ここに写真を挿入]**
> - **対象画面**：Azureポータルの「DNS ゾーン（`tiktak-dev.com`）」の管理画面
> - **必要な要素**：レコードセット一覧に並んでいる、名前が `www` と `api` のCNAMEレコード、および `asuid.www` と `asuid.api` のTXTレコードの一覧
> - **役割**：お名前.comから移譲されたDNSの箱の中に、必要なルーティングと検証用のレコードが正常に登録されていることの証明

- **検証の事実**：Azure DNS Zone内において、フロントエンドおよびバックエンドへのCNAME転送レコード、ならびにApp Serviceへの所有権証明用TXTレコードが、設計仕様通りに正常に登録されていることを確認。

### ③ App Service 側のカスタムドメインおよび SSL バインド状態

> **[ここに写真を挿入]**
> - **対象画面**：Azureポータルの Web App（`web-3tier-demo-tak2026`）の管理メニュー内にある「カスタムドメイン」の画面
> - **必要な要素**：割り当てられたカスタムドメイン一覧に `://tiktak-dev.com` が登録されており、かつ「SSL の状態」の項目に「SNI SSL」または「保護されている」と明記されている状態
> - **役割**：App Service側でドメインの紐付けとSSL証明書のバインドが正常に完了していることの証明

- **検証の事実**：対象のApp Service（Web AppおよびAPI App）の環境設定において、追加したカスタムドメインがバインディングされていることを確認。同時に、マネージド証明書の自動発行により、対象ホスト名に対して「SNI SSL」によるTLS/SSL暗号化が正常にバインドされていることを確認。

