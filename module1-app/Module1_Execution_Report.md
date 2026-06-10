#  【最終提出用】Azure 3層Webアプリケーション インフラ構築実績（Module 1 完了版）

## 1. 概要（Overview）
Node.js（Web）× Node.js（API）× MySQL（DB）によるクラウドネイティブな 3 層アプリケーション基盤を Azure 上に構築。
Web 層は独自の Node.js サーバー（server.js）を App Service 上で稼働させ、API 層（Node.js）および MySQL Flexible Server と連携し、Web → API → DB の疎通までを完了。実務レベルの構築・検証プロセスを再現しました。

## 2. 技術スタック（Tech Stack）
- **Web層**：Azure App Service（Linux / Node.js）
  - 独自 Node.js サーバー（server.js）
  - index.html を動的に加工して返却
  - API 疎通テスト機能を内蔵
  - pm2 による常駐起動
- **API層**：Azure App Service（Linux / Node.js）
  - Express ベース of API
  - DB 接続処理を担当
  - CORS 設定でブラウザからのアクセスを許可
- **DB層**：Azure Database for MySQL – Flexible Server
  - SSL/TLS 必須
  - 最小権限ユーザーで運用
  - Firewall による接続元制限

###  Azure 3層Webアプリケーション構成図
![Azure3層構成図](../docs/images/architecture-diagram.png)

## 3. WBS に基づくタスク実行実績（Execution Based on WBS）
実務を想定し、要件からタスクを分解（WBS化）。スプリント計画に沿って Module 1 を完了。

### ■ Jiraスプリントタスク完了実績
![Jiraスプリント完了実績](../docs/images/JiraM1完了.png)

---

### ✔ Web → API 結合（2種類の通信の役割を明確化）
本構成では Web 層（Node.js）と API 層（Node.js）が別 App Service のため、以下の 2種類の通信 が発生する。

#### ① Web層 Node.js（server.js）→ API（サーバー間通信）
- **目的**：Web 層が API の稼働状況を確認する疎通テスト。API が返す特定文字（`API_SUCCESS_OK`）を検証し、結果を `index.html` に動的に埋め込み、通知バーとして表示。
- **特徴**：サーバー間通信のため CORS 不要。API や DB が落ちていても Web 画面は正常表示され、その代わり、通知バーに「API疎通失敗」「DB疎通なし」などのメッセージを返せる。

#### ② ブラウザ → API（本番通信）
- **目的**：画面に DB のデータを表示するための本番 API 呼び出し。
- **特徴**：Web と API が別ドメインのため CORS が必要。API → DB が失敗している場合、画面にデータは表示されない。

 **この 2 種類の通信により、“Web 画面は正常に表示されるが、DB の結果だけ表示されない” といった状態を正しく検知・表示できる構成となっている。**

---

- **✔ Web 基盤構築（Node.js Web サーバー）**
  - App Service（Web）作成、Japan West のクォータ制限を回避
  - `server.js` / `index.html` をデプロイ、pm2 による Node.js 常駐起動
  - ブラウザでの動作確認（HTTP 200 / 動的 HTML 生成）
- **✔ API 基盤構築**
  - App Service（API）作成（Web と同一プランでコスト最適化）
  - CORS 設定で Web → API の通信を許可、Node.js ランタイムの起動検証（pm2 / Oryx）
- **✔ DB 基盤構築**
  - MySQL Flexible Server 作成（Japan East）、Firewall による接続元制限、最小権限ユーザー作成
  - SSL/TLS 必須設定に対応（`rejectUnauthorized: false`）
- **✔ Web → API 結合**
  - 環境変数（`API_URL`）で API エンドポイントを安全に管理
  - Web 層の `server.js` から API へ疎通テスト（http/https リクエスト）を実行し、レスポンス内容を Web 画面に動的表示
- **✔ API → DB 結合**
  - Connection String を App Service の環境変数に安全に配置
  - SELECT クエリの実行成功、API → DB → API の往復通信を確認

## 4. 課題解決（Troubleshooting 実績）

### 事象①：HTTP 200/404 やアプリエラーが頻発し、画面が正常に反映されない（App Serviceのデプロイ罠）

- **原因**：
  初回デプロイ時に App Service のフォールバック動作（hostingstart.html）が残存。その後、試行錯誤の中でデプロイを繰り返すうちに、Kudu内部に古いビルドキャッシュやデプロイゴミが蓄積し、アプリの更新が正常に反映されず404エラーやアプリエラーが発生。また、Node.js（server.js）側での静的コンテンツ（index.html）の配信・ルーティング制御にも考慮漏れがあった。
- **対策**：
  DevTools の Network タブでレスポンスの本体とステータスを追跡。Kudu/SSH 経由で内部コンソールへ入り、`/home/site/wwwroot` 内の不要なキャッシュ・一時フォルダを一度完全にクレンジング（削除）して環境をリセット。合わせて `server.js` の静的ファイル配信ロジック（Expressの静的ルーティング等）を正しく修正・配置し、再デプロイを行うことで正常な画面表示へと復旧させた。
- **強み**：
  不具合直面時に単なる上書きデプロイを繰り返すだけでなく、PaaS（App Service）のファイル構成やビルドプロセスにまで踏み込んで原因を特定できる。インフラのキャッシュ起因による環境トラブルと、アプリケーションのコード（ルーティング）起因によるバグの双方に視野を広げ、泥臭くも最短でボトルネックを突き止めて解決する自走力がある。

### 事象②：Azure Database for MySQL の SSL 必須設定による接続エラー

