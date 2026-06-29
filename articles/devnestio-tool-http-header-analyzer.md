---
title: "HTTPヘッダーアナライザーをVanilla JSで作った話（セキュリティスコアリング付き、147テスト）"
emoji: "🔍"
type: "tech"
topics: ["claude", "ai", "devtools", "webdev", "security"]
published: false
---

## 作ったもの

開発者向け無料ツール群「devnestio」に **HTTP Header Analyzer** を追加しました。HTTPレスポンスヘッダーを貼り付けるだけで、セキュリティスコアの評価と不足ヘッダーの警告を表示するツールです。

👉 https://http-header-analyzer.pages.dev

## なぜ作ったか

本番サーバーのセキュリティヘッダーが正しく設定されているか確認したいとき、`curl -I` で取得したヘッダーをどう評価すればいいか迷うことがあります。`X-Content-Type-Options`は設定しているか、`Strict-Transport-Security`のmax-ageは十分か——チェック項目を手で確認するのは面倒です。ヘッダーを貼るだけでスコアリングしてくれるツールがほしかった。

## 主な機能

- **セキュリティスコアの算出** — 0〜100点、グレードA〜Fで表示
- **不足ヘッダーの一覧と推奨値** — `Content-Security-Policy`, `X-Frame-Options`, `HSTS`, `Referrer-Policy`など
- **各ヘッダーの値の解説** — max-age値の妥当性チェックなど
- **キャッシュ関連ヘッダーの分析** — `Cache-Control`, `ETag`, `Last-Modified`の組み合わせ評価
- **コンテンツタイプの確認** — `Content-Type`のcharset設定漏れを指摘

## 技術的なポイント：スコアリングロジック

セキュリティスコアはヘッダーの有無と値の品質を組み合わせて計算します。

```js
const SECURITY_HEADERS = {
  'strict-transport-security': { weight: 20, check: (v) => /max-age=(\d+)/.test(v) && parseInt(v.match(/max-age=(\d+)/)[1]) >= 31536000 },
  'content-security-policy':   { weight: 25, check: (v) => v.length > 0 },
  'x-frame-options':           { weight: 15, check: (v) => /^(DENY|SAMEORIGIN)$/i.test(v) },
  'x-content-type-options':    { weight: 10, check: (v) => /^nosniff$/i.test(v) },
  'referrer-policy':           { weight: 10, check: (v) => v.length > 0 },
  'permissions-policy':        { weight: 10, check: (v) => v.length > 0 },
};

function calcScore(headers) {
  let score = 0;
  for (const [name, { weight, check }] of Object.entries(SECURITY_HEADERS)) {
    const val = headers[name.toLowerCase()] || '';
    if (check(val)) score += weight;
  }
  return Math.min(100, score);
}
```

HSTSはmax-ageが1年（31536000秒）以上あって初めてフル点数にするなど、値の妥当性まで見ています。

## テスト結果

**147/147テスト PASS**。ヘッダー名の大文字小文字正規化、HSTSのmax-age境界値、CSP値の解析、グレード判定ロジックなどを網羅しています。

## 実装はAIエージェントに任せた

Claude Codeに発注書を渡しています。「スコアの根拠を表示すること」「どのヘッダーが何点分か明示すること」という要件を入れたことで、ユーザーが改善アクションを取りやすい出力になりました。

## 使ってみてください

👉 **[devnestio - Developer Tools Hub](https://devnestio.pages.dev)**

`curl -I https://yoursite.com` の結果をそのまま貼ってみてください。
