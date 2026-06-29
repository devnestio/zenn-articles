---
title: "数値基数変換ツールをVanilla JSで作った — BigInt で任意基数に対応"
emoji: "🔢"
type: tech
topics: ["javascript", "webdev", "tools", "programming"]
published: false
---

## これは何？

[DevNest.io](https://devnestio.pages.dev/) のツールとして作った数値基数変換ツールです。

**URL**: https://number-base-converter.pages.dev/

2進数・8進数・10進数・16進数をはじめ、基数2〜36の間で即座に変換します。BigIntを使って大きな整数も精度を失わずに変換できます。

## 実装

### 変換の核心: BigInt を使う

JavaScript の `Number` 型は53ビット整数しか正確に表現できません（`Number.MAX_SAFE_INTEGER = 2^53 - 1`）。基数変換ツールは大きな値を扱うことが多いので、`BigInt` を使って任意精度で計算します。

```js
const DIGITS = '0123456789abcdefghijklmnopqrstuvwxyz';

function parseNumber(str, base) {
  const s = str.trim();
  if (!s || !isValidForBase(s, base)) return null;
  const neg = s.startsWith('-');
  const digits = neg ? s.slice(1) : s;
  let value = BigInt(0);
  const bigBase = BigInt(base);
  for (const ch of digits.toLowerCase()) {
    const d = DIGITS.indexOf(ch);
    if (d < 0 || d >= base) return null;
    value = value * bigBase + BigInt(d);
  }
  return neg ? -value : value;
}

function toBase(value, base) {
  if (value === BigInt(0)) return '0';
  const neg = value < BigInt(0);
  let n = neg ? -value : value;
  const bigBase = BigInt(base);
  let result = '';
  while (n > BigInt(0)) {
    result = DIGITS[Number(n % bigBase)] + result;
    n = n / bigBase;
  }
  return (neg ? '-' : '') + result;
}
```

`DIGITS` は `"0123456789abcdefghijklmnopqrstuvwxyz"` — 基数36まで対応できます。

### バリデーション

```js
function isValidForBase(str, base) {
  const valid = DIGITS.slice(0, base);
  const re = new RegExp(`^-?[${valid}]+$`, 'i');
  return re.test(str.trim());
}
```

基数に対して許可される文字セットを動的に生成して正規表現でチェックします。大文字・小文字を区別しません。

### 2進数のニブル表示

```js
function groupBinary(binStr) {
  const neg = binStr.startsWith('-');
  const digits = neg ? binStr.slice(1) : binStr;
  const padded = digits.padStart(Math.ceil(digits.length / 4) * 4, '0');
  const nibbles = [];
  for (let i = 0; i < padded.length; i += 4) {
    nibbles.push(padded.slice(i, i + 4));
  }
  return (neg ? '-' : '') + nibbles.join(' ');
}
```

4ビット（ニブル）単位でスペース区切りして読みやすくします。`11111111` → `1111 1111`。

## 機能

- **9種の名前付き基数** — Binary(2), Ternary(3), Quaternary(4), Octal(8), Decimal(10), Duodecimal(12), Hexadecimal(16), Base32(32), Base36(36)
- **カスタム基数入力** — 2〜36の任意の基数を指定可能
- **負の数対応** — `-42` などを入力すると全基数で負の値として変換
- **ニブル表示** — 2進数を4ビット単位でグループ化
- **クイック入力** — よく使う値（0, 1, 42, 255, ff, 1010など）のクイックボタン
- **カードクリックでコピー** — 変換結果をクリップボードにコピー＆その基数で入力に反映

## テスト

Node.js `assert` モジュールで **122テスト** を実装、全パス確認済み。

主なテストカテゴリ:
- `isValidForBase` — 各基数での許可/拒否文字
- `parseNumber` — 全標準基数、ネガティブ、スペーストリム、大文字
- `toBase` — 0, 負の数、大きな値、全基数のラウンドトリップ
- `groupBinary` — ニブルグループ化、パディング、負符号の保持
- 基数間整合性（255 = ff = 377 = 11111111b）

## 使い方

https://number-base-converter.pages.dev/ にアクセスして、数値と入力基数を選んで入力するだけです。

---

DevNest.io では他にも開発者向けツールを公開しています。