- **原因**：
  Azure Database for MySQL（Flexible Server）側の初期セキュリティ設定（require_secure_transport=ON）により、非暗号化（SSLなし）の接続試行がすべて自動的に遮断されていたため。
- **対策**：
  安易にインフラ側のセキュリティ設定を緩めてエラーを回避するような「ワークアラウンド（応急処置）」ではなく、クラウドのセキュリティ要件に準拠する根本解決を選択。Node.js（mysql2）のデータベース接続設定において、通信時に強制的にSSLを認識・利用するオプションを明示的に追加し、セキュアな暗号化通信経路を確立して接続を成功させた。
- **強み**：
  インフラが求める厳格なセキュリティ要件を正しく理解し、アプリケーション側の接続実装に落とし込める。仕様（公式ドキュメント）に基づく根本原因の特定と、本番環境を見据えた堅牢なセキュアコーディングを実践できる。

### 事象③：非同期処理の制御不備による、画面へのデータ未反映バグ

- **原因**：
  Node.js（server.js）内部で、HTMLテンプレートファイルを読み込む処理（fs.readFile）の非同期制御が不適切だったため。ファイルの読み込み完了を待たずにレスポンス処理が先走って実行され、ブラウザ側にデータが空の状態で返却されていた。
- **対策**：
  `fs.promises.readFile` と `await` を組み合わせた記述にリファクタリング。ファイルの読み込みが完全に完了してからレスポンスを返却するよう、プログラムの実行順序を厳密に制御して修正した。
- **強み**：
  JavaScript/Node.js特有の非同期処理の挙動（イベントループやPromise）を正しく理解している。画面にデータが映らない原因が、ネットワークの疎通不良なのか、バックエンドコードの処理順序のバグなのかをロジカルに切り分けてデバッグできる。

### 事象④：Web ➔ API 連携時の CORS 接続エラー

- **原因**：
  Web層とAPI層がそれぞれ異なるドメイン（異なる App Service）で稼働しているため、ブラウザの同一オリジンポリシーによってクロスオリジン通信が標準でブロックされたため。
- **対策**：
  API 側の App Service にある「CORS設定メニュー」より、フロントエンド（Web側）のURLを明示的にホワイトリストに登録。クラウド（PaaS）の提供する標準機能を活用し、安全にクロスオリジン通信を許可した。
- **強み**：
  複数コンポーネントに分かれた分散システム（3層構成）において、ブラウザのセキュリティ仕様（CORS）に起因する通信遮断を正しく見抜き、PaaSの設定変更によって迅速に問題を解消できる。

### 事象⑤：API ➔ DB 結合時の接続タイムアウトエラー

- **原因**：
  MySQL（Flexible Server）側の初期のIPファイアウォール制限により、App Service（API側）からの動的アウトバウンドIPによるアクセスが遮断され、通信が確立できずタイムアウトが発生していた。
- **対策**：
  MySQL のネットワーク構成（ファイアウォール設定）にて「Azure 内の任意のサービスからのアクセスを許可する」のフラグを有効化。同一クラウド（Azure）のネットワークセグメント内において、安全にPaaS間で通信を行えるようネットワーク統合を構成して解決した。
- **強み**：
  アプリケーションコードの不備だけでなく、クラウドインフラ（PaaS）側のネットワークセキュリティ（ファイアウォール設定）の壁に素早く気づき、クラウドの作法に則った設定変更で迅速に通信経路を導通させられる。



## 5. Module 1 の成果（Result）
- Azure 上に 3 層構造（Web / API / DB）の基盤を構築
- Web（Node.js）→ API（Node.js）→ DB（MySQL Flexible Server）の疎通を SSL/TLS で実現
- Web 層で API のレスポンスを動的に表示する仕組みを構築
- デプロイ後の動作差異に対して、ログ解析・設定見直しを通じて問題を解決

## 6. Module 1 完了エビデンス

### ① 外部アクセスからWebブラウザ閲覧
![Webブラウザ疎通成功画面](../docs/images/Web-Api-DB.png)

- **エンドツーエンドの疎通検証ロジック**：
  1. **Web ➔ API の疎通確認**：Web層のNode.jsサーバー（server.js）からAPIのエンドポイントへサーバー間通信を実行。APIから応答が返ってきた時点で、Web層がHTML内の「Web ➔ API」ステータスを🟢（有効）に加工。
  2. **API ➔ DB のSSL/TLS結合確認**：API内部でAzureの環境変数を検証後、`ssl: { rejectUnauthorized: true }` を有効化した状態でMySQL Flexible Serverへセキュアに接続。
  3. **データ取得による結合証明**：APIがDBから `SELECT` クエリで商品データを動的に取得し、成功ステータス（`status: "API_SUCCESS_OK"`）を返却。Web層のサーバー（server.js）がこのレスポンス内容をフックし、フロントエンド（React）画面の「API ➔ DB」ランプを🟢（有効）に動的変換した上でブラウザへ画面を返却しています。

### ② APIのWebAppは外部からの接続を拒否設定
![APIアクセス拒否画面](../docs/images/Api拒否.png)
- **セキュリティ担保の証明**：API側のApp Serviceへの直接アクセスを遮断し、セキュリティ上フロントのWeb層だけアクセスが可能となっている状態を担保。

### ③ Web 画面に DB のデータの反映
![DBデータ動的反映](../docs/images/DBデータ反映前.png)
![DBデータ動的反映](../docs/images/DBデータ反映後.png)
- **データ連動の証明**：MySQL内のレコード値を変更し、それがAPIを経由してWeb画面にリアルタイムかつ動的に反映されることを確認済みです。
