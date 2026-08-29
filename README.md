# Yuki Proxy

Cloudflare Workers で動く汎用Webプロキシ。任意のURLをWorker経由で取得・表示する。

## 構成

```
worker.js    Cloudflare Workers本体(プロキシ処理)
proxy.html   フロントエンド(ホーム画面 / ショートカット付き)
```

## worker.js

`?url=` にターゲットURLを渡すと、そのサイトを取得して返す。

```
https://yuki-proxy.shimoshimo0204.workers.dev/?url=https://example.com
```

### やっていること

- HTML内の `href` / `src` / `action` / `srcset` をすべてプロキシ経由URLに書き換え
- CSSの `url()` も書き換え(背景画像など)
- リダイレクト(301/302/303/307/308)を追跡し、Locationヘッダーもプロキシ経由に変換
- レスポンスの `Content-Security-Policy` / `X-Frame-Options` を除去(表示崩れ防止)
- CORSヘッダーを付与

### デプロイ

1. Cloudflareダッシュボード → **Workers & Pages** → **Create** → **Create Worker**
2. `worker.js` の中身を全部貼り付けて **Deploy**
3. 動作確認: `https://<your-worker>.workers.dev/?url=https://example.com`

wrangler CLIを使う場合:

```bash
wrangler deploy
```

### 既知の制限

- JS内で動的に `fetch()` するSPA的なサイトは、プロキシを経由しない絶対パスのAPIコールが失敗することがある
- Cookie認証が必要なサイトは通らないことが多い
- 個人・検証用途向け。無制限の公開プロキシとして使うとサービス規約に触れる可能性あり

## proxy.html

Chromeの新しいタブ風のホーム画面。ショートカットタイルをクリックするか、URLを直接入力してWorker経由でサイトを開く。

### 設定項目(ファイル内で編集)

`<script>` 内、先頭付近の定数を編集する。

```javascript
const WORKER_ORIGIN = 'https://yuki-proxy.shimoshimo0204.workers.dev'; // Worker URL
const ICON_BASE = 'https://tanabesan.github.io/proxy/file/';          // アイコン画像の置き場所
```

### ショートカットの追加・編集

`favorites` 配列を直接編集する。`icon` はアイコン画像のファイル名(`ICON_BASE` 配下)。画像が見つからない場合は自動で頭文字アイコンにフォールバックする。

```javascript
let favorites = [
  { name: 'YouTube',  url: 'https://www.youtube.com', icon: 'youtube.png' },
  { name: 'Google',   url: 'https://www.google.com',  icon: 'google.png' },
  { name: 'LOLBeans', url: 'https://lolbeans.io',      icon: 'lolbeans.png' },
  { name: 'Wikipedia',url: 'https://ja.wikipedia.org', icon: 'wikipedia.png' },
];
```

- 画面内の「+ 追加」タイルから、TARGET URL欄の内容をその場で追加することも可能(画像なし、頭文字アイコン表示)
- 追加したショートカットは**ページを再読み込みすると消える**(セッション内のみ保持。永続化はしていない)

### 使い方

1. `proxy.html` をブラウザで開く(またはどこかにホスティングする)
2. ショートカットをクリック、またはTARGET URL欄にURLを入力して「経路を開く」

## 今後の候補

- ショートカットの永続化(localStorage or Firestore)
- SPA対応(Service Worker注入によるfetchフック)
