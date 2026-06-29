---
title: "カラーパレットジェネレーターをブラウザで完結させた話（デザイン・フロントエンド向け）"
emoji: "🎨"
type: "tech"
topics: ["css", "color", "design", "javascript", "devtools"]
published: true
---

## 作ったもの

ブラウザだけで動くカラーパレットジェネレーター。ベースカラーを入力すると、補色・三色配色・類似色など6種類のパレットを即座に生成します。

👉 https://color-palette-gen-bxd.pages.dev

## 機能

- **補色（Complementary）** — 色相環の反対側（180°）
- **三色配色（Triadic）** — 均等な3色（120°間隔）
- **類似色（Analogous）** — 隣接する色（±30°、±60°）
- **スプリット補色（Split-Complementary）** — 補色の両隣
- **正方形配色（Square）** — 均等な4色（90°間隔）
- **モノクロマティック** — 明度違いの7色シェード

各スウォッチはHEX・RGB・HSLの3形式で表示。クリック一発でコピー。CSSカスタムプロパティ形式でのエクスポートも可能。

## 技術

- Vanilla JS / 単一HTMLファイル
- 色彩数学（HSL↔RGB↔HEX変換、色相シフト、明度調整）をブラウザ内で完結
- Cloudflare Pages でホスティング

## カラーマスの計算

```js
function shiftHue(hex, delta) {
  const rgb = hexToRgb(hex);
  const { h, s, l } = rgbToHsl(rgb.r, rgb.g, rgb.b);
  const newH = ((h + delta) % 360 + 360) % 360;
  const nr = hslToRgb(newH, s, l);
  return rgbToHex(nr.r, nr.g, nr.b);
}

// 補色 = +180°、三色 = +120°/+240°、類似色 = ±30°
```

## devnestio について

https://devnestio.pages.dev

フロントエンド・インフラ・バックエンドエンジニア向けのブラウザツール集。すべて無料・登録不要。
