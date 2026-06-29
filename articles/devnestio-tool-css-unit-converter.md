---
title: "CSSユニットコンバーターをVanilla JSで作った"
emoji: "📐"
type: tech
topics: ["javascript", "css", "webdev", "tools"]
published: false
---

## これは何？

[DevNest.io](https://devnestio.pages.dev/) のツールとして作ったCSSユニットコンバーターです。

**URL**: https://css-unit-converter.pages.dev/

px・em・rem・%・vw・vh・pt・pc・cm・mm・in の11単位を相互変換します。ベースフォントサイズ、ビューポートサイズ、親要素サイズをカスタマイズでき、実際のプロジェクト環境に合わせた変換が可能です。

## 実装

### 物理単位の定数

```js
const PX_PER_PT = 96 / 72;         // CSS: 1pt = 1/72in, 1in = 96px
const PX_PER_PC = PX_PER_PT * 12;  // 1pc = 12pt
const PX_PER_CM = 96 / 2.54;       // 1in = 2.54cm
const PX_PER_MM = PX_PER_CM / 10;
const PX_PER_IN = 96;
```

CSS仕様の基準ピクセルは `1in = 96px`。そこから他の物理単位を導出します。`pt` はDTPの伝統的な単位（1pt = 1/72in）、`pc`（パイカ）は12ptです。

### 任意単位 → px 変換

```js
function toPx(value, unit, ctx) {
  switch (unit) {
    case 'px':  return value;
    case 'em':  return value * ctx.baseFontPx;
    case 'rem': return value * ctx.baseFontPx;
    case 'pct': return value / 100 * ctx.parentPx;
    case 'vw':  return value / 100 * ctx.vpWidth;
    case 'vh':  return value / 100 * ctx.vpHeight;
    case 'pt':  return value * PX_PER_PT;
    case 'pc':  return value * PX_PER_PC;
    case 'cm':  return value * PX_PER_CM;
    case 'mm':  return value * PX_PER_MM;
    case 'in':  return value * PX_PER_IN;
    default:    return null;
  }
}
```

コンテキスト依存単位（em・rem・%・vw・vh）は `ctx` オブジェクトの設定値を使用。物理単位（pt・pc・cm・mm・in）は定数で計算するためコンテキスト不要。

### px → 任意単位 変換

```js
function fromPx(px, unit, ctx) {
  switch (unit) {
    case 'em':  return px / ctx.baseFontPx;
    case 'rem': return px / ctx.baseFontPx;
    case 'pct': return px / ctx.parentPx * 100;
    case 'vw':  return px / ctx.vpWidth * 100;
    case 'vh':  return px / ctx.vpHeight * 100;
    // ...
  }
}
```

### 全単位への一括変換

```js
function convertAll(value, fromUnit, ctx) {
  const px = toPx(value, fromUnit, ctx);
  if (px === null) return null;
  const result = {};
  for (const u of UNITS) result[u] = fromPx(px, u, ctx);
  return result;
}
```

入力を一度pxに変換し、そこから全11単位に変換します。px を中間表現として使うことで変換ロジックを一箇所に集約できます。

### 数値フォーマット

```js
function formatValue(n) {
  if (n === null || isNaN(n) || !isFinite(n)) return '—';
  const abs = Math.abs(n);
  if (abs === 0) return '0';
  if (abs < 0.001) return n.toExponential(4);
  if (abs < 1) return parseFloat(n.toFixed(6)).toString();
  return parseFloat(n.toFixed(4)).toString();
}
```

`parseFloat(n.toFixed(4)).toString()` で末尾ゼロを除去（`"1.5000"` → `"1.5"`）。`0.001` 未満は指数表記。

## 機能

- **11単位対応** — px / em / rem / % / vw / vh / pt / pc / cm / mm / in
- **カスタムコンテキスト** — ベースフォントサイズ・ビューポート幅/高さ・親要素サイズを設定
- **全単位を一覧表示** — 入力値を変更するたびに全11単位をリアルタイム更新
- **元の単位をハイライト** — ソース単位のカードを緑枠で表示
- **クリックでコピー** — 各カードをクリックで `16px` 形式でクリップボードにコピー

## テスト

Node.js `assert` モジュールで **122テスト** を実装、全パス確認済み。

テストカテゴリ:
- 定数チェック — `1in=96px`, `1pt=96/72px`, `1pc=12pt`, `1cm=10mm`
- `toPx` — 全11単位、負の値、未知単位→null
- `fromPx` — 全11単位、ゼロ、負の値
- `convert` — 単位間ペア変換
- `convertAll` — 全キー存在確認、ゼロ変換
- `formatValue` — null/NaN/Infinity/ゼロ/指数表記
- Roundtrip — 10ペアすべてで往復変換の一致確認
- Context sensitivity — baseFontPx/vpWidth/vpHeight/parentPx の変化が正しく反映されるか
- UNITS配列 — 全11単位の存在確認

## 使い方

https://css-unit-converter.pages.dev/ にアクセスして、値を入力し単位を選ぶだけです。

---

DevNest.io では他にも開発者向けツールを公開しています。
