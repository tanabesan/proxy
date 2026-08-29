# Yuki Proxy

Cloudflare Workers で動く汎用Webプロキシ。

## 構成

```
worker.js    Cloudflare Workers本体
proxy.html   ホーム画面(ショートカット付き)
```

## デプロイ

1. Cloudflareダッシュボード → **Workers & Pages** → **Create** → **Create Worker**
2. `worker.js` の中身を貼り付けて **Deploy**

## 使い方

`proxy.html` を開いて、ショートカットをクリックするかTARGET URLを入力して開く。

## 既知の制限

- 一部のSPA的なサイトはうまく動かないことがある
- Cookie認証が必要なサイトは通らないことが多い
