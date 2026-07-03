# 見積AI帳場 テンプレート版

「見積AI帳場」を別の会社・別のFirebaseプロジェクトで動かすための、設定値を空にしたテンプレートです。
このフォルダの中身一式（`index.html` / `manifest.json` / `sw.js` / `icons/`）をそのまま自分のホスティング先にアップロードして使ってください。`firestore.rules` は配布用ファイルではなく、Firebase Consoleに貼り付けるための参照ファイルです。

## セットアップ手順

### 1. Firebaseプロジェクトを作成
1. https://console.firebase.google.com で新規プロジェクトを作成
2. 「Authentication」→「Sign-in method」で **メール/パスワード** を有効化
3. 「Firestore Database」を作成（本番モードで可）

### 2. Firestoreのセキュリティルールを設定
1. Firestore Database →「ルール」タブを開く
2. このフォルダの `firestore.rules` の中身を貼り付けて「公開」

### 3. Firebaseの接続情報を入力
1. プロジェクト設定 →「マイアプリ」→ ウェブアプリを追加し、表示された設定値をコピー
2. `index.html` 内の `firebaseConfig` に値を貼り付け

```js
const firebaseConfig = {
  apiKey: "...",
  authDomain: "...",
  projectId: "...",
  storageBucket: "...",
  messagingSenderId: "...",
  appId: "..."
};
```

未入力のままでも起動はしますが、ログイン・単価表・見積履歴の保存ができません。

### 4. AI呼び出しエンドポイントを用意
見積項目の抽出・単価概算・メーカー単価表のジャンル分類にAIを使います。任意のAIモデル（Gemini、GPT等）を呼び出すサーバーレス関数（Cloud Functions等）を用意し、以下の契約に合わせてください。

**リクエスト**（アプリ→あなたのエンドポイント、POST・JSON）
```json
{
  "prompt": "...",
  "mode": "normal",
  "mediaContents": [{ "type": "image/png", "base64": "..." }],
  "responseSchema": { "...": "..." },
  "useSearch": false
}
```
ヘッダー: `Content-Type: application/json`, `x-access-key: <あなたが決めたキー>`
（`mediaContents` / `responseSchema` / `useSearch` は省略される場合があります）

**レスポンス**（あなたのエンドポイント→アプリ、JSON）
以下のいずれかの形になっていればOKです：
```json
{ "data": { "candidates": [ { "content": { "parts": [ { "text": "..." } ] } } ] } }
```
```json
{ "text": "..." }
```
```json
{ "data": { "text": "..." } }
```
`responseSchema` を指定したリクエストの場合、`text` の中身はJSON文字列にしてください（前後に \`\`\`json のようなMarkdown記法が付いていても自動で除去されます）。

用意ができたら `index.html` 内の以下を書き換えてください。

```js
const ASK_AI_ENDPOINT = "https://your-endpoint-url";
const ACCESS_KEY = "your-access-key";
```

### 5. 会社名を設定
Excel見積書の「発行元」欄に使われます。

```js
const COMPANY_NAME = "貴社名を入力してください";
```

必要であれば、ログイン画面の「現場帳場」表示（`<div class="auth-tagline">`）やヘッダー内の同表示（`<div class="eyebrow">`）も合わせて書き換えてください。

### 6. ホスティング
`index.html` / `manifest.json` / `sw.js` / `icons/` をまとめてHTTPS配信できる場所（Firebase Hosting、GitHub Pages等）にアップロードしてください。音声入力・PWAインストールにはHTTPSが必須です。

### 7. 最初の管理者を設定
このアプリには「管理者を新規作成する」機能はありません。以下の手順で最初の1人を管理者にしてください。
1. アプリの「新規登録」から通常のユーザーとしてアカウントを作成
2. Firebase Console →「Firestore Database」→ `mitsumori_users` コレクション → 該当ユーザーのドキュメントを開く
3. `isAdmin` フィールドを `true` に変更

以降は、アプリ右上の「⚙️ 管理」パネルから他のユーザーの管理者権限・表示/非表示を操作できます。
