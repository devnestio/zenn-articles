---
title: "JSONフォーマッター & バリデーターをVanilla JSで作った"
emoji: "📋"
type: tech
topics: ["javascript", "webdev", "json", "tools"]
published: false
---

## これは何？

[DevNest.io](https://devnestio.pages.dev/) のツールとして作ったJSONフォーマッター & バリデーターです。

**URL**: https://json-formatter.pages.dev/

JSONの整形（Beautify）・圧縮（Minify）・キーのアルファベット順ソート・バリデーションをブラウザ内で行います。データは一切サーバーに送信されません。

## 実装

### JSONのパース

```js
function parseJSON(text) {
  try {
    const val = JSON.parse(text);
    return { ok: true, value: val, error: null };
  } catch(e) {
    return { ok: false, value: null, error: e.message };
  }
}
```

`JSON.parse` をそのまま使い、成功/失敗を `{ok, value, error}` でラップします。エラーメッセージも返すので、UIに「line 3: Unexpected token」のように表示できます。

### フォーマット & 圧縮

```js
function formatJSON(value, indent) {
  const spaces = indent === 'tab' ? '\t' : Number(indent);
  return JSON.stringify(value, null, spaces);
}

function minifyJSON(value) {
  return JSON.stringify(value);
}
```

`JSON.stringify` の第3引数でインデントを指定。`'tab'` のとき `'\t'` を渡すとタブインデントになります。

### キーの再帰ソート

```js
function sortKeysDeep(value) {
  if (Array.isArray(value)) return value.map(sortKeysDeep);
  if (value !== null && typeof value === 'object') {
    const sorted = {};
    for (const key of Object.keys(value).sort()) {
      sorted[key] = sortKeysDeep(value[key]);
    }
    return sorted;
  }
  return value;
}
```

- 配列: 各要素を再帰的に処理（配列自体の順序は変えない）
- オブジェクト: `Object.keys().sort()` でキーをソートして新しいオブジェクトに詰め直す
- プリミティブ: そのまま返す

### 統計情報の計算

```js
function countKeys(value) {
  if (Array.isArray(value)) return value.reduce((s, v) => s + countKeys(v), 0);
  if (value !== null && typeof value === 'object') {
    const keys = Object.keys(value);
    return keys.length + keys.reduce((s, k) => s + countKeys(value[k]), 0);
  }
  return 0;
}

function maxDepth(value, d = 0) {
  if (Array.isArray(value)) {
    if (value.length === 0) return d + 1;
    return Math.max(...value.map(v => maxDepth(v, d + 1)));
  }
  if (value !== null && typeof value === 'object') {
    const vals = Object.values(value);
    if (vals.length === 0) return d + 1;
    return Math.max(...vals.map(v => maxDepth(v, d + 1)));
  }
  return d;
}

function countValues(value) {
  if (Array.isArray(value)) return value.reduce((s, v) => s + countValues(v), 0);
  if (value !== null && typeof value === 'object') {
    return Object.values(value).reduce((s, v) => s + countValues(v), 0);
  }
  return 1;
}
```

`maxDepth` は空のオブジェクト/配列を深さ +1 として扱います（コンテナ自体が1層）。

### 型判定

```js
function getType(value) {
  if (value === null) return 'null';
  if (Array.isArray(value)) return 'array';
  return typeof value;
}
```

`typeof null === 'object'` と `typeof [] === 'object'` のJSの挙動を補正して正確な型名を返します。

## 機能

- **Format / Beautify** — 2スペース / 4スペース / タブのインデントを選択可能
- **Minify** — 空白・改行を除去してコンパクトに
- **Sort Keys** — オブジェクトキーをアルファベット順に再帰ソート
- **リアルタイムバリデーション** — 入力するたびに Valid / Invalid を表示
- **統計パネル** — Type / Keys数 / Values数 / Objects数 / Arrays数 / 最大深度
- **2ペインUI** — Input / Output を並列表示
- **クリップボード操作** — Paste / Copy Input / Copy Output / Swap

## テスト

Node.js `assert` モジュールで **124テスト** を実装、全パス確認済み。

テストカテゴリ:
- `parseJSON` — 全JSONリテラル型、無効パターン（末尾カンマ、シングルクォート等）
- `formatJSON` — インデント2/4/タブ、各型
- `minifyJSON` — 空白除去、ラウンドトリップ
- `sortKeysDeep` — ネスト、配列内オブジェクト、プリミティブ
- `countKeys` — 再帰カウント、配列混在
- `maxDepth` — 空コンテナ、深いネスト
- `countValues` — プリミティブ、ネスト
- `countArrays` / `countObjects` — 再帰カウント
- `getType` — null/array/object/number/string/boolean
- Integration — サンプルオブジェクトのroundtrip

## 使い方

https://json-formatter.pages.dev/ にアクセスして、JSONを貼り付けてFormatボタンを押してください。

---

DevNest.io では他にも開発者向けツールを公開しています。
