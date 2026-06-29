---
title: "JSONスキーマバリデーターをゼロ依存で作った話（Draft-07対応、173テスト）"
emoji: "📋"
type: "tech"
topics: ["claude", "ai", "devtools", "webdev", "json"]
published: false
---

## 作ったもの

開発者向け無料ツール群「devnestio」に **JSON Schema Validator** を追加しました。JSON SchemaのDraft-07仕様に準拠したバリデーターで、外部ライブラリを一切使わずブラウザ内だけで動作します。

👉 https://json-schema-validator-dev.pages.dev

## なぜ作ったか

APIのリクエスト／レスポンスのスキーマ定義を書くとき、「このスキーマで本当にバリデーションが通るか」を手軽に確認したいことがあります。既存のオンラインツールは`ajv`などの外部ライブラリを読み込んでいることが多く、ページが重い。ブラウザ完結の軽量版を作りたかったのが動機です。

## 主な機能

- **JSON Schema Draft-07の主要キーワード対応** — `type`, `properties`, `required`, `minimum`, `maximum`, `minLength`, `maxLength`, `pattern`, `enum`, `anyOf`, `allOf`, `oneOf`, `not`, `$ref`
- **エラー箇所のパス表示** — `$.user.age` のようにJSONPointer形式で指摘
- **サンプルスキーマ集** — ユーザープロフィール、商品、住所などのテンプレート付き
- **リアルタイムバリデーション** — 入力と同時に結果を表示

## 技術的なポイント：$refの再帰解決

JSON Schemaで最も実装が難しいのが`$ref`による参照解決です。スキーマが自己参照（再帰的な木構造など）を持つ場合、解決時に無限ループに陥る危険があります。

```js
function resolveRef(ref, rootSchema, visited = new Set()) {
  if (visited.has(ref)) return {}; // 循環参照ガード
  visited.add(ref);
  
  if (ref.startsWith('#/')) {
    const path = ref.slice(2).split('/');
    let node = rootSchema;
    for (const key of path) node = node?.[key];
    return node ? resolveRef(node, rootSchema, visited) : {};
  }
  return {};
}
```

`visited`セットで訪問済み参照を記録し、循環が検出されたら空オブジェクトを返すことで安全に処理しています。

## テスト結果

**173/173テスト PASS**。必須フィールド欠落・型エラー・数値範囲外・パターン不一致・`$ref`循環参照など、実際のAPIスキーマで発生しやすいケースを幅広くカバーしています。

## 実装はAIエージェントに任せた

Claude Codeに発注書を渡して実装してもらっています。`$ref`の再帰解決と循環参照ガードを要件に明示したことで、エッジケースまで対応した実装が得られました。

## 使ってみてください

👉 **[devnestio - Developer Tools Hub](https://devnestio.pages.dev)**

APIスキーマの動作確認やJSON Schemaの学習にどうぞ。
