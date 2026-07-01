---
title: "Base64エンコーダー/デコーダーをブラウザで作った（ファイル・URLセーフ対応）"
emoji: "🔄"
type: "idea"
topics: ["javascript", "tools", "webdev", "encoding"]
published: false
---

# Base64 Encoder / Decoder

テキストとファイルのBase64エンコード・デコードに対応したブラウザツールを作りました。

https://devnestio.pages.dev/base64-tool/

## 機能

- **テキストモード** — 文字列のエンコード・デコード
- **ファイルモード** — ドラッグ&ドロップでファイルをBase64化
- **URLセーフモード** — `+`→`-`、`/`→`_`に変換
- **MIME改行** — 76文字ごとに改行（RFC 2045準拠）
- **文字数カウント** — 入出力それぞれの文字数を表示
- **コピーボタン** — ワンクリックでクリップボードへ

## UTF-8対応

絵文字や日本語などのマルチバイト文字を正しく処理するため、`TextEncoder`を通してバイト列に変換してからBase64化：

```js
function toBase64(str, urlSafe, lineBreaks) {
  const bytes = new TextEncoder().encode(str);
  let b64 = btoa(String.fromCharCode(...bytes));
  if (urlSafe) b64 = b64.replace(/\+/g, '-').replace(/\//g, '_');
  if (lineBreaks) b64 = b64.match(/.{1,76}/g).join('\n');
  return b64;
}
```

デコードは`TextDecoder`で逆変換。パディング（`=`）は自動補完。

## Base64の基本情報

| 項目 | 値 |
|------|-----|
| 文字セット | A–Z, a–z, 0–9, +, / |
| URLセーフ | A–Z, a–z, 0–9, -, _ |
| パディング | =（1〜2文字） |
| サイズ増加 | 約33%大きくなる |
| MIME行長 | 76文字 |
| 1文字あたり | 6ビット |

---

[DevNestio](https://devnestio.pages.dev/) — ブラウザで使える開発者向けツール集
