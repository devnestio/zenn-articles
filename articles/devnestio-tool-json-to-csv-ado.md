---
title: "ネストオブジェクトのフラット化対応 JSON↔CSVコンバーター"
emoji: "🔄"
type: "tech"
topics: ["javascript","json","csv","webdev","browser"]
published: false
---

## はじめに

JSONとCSVの相互変換ツールをブラウザのみで実装しました。ネストされたオブジェクトのフラット化にも対応しています。

**ツール:** https://devnestio.pages.dev/json-to-csv/

## ネストオブジェクトのフラット化

```js
function flattenObj(obj, prefix) {
  const out = {};
  for (const [k, v] of Object.entries(obj)) {
    const key = prefix ? prefix + '.' + k : k;
    if (v !== null && typeof v === 'object' && !Array.isArray(v)) {
      Object.assign(out, flattenObj(v, key));
    } else {
      out[key] = v === null ? '' : Array.isArray(v) ? JSON.stringify(v) : String(v);
    }
  }
  return out;
}
```

`{"address": {"city": "Tokyo"}}` → `address.city` カラムに変換されます。

## CSVフィールドのクォート処理

```js
function escapeCsvField(val, delim, q) {
  if (!q) return val;
  const needsQuote = val.includes(delim) || val.includes(q) || val.includes('\n');
  if (!needsQuote) return val;
  return q + val.replace(new RegExp(q, 'g'), q + q) + q;
}
```

デリミタ、クォート文字、改行を含むフィールドを自動でクォートします。

## CSV → JSONのパーサー

```js
function parseCsvLine(line, delim, q) {
  const fields = [];
  let cur = '', inQ = false;
  for (let i = 0; i < line.length; i++) {
    const c = line[i];
    if (q && c === q) {
      if (inQ && line[i+1] === q) { cur += q; i++; } // エスケープされたクォート
      else inQ = !inQ;
    } else if (!inQ && c === delim) {
      fields.push(cur); cur = '';
    } else cur += c;
  }
  fields.push(cur);
  return fields;
}
```

## 機能

- **JSON → CSV:** フラット化、ヘッダー行、複数デリミタ（`,` `;` `\t` `|`）
- **CSV → JSON:** ヘッダー行あり/なし対応
- **クォート:** ダブル・シングル・なし
- **BOM:** Excel用のBOM（`﻿`）付加オプション
- **プレビュー:** 先頭10行をテーブル表示
- **ダウンロード:** `.csv` / `.json` で保存
