---
title: "バイト・ビット変換ツールをVanilla JSで作った — IEC vs SI 両対応"
emoji: "💾"
type: tech
topics: ["javascript", "webdev", "tools", "programming"]
published: false
---

## これは何？

[DevNest.io](https://devnestio.pages.dev/) のツールとして作ったバイト＆ビット変換ツールです。

**URL**: https://byte-bit-converter.pages.dev/

bit / byte / KiB / MiB / GiB / TiB / PiB / EiB と KB / MB / GB / TB / PB / EB の間を即座に変換します。二進法（IEC、÷1024）と十進法（SI、÷1000）の両方のモードに対応しています。

## IEC と SI の違い

「1 GB = 1,000,000,000 バイト」か「1,073,741,824 バイト」か — どちらも正解です。

| 規格 | 単位 | 基数 | 例 |
|------|------|------|-----|
| IEC 80000-13 | KiB, MiB, GiB... | 1024 | 1 KiB = 1,024 B |
| SI / 国際単位系 | KB, MB, GB... | 1000 | 1 KB = 1,000 B |

ハードディスクメーカーはSI（1TB = 10^12バイト）を使うのに対し、OSのファイルシステムはIEC（1 TiB = 2^40バイト）を使うため、"1 TB のHDD" がOSでは約931 GiBと表示される差が生まれます。

## 実装

### ユニット定義

```js
const BINARY_UNITS = [
  { name: 'Bit',      abbr: 'b',   factor: 1/8 },
  { name: 'Byte',     abbr: 'B',   factor: 1 },
  { name: 'Kibibyte', abbr: 'KiB', factor: 1024 },
  { name: 'Mebibyte', abbr: 'MiB', factor: 1024 ** 2 },
  { name: 'Gibibyte', abbr: 'GiB', factor: 1024 ** 3 },
  { name: 'Tebibyte', abbr: 'TiB', factor: 1024 ** 4 },
  { name: 'Pebibyte', abbr: 'PiB', factor: 1024 ** 5 },
  { name: 'Exbibyte', abbr: 'EiB', factor: 1024 ** 6 },
];

const DECIMAL_UNITS = [
  { name: 'Bit',      abbr: 'b',  factor: 1/8 },
  { name: 'Byte',     abbr: 'B',  factor: 1 },
  { name: 'Kilobyte', abbr: 'KB', factor: 1e3 },
  { name: 'Megabyte', abbr: 'MB', factor: 1e6 },
  { name: 'Gigabyte', abbr: 'GB', factor: 1e9 },
  { name: 'Terabyte', abbr: 'TB', factor: 1e12 },
  { name: 'Petabyte', abbr: 'PB', factor: 1e15 },
  { name: 'Exabyte',  abbr: 'EB', factor: 1e18 },
];
```

各ユニットに「バイト換算係数（factor）」を持たせ、変換は常にバイトを中間単位として行います。

### 変換ロジック

```js
function toBytes(value, unit) { return value * unit.factor; }
function fromBytes(bytes, unit) { return bytes / unit.factor; }

function convert(value, fromUnit, units) {
  const bytes = toBytes(value, fromUnit);
  return units.map(u => ({ ...u, result: fromBytes(bytes, u) }));
}
```

たった3関数で完結します。`convert` は「入力値をバイトに変換し、全ユニット分のバイトからの変換結果を返す」だけです。

### 結果のフォーマット

```js
function formatResult(n) {
  if (n === 0) return '0';
  if (!isFinite(n)) return '∞';
  if (n < 1e-6) return n.toExponential(4);
  if (n >= 1e15) return n.toExponential(4);
  if (Number.isInteger(n)) return n.toLocaleString('en-US');
  const s = parseFloat(n.toPrecision(10)).toString();
  const [int, dec] = s.split('.');
  const intFmt = parseInt(int, 10).toLocaleString('en-US');
  return dec ? `${intFmt}.${dec}` : intFmt;
}
```

- 極小値（< 1e-6）と極大値（>= 1e15）は指数表記
- 整数は `toLocaleString` でカンマ区切り（`1,024,000`）
- 小数は10有効桁で丸めた後、整数部だけカンマ区切り

### クイックリファレンステーブル

ビルド時に隣接するユニット間の比率を計算して表示します:

```js
const rows = units.slice(1).map((u, i) => {
  const prev = units[i];
  const ratio = u.factor / prev.factor;
  return `<tr>
    <td>1 ${u.abbr}</td>
    <td>= ${ratio.toLocaleString('en-US')} ${prev.abbr}</td>
    <td>= ${u.factor.toLocaleString('en-US')} B</td>
  </tr>`;
});
```

## テスト

Node.js `assert` モジュールで **123テスト** を実装、全パス確認済み。

主なテストカテゴリ:
- ユニット定義（8エントリ、factor の正確性）
- `toBytes` / `fromBytes` の往復確認
- `convert` の全ユニット出力
- `formatResult` の境界値（0, Infinity, 1e-7, 1e15, 整数, 小数）
- IEC vs SI の差分（1 KiB - 1 KB = 24 バイト、1 MiB - 1 MB = 48,576 バイト）
- 連続するユニット間の比率（binary: 1024, decimal: 1000）

## 使い方

https://byte-bit-converter.pages.dev/ にアクセスして、値を入力し単位を選択するだけです。モード切り替えボタンで Binary (IEC) / Decimal (SI) を切り替えられます。結果カードをクリックするとその値をクリップボードにコピーできます。

---

DevNest.io では他にも開発者向けツールを公開しています。
