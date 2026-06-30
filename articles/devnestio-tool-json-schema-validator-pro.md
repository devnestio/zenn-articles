---
title: "JSON Schema Validator を作った — Draft-07/2019-09/2020-12対応、スキーマ推論機能つき"
emoji: "✅"
type: "tech"
topics: ["claude", "ai", "devtools", "webdev", "javascript"]
published: true
---

## どんなツール？

JSONデータがスキーマ仕様に合っているか確認したいとき、通常はAjvなどのライブラリをインストールするか、オンラインサービスにデータを送信する必要がある。

**JSON Schema Validator** は、外部ライブラリ不要・データ送信なしで、ブラウザだけでJSONスキーマ検証ができるツール。

👉 https://json-schema-validator-abb.pages.dev

## 主な機能

### 対応スキーマドラフト
- JSON Schema Draft-07
- JSON Schema 2019-09
- JSON Schema 2020-12

### 対応キーワード（抜粋）

**型・値**: `type`、`enum`、`const`

**文字列**: `minLength`、`maxLength`、`pattern`、`format`

**数値**: `minimum`、`maximum`、`exclusiveMinimum`、`exclusiveMaximum`、`multipleOf`

**オブジェクト**: `properties`、`required`、`additionalProperties`、`patternProperties`、`minProperties`、`maxProperties`、`dependentRequired`

**配列**: `items`、`prefixItems`、`minItems`、`maxItems`、`uniqueItems`、`contains`

**組み合わせ**: `allOf`、`anyOf`、`oneOf`、`not`、`if/then/else`

### フォーマット検証
`email`、`date`、`time`、`date-time`、`uri`、`ipv4`、`ipv6`、`uuid`、`hostname`

### JSONからスキーマ推論
「Generate Schema from JSON」ボタンでJSONデータからスキーマを自動生成。

### サンプルスキーマ
6種類のサンプルをクリックで即ロード（User Profile、Product、Blog Post、Address、Invalid Example、oneOf Colors）。

## エラー例

```
入力JSON: { "name": "", "age": -5, "extra": "not allowed" }

スキーマ:
{
  "type": "object",
  "required": ["name", "age"],
  "properties": {
    "name": { "type": "string", "minLength": 1 },
    "age":  { "type": "integer", "minimum": 0 }
  },
  "additionalProperties": false
}

結果 (3 errors):
✗ name  : String length 0 < minLength 1
✗ age   : -5 < minimum 0
✗ extra : Additional property not allowed
```

## テスト

123件のテスト全通過。

```bash
node tests/test.js
# Total: 123 | Passed: 123 | Failed: 0
```

---

[devnestio](https://devnestio.pages.dev) - 無料の開発者ツール集
