---
title: "進数変換ツールをブラウザだけで作った話（2進・8進・10進・16進）"
emoji: "🔢"
type: "tech"
topics: ["javascript", "programming", "devtools", "binary", "webdev"]
published: false
---

## 作ったもの

入力すると即座に2進・8進・10進・16進すべてに変換するツール。カスタム基数（2〜36進）やASCII対応付き。

👉 https://number-system-converter.pages.dev

## 機能

- **リアルタイム変換** — どのフィールドを編集しても他が即更新
- **負の数対応** — `-` プレフィックスで完全サポート
- **4ビット区切り表示** — 2進数をニブル単位で読みやすく
- **数値属性** — ビット長・バイト長・素数判定・偶奇・2のべき乗判定
- **カスタム基数** — 2〜36進に変換
- **ASCII参照** — 入力値の文字・Unicodeコードポイント・印刷可否を表示
- 各フィールドのワンクリックコピー

## 変換のコア

JavaScriptの標準機能だけで実装しています。

```js
// 入力 → 数値
function parseValue(str, base) {
  const negative = str[0] === '-';
  const digits = negative ? str.slice(1) : str;
  const n = parseInt(digits, base);
  if (isNaN(n)) return null;
  return negative ? -n : n;
}

// 数値 → 各進数
const toBinary = n => n < 0 ? '-' + Math.abs(n).toString(2) : n.toString(2);
const toHex    = n => n < 0 ? '-' + Math.abs(n).toString(16).toUpperCase() : n.toString(16).toUpperCase();
```

カスタム基数は `0123456789ABCDEFGHIJKLMNOPQRSTUVWXYZ` の文字テーブルを使って自前実装。

## テスト

Node.js で155件のユニットテストを書いて全通過を確認してからデプロイしています。`parseValue`・`toBinary`・`isPrime`・`groupBinary`・`toCustomBase`・`getAsciiChar` などを個別にテスト。

## devnestio について

https://devnestio.pages.dev

フロントエンド・インフラ向けの無料ブラウザツール集。
