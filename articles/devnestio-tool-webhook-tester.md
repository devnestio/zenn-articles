---
title: "Webhook Tester & Inspector を作った — HMAC検証、ペイロード解析、12種テンプレート"
emoji: "🪝"
type: "tech"
topics: ["claude", "ai", "devtools", "webdev", "javascript"]
published: true
---

## どんなツール？

Webhookのテストは面倒だ。受信エンドポイントを公開しないといけないし、ngrokを立てたり、サードパーティサービスに頼ったりする必要がある。

**Webhook Tester & Inspector** は、ブラウザだけでWebhookのビルド・送信・ペイロード解析・HMAC署名検証ができるツール。

👉 https://webhook-tester-abd.pages.dev

## 主な機能

### ビルダー & 送信
12種類のプラットフォームテンプレートから選択：

| テンプレート | 内容 |
|------------|------|
| GitHub Push | pushイベント |
| GitHub PR | pull_requestイベント |
| Stripe Payment | payment_intent.succeeded |
| Stripe Subscription | customer.subscription.created |
| Slack Command | スラッシュコマンド |
| Slack Event | message event |
| Shopify Order | order/created |
| SendGrid | email配信イベント |
| Twilio SMS | 着信SMS |
| Generic Event | 汎用イベント |
| Ping / Health | ヘルスチェック |
| Jira Issue | issue作成 |

ブラウザから実際にHTTPリクエストを送信（CORS対応エンドポイントまたは自前のAPIに有効）。

### ペイロードインスペクター
JSONペイロードを貼り付けると即分析：
- ルート型、トップレベルキー数
- 最大ネスト深度
- 総ノード数
- リーフフィールド数
- ペイロードサイズ
- 全パスのフラット化表示（例: `data.object.amount = 2000`）
- シンタックスハイライト表示

### HMAC署名検証
GitHub（X-Hub-Signature-256）、Stripe、Shopifyなどのサービスが送ってくるHMAC署名を検証：

```
シークレット: webhooksecret
ペイロード:   {"action":"push"}
アルゴリズム: HMAC-SHA256
プレフィックス: sha256=

計算結果: sha256=a7f3c9b...
受信署名: sha256=a7f3c9b...   ✓ 一致
```

また逆に、自前のWebhookに署名を付与する場合の署名生成にも対応。

Web Crypto APIを使用するため、シークレットキーはブラウザ外に出ない。

### 送信履歴
送信したリクエストをlocalStorageに記録。URLとペイロードを再ロードできる。

## 実装のポイント

### HMAC署名計算（Web Crypto API）

```javascript
async function hmacSign(secret, message, algo = 'SHA-256') {
  const enc = new TextEncoder();
  const key = await crypto.subtle.importKey(
    'raw', enc.encode(secret),
    { name: 'HMAC', hash: algo },
    false, ['sign']
  );
  const sig = await crypto.subtle.sign('HMAC', key, enc.encode(message));
  return Array.from(new Uint8Array(sig))
    .map(b => b.toString(16).padStart(2, '0'))
    .join('');
}
```

サーバー送信なしで完全にブラウザ内で完結。

### ペイロードのパス解析

```javascript
function analyzePayload(data, prefix = '', paths = []) {
  if (Array.isArray(data)) {
    data.forEach((v, i) => analyzePayload(v, `${prefix}[${i}]`, paths));
    paths.push({ path: prefix || '(root)', type: 'array', value: `[${data.length} items]` });
  } else if (data !== null && typeof data === 'object') {
    Object.entries(data).forEach(([k, v]) =>
      analyzePayload(v, prefix ? `${prefix}.${k}` : k, paths)
    );
    if (prefix) paths.push({ path: prefix, type: 'object', value: `{...}` });
  } else {
    paths.push({ path: prefix || '(root)', type: typeof data, value: String(data) });
  }
  return paths;
}
```

再帰でネストした構造を全てドット記法に展開。

## テスト

79件のテスト全通過。

```bash
node tests/test.js
# Total: 79 | Passed: 79 | Failed: 0
```

---

[devnestio](https://devnestio.pages.dev) - 無料の開発者ツール集
