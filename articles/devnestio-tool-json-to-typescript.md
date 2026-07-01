---
title: "JSONをTypeScriptインターフェースに変換する — ネスト対応・配列型推論（86テスト）"
emoji: "🔷"
type: "tech"
topics: ["typescript", "javascript", "webdev", "devtools"]
published: false
---

## 作ったもの

開発者向け無料ツール群「devnestio」に、**JSON to TypeScript Interface Generator** を追加しました。

👉 https://devnestio.pages.dev/json-to-typescript/

JSONを貼り付けるだけでTypeScriptインターフェースが生成されます。

---

## 主な機能

- **型の自動推論** — string, number, boolean, null, ネストオブジェクト, 配列
- **ネストインターフェース** — オブジェクトプロパティは自動的にサブインターフェースを生成
- **配列型推論** — 最初の要素からアイテム型を推論、`T[]`形式で出力
- **オプション** — optional(`?`), readonly, export, セミコロン有無
- **ルート名変更** — デフォルト`Root`から任意の名前に変更可能
- **構文ハイライト** — キーワード紫・インターフェース名青・型名緑
- **自動変換** — 400ms後に自動実行（手動ボタンも有）

---

## 変換例

**入力JSON:**
```json
{
  "id": 42,
  "name": "Alice",
  "active": true,
  "profile": {
    "avatar": "url",
    "bio": "開発者"
  }
}
```

**生成されるTypeScript:**
```typescript
export interface Profile {
  avatar: string;
  bio: string;
}

export interface Root {
  id: number;
  name: string;
  active: boolean;
  profile: Profile;
}
```

---

## 型推論ルール

| JSON値 | TypeScript型 |
|--------|------------|
| `"hello"` | `string` |
| `42`, `3.14` | `number` |
| `true`/`false` | `boolean` |
| `null` | `null` |
| `[...]` | `ItemType[]` |
| `{}` | 生成インターフェース |
| `[]` (空) | `unknown[]` |

---

## テスト

**86 / 86 PASS ✅**

全型推論、ネストインターフェース生成、配列型、オプション組み合わせ、エラー処理をテスト。

👉 https://devnestio.pages.dev
