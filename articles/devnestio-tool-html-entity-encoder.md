---
title: "HTMLエンティティエンコーダー/デコーダーをVanilla JSで作った"
emoji: "🔤"
type: tech
topics: ["javascript", "html", "webdev", "tools"]
published: false
---

## これは何？

[DevNest.io](https://devnestio.pages.dev/) のツールとして作ったHTMLエンティティエンコーダー/デコーダーです。

**URL**: https://html-entity-encoder.pages.dev/

HTMLの特殊文字をエンティティに変換（エンコード）し、逆にエンティティを文字に戻す（デコード）ツールです。名前付きエンティティ・10進数・16進数・全文字エンコードの4モードに対応。

## 実装

### 名前付きエンティティマップ

```js
const NAMED = {
  '&': 'amp', '<': 'lt', '>': 'gt', '"': 'quot', "'": 'apos',
  '©': 'copy', '®': 'reg', '™': 'trade', '°': 'deg',
  '±': 'plusmn', '×': 'times', '÷': 'divide',
  '€': 'euro', '£': 'pound', '¥': 'yen', '¢': 'cent',
  '–': 'ndash', '—': 'mdash', '…': 'hellip',
  '→': 'rarr', '•': 'bull',
  // ...
};

const NAMED_TO_CHAR = Object.fromEntries(
  Object.entries(NAMED).map(([ch, name]) => [name, ch])
);
```

マップを逆引きするため `Object.fromEntries` でデコード用マップも生成。

### エンコード4モード

```js
// 名前付き: & → &amp;  © → &copy;
function encodeHtmlNamed(text) {
  return text.replace(/[&<>"' ©®™...]/g, ch =>
    NAMED[ch] ? `&${NAMED[ch]};` : ch
  );
}

// 10進数: & → &#38;
function encodeHtmlDecimal(text) {
  return text.replace(/[&<>"']/g, ch => `&#${ch.charCodeAt(0)};`);
}

// 16進数: & → &#x26;
function encodeHtmlHex(text) {
  return text.replace(/[&<>"']/g, ch => `&#x${ch.charCodeAt(0).toString(16)};`);
}

// 全文字: ASCII以外をすべてエンコード
function encodeHtmlAll(text) {
  let out = '';
  for (const ch of text) {
    const cp = ch.codePointAt(0);
    out += cp > 127 || /[&<>"']/.test(ch) ? `&#x${cp.toString(16)};` : ch;
  }
  return out;
}
```

`encodeHtmlAll` は `for...of` でサロゲートペア対応のコードポイントを反復し、`codePointAt(0)` で取得します。

### デコード

```js
function decodeHtml(text) {
  return text
    .replace(/&([a-zA-Z]+);/g, (_, name) =>
      name in NAMED_TO_CHAR ? NAMED_TO_CHAR[name] : `&${name};`
    )
    .replace(/&#x([0-9a-fA-F]+);/g, (_, hex) =>
      String.fromCodePoint(parseInt(hex, 16))
    )
    .replace(/&#([0-9]+);/g, (_, dec) =>
      String.fromCodePoint(parseInt(dec, 10))
    );
}
```

3段階の正規表現で名前付き・16進数・10進数エンティティをすべて処理。未知の名前付きエンティティはそのまま保持します。

## 機能

- **4つのエンコードモード** — Named / Decimal / Hex / Encode All
- **デコード** — 3種のエンティティ形式を自動判定して復元
- **双方向ペイン** — Input / Output を並列表示
- **統計情報** — 入力文字数 / 出力文字数 / エンティティ数 / サイズ比
- **エンティティ参照テーブル** — よく使う17エンティティをクリックで挿入
- **Swap** — 入出力を入れ替えてデコードに再利用

## テスト

Node.js `assert` モジュールで **120テスト** を実装、全パス確認済み。

テストカテゴリ:
- `NAMED` マップ — 主要エントリの確認
- `NAMED_TO_CHAR` — 逆引きマップ、往復一致
- `encodeHtmlNamed` — 5つの必須特殊文字、記号、タグ全体
- `encodeHtmlDecimal` — コードポイント確認
- `encodeHtmlHex` — 16進数フォーマット確認
- `encodeHtmlAll` — 非ASCII文字エンコード、ASCII文字保持
- `encodeHtml` — 全モードディスパッチ、未知モードフォールバック
- `decodeHtml` — 名前付き/10進/16進、混在、未知エンティティ保持
- Roundtrip — 全4モードで encode→decode→original
- `countEntities` — 各エンティティ形式
- `hasHtmlChars` — 5文字の特殊文字

## 使い方

https://html-entity-encoder.pages.dev/ にアクセスして、テキストまたはHTMLを貼り付けて「Encode」をクリックしてください。

---

DevNest.io では他にも開発者向けツールを公開しています。
