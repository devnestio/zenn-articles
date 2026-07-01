---
title: "ブラウザだけで動くOpenAPI/Swaggerバリデーターを作った"
emoji: "📋"
type: "tech"
topics: ["javascript","openapi","swagger","webdev","browser"]
published: false
---

## はじめに

OpenAPIドキュメントのバリデーションは通常サーバー側ツールが必要です。ブラウザのみで動作するバリデーターを実装しました。

**ツール:** https://devnestio.pages.dev/openapi-validator/

## 対応バージョン

- OpenAPI 3.0.x / 3.1.x（JSON / YAML）
- Swagger 2.0（JSON / YAML）

## バリデーション内容

### 共通
- ルートフィールド必須チェック（`openapi`/`swagger`, `info`, `paths`）
- `info.title` と `info.version` の存在確認
- パスが `/` で始まるかどうか

### OpenAPI 3.x 追加チェック
- `operationId` の重複検出
- HTTPステータスコードの妥当性
- `servers` セクションの存在確認
- スキーマに `type` または合成キーワードがあるか

## 実装

```js
function validateV3(doc) {
  required(doc, ['openapi', 'info', 'paths'], '#');
  const operationIds = new Set();

  for (const [path, item] of Object.entries(doc.paths || {})) {
    if (!path.startsWith('/')) {
      addIssue('error', `#/paths/${path}`, 'Path must start with /', '');
    }
    for (const method of ['get','post','put','delete','patch','trace']) {
      const op = item[method];
      if (!op) continue;
      const loc = `#/paths/${path}/${method}`;

      if (!op.responses) {
        addIssue('error', loc, 'Operation missing responses', '');
      }
      if (op.operationId) {
        if (operationIds.has(op.operationId)) {
          addIssue('error', loc, 'Duplicate operationId', op.operationId);
        } else {
          operationIds.add(op.operationId);
        }
      } else {
        addIssue('info', loc, 'Missing operationId', '');
      }
    }
  }
}
```

## 簡易YAMLパーサー

外部ライブラリを使わずに基本的なYAMLを解析します。

```js
function parseYaml(str) {
  const lines = str.split('\n');
  const root = {};
  const stack = [{ obj: root, indent: -1 }];
  for (const rawLine of lines) {
    const indent = rawLine.search(/\S/);
    const line = rawLine.trim();
    if (line.startsWith('- ')) {
      // 配列要素
    } else if (line.includes(':')) {
      const [key, ...rest] = line.split(':');
      // オブジェクトフィールド
    }
  }
  return root;
}
```

## 表示機能

- エラー・警告・情報の件数サマリー
- APIサマリー（タイトル、バージョン、パス数、操作数）
- エンドポイント一覧（メソッドバッジ付き）
- 問題ごとのJSONパス表示
