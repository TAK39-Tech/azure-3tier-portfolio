# 🌐 【実務想定】Azure 3層Webアプリケーション インフラ構築実績

## 1. 概要（Overview）
Node.js（Web）× Node.js（API）× MySQL（DB）によるクラウドネイティブな 3 層アプリケーション基盤を Azure 上に構築。
Web 層は独自の Node.js サーバー（server.js）を App Service 上で稼働させ、API 層および MySQL Flexible Server と連携。Web → API → DB のセキュアな双方向疎通までを完了し、実務レベルの構築・検証プロセスを再現しました。

## 2. 技術スタック（Tech Stack）
- **Web層**：Azure App Service（Linux / Node.js） / 独自 Node.js サーバー（server.js）
- **API層**：Azure App Service（Linux / Node.js） / Expressベース API
- **DB層**：Azure Database for MySQL – Flexible Server（SSL/TLS 必須）

## 3. WBS に基づくタスク実行実績（Execution Based on WBS）
実務を想定し、要件からタスクを分解（WBS化）。スプリント計画に沿って基盤構築を完了。

- **✔ 2種類の通信の役割明確化**
  - **① Web層 ➔ API（サーバー間通信）**: CORS不要。Web層がAPIの稼働状況を確認し、結果を `index.html` に動的埋め込み。
  - **② ブラウザ ➔ API（本番通信）**: CORS必須。画面にDBデータを表示するための本番API呼び出し。
- **✔ Web基盤構築**
  - App Service（Web）作成および Japan West のクォータ制限回避
  - `server.js` / `index.html` のデプロイと pm2 による常駐起動
- **✔ API基盤構築**
  - App Service（API）作成（同一プラン内でのコスト最適化）
  - Node.js ランタイムの起動検証（pm2 / Oryx）
- **✔ DB基盤構築**
  - MySQL Flexible Server 作成（Japan East）、Firewall 接続制限、最小権限ユーザー作成
  - SSL/TLS 必須設定に対応（rejectUnauthorized: false）

## 4. 課題解決（Troubleshooting 実績）

### 事象①：HTTP 200 が返るのに画面が真っ白で動かない（App Serviceの罠）
- **原因**：App Service のフォールバック動作（`hostingstart.html`）が発動し、Node.js アプリのプロセスが起動していないにもかかわらず、表面上は 200 OK を返していた。
- **対策**：DevTools の Network タブでレスポンスの本体を確認。Kudu/SSH 経由で内部コンソールへ入り、`node_modules` の不足を特定。`npm install` および `pm2` でプロセスを明示的に立ち上げて復旧。
- **強み**：表面的なステータスコードに依存せず、インフラとアプリの境界を理解した上で論理的に切り分けできる。

### 事象②：MySQL が SSL 必須設定により接続を拒否（セキュリティ要件）
- **原因**：MySQL Flexible Server 側の初期セキュリティ要件（`require_secure_transport=ON`）により、非暗号化の接続試行がすべて遮断されていた。
- **対策**：Node.js（mysql2）の接続時オプションに `ssl: { rejectUnauthorized: true }` を追加し、クラウド特有のセキュリティ要件に準拠したセキュアなコードへと修正。

### 事象③：非同期処理の順序問題により画面へデータが反映されない
- **原因**：API（Node.js）内部で非同期処理（`fs.readFile`等）のハンドリングが不適切であり、DB からのデータ取得が完了する前に、空の状態で HTML やレスポンスを返却していた。
- **対策**：`fs.promises.readFile` ＋ `await` による非同期制御へのリファクタリングを実施し、プログラムの実行順序を厳密に制御。

### 事象④：Web ➔ API 連携時の CORS 接続エラー
- **原因**：Web と API がそれぞれ異なる App Service（異なるURL）で稼働しているため、ブラウザの同一オリジンポリシーによってクロスオリジン通信がブロックされた。
- **対策**：API 側の App Service の CORS 設定メニューより、Web 側のフロントエンドURLを明示的にホワイトリストに登録して解決。

### 事象⑤：API ➔ DB 結合時の接続タイムアウトエラー
- **原因**：MySQL 側の IP ファイアウォール制限により、App Service（API）の動的アウトバウンド IP からのアクセスが初期設定で弾かれていた。
- **対策**：MySQL のネットワーク構成にて「Azure 内の任意のサービスからのアクセスを許可する」のチェックを有効化し、クラウド内のセキュアな通信を許可。

## 5. エンドツーエンドの疎通検証ロジック
本アーキテクチャでは、裏側で「本当に3層が安全に結合しているか」をリアルタイムに検証するロジックを組み込んでいます。

1. **Web ➔ API の疎通確認**
   Web層のNode.jsサーバー（server.js）からAPIのエンドポイントへサーバー間通信を実行。APIから応答が返ってきた時点で、Web層がHTML内の「Web ➔ API」ステータスを🟢（有効）に加工します。
2. **API ➔ DB のSSL/TLS結合確認**
   API内部でAzureの環境変数を検証後、`ssl: { rejectUnauthorized: true }` を有効化した状態でMySQL Flexible Serverへセキュアに接続。
3. **データ取得による結合証明**
   APIがDBから `SELECT` クエリで商品データを動的に取得し、成功ステータス（`status: "API_SUCCESS_OK"`）を返却。Web層のサーバー（server.js）がこのレスポンス内容をフックし、フロントエンド（React）画面の「API ➔ DB」ランプを🟢（有効）に動的変換した上でブラウザへ画面を返却しています。

## 6. 構築・結合エビデンス（検証結果）

### ■ 3層結合・疎通テスト成功画面
<!-- 💡 ここに、アプリ画面とデベロッパーツールのNetworkタブを左右に並べたスクショ（1枚）を貼り付ける -->
