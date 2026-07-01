---
title: "RFC 4648 Base32エンコーダー/デコーダーをVanilla JSで実装"
emoji: "🔡"
type: "tech"
topics: ["javascript","encoding","webapi","browser","base32"]
published: false
---

## はじめに

Base32エンコーディング（RFC 4648）をブラウザのみで実装しました。TOTPの秘密鍵、DNSラベル、ファイルIDなどで広く使われています。

**ツール:** https://devnestio.pages.dev/base32-encoder/

## エンコード実装

```js
const STD_ALPHABET = 'ABCDEFGHIJKLMNOPQRSTUVWXYZ234567';
const HEX_ALPHABET = '0123456789ABCDEFGHIJKLMNOPQRSTUV';

function base32Encode(input, alphabet, addPadding) {
  const bytes = new TextEncoder().encode(input);
  let bits = 0, value = 0, out = '';
  for (const byte of bytes) {
    value = (value << 8) | byte;
    bits += 8;
    while (bits >= 5) {
      out += alphabet[(value >>> (bits - 5)) & 31];
      bits -= 5;
    }
  }
  if (bits > 0) out += alphabet[(value << (5 - bits)) & 31];
  if (addPadding) while (out.length % 8 !== 0) out += '=';
  return out;
}
```

## デコード実装

```js
function base32Decode(input, alphabet) {
  const clean = input.toUpperCase().replace(/=+$/, '').replace(/\s/g, '');
  let bits = 0, value = 0;
  const bytes = [];
  for (const c of clean) {
    const idx = alphabet.indexOf(c);
    if (idx < 0) throw new Error(`Invalid Base32 character: '${c}'`);
    value = (value << 5) | idx;
    bits += 5;
    if (bits >= 8) {
      bytes.push((value >>> (bits - 8)) & 0xff);
      bits -= 8;
    }
  }
  return new TextDecoder().decode(new Uint8Array(bytes));
}
```

## 仕組み

- 5ビット単位で処理（32 = 2^5）
- 8文字ごとに `=` でパディング（8 × 5 = 40ビット = 5バイト）
- オーバーヘッドは約60%

## 機能

- **アルファベット:** 標準（RFC 4648）/ 拡張Hex（Base32Hex）
- **パディング:** `=` の有無を選択可能
- **統計:** 入力バイト数、出力文字数、オーバーヘッド率、パディング文字数
- **参照テーブル:** 32文字すべての値を表示
